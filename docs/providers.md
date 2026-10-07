# Conectar servicios sin acoplar el dominio

No hay una lista cerrada de proveedores. La aplicación conoce cuatro `Protocol` de Python
en `src/mostrador/ports.py`. Elegir una plataforma implica implementar sus adaptadores,
instalar las dependencias que necesite y validar sus garantías; no modificar `Copilot`.

## Ejemplo ejecutable de inyección

Desde la raíz del repositorio:

```sh
COPILOT_MODE=demo \
COPILOT_ADAPTER_FACTORY=examples.local_factory:build_adapters \
uv run uvicorn mostrador.api:create_app --factory --host 127.0.0.1 --port 8000
```

Ese ejemplo sustituye solamente el intérprete de búsquedas y sigue marcado como demo.
El módulo de la fábrica debe estar instalado o disponible en `PYTHONPATH`; `examples/`
se distribuye como ejemplo del repo, no como parte del paquete wheel de producción.

En una integración real, la fábrica devuelve:

```python
from mostrador.ports import Adapters

def build_adapters() -> Adapters:
    # Cada objeto se construye con configuración privada del despliegue.
    return Adapters(
        commerce=your_commerce,
        store=your_workflow_store,
        identity=your_identity,
        assistant=your_assistant,
        is_demo=False,
    )
```

Los nombres `your_*` son ilustrativos: no son conectores incluidos.
Configurar `COPILOT_MODE=external` y `COPILOT_ADAPTER_FACTORY=paquete:build_adapters`.
El arranque falla si falta la fábrica o si devuelve un bundle marcado como demo.
`is_demo` es una declaración del integrador, no una certificación: no cambiarlo a `False`
para publicar adaptadores de prueba. La fábrica es configuración confiable del servidor,
nunca una entrada del usuario o una herramienta que pueda elegir el modelo.

## Obligaciones por contrato

### `Commerce`

- `search(query)`: devuelve `Offer` con SKU exacto, sucursal, presentación, stock, versión,
  precio en centavos, moneda y fuente. No inventar coincidencias ni equivalencias clínicas.
- `quote(sku, branch_id, quantity)`: cotización autoritativa, cantidad válida y hora UTC
  en segundos Unix. No reservar ni modificar estado en esta llamada.
- `reserve(quote, operation_id)`: verifica condiciones y escribe de forma atómica.
  La misma clave y payload devuelve el mismo recibo; misma clave con otro payload falla.
  Usar `operation_id` como clave idempotente en el proveedor, no generar otra en cada intento.
- `DomainError` se usa solo para rechazos **sin efectos garantizados** y códigos seguros,
  nunca para copiar mensajes internos o PII. Un resultado ambiguo lanza otra excepción.
- Normalizar el recibo; devolver únicamente campos públicos necesarios, sin respuestas
  completas del ERP, tokens o datos personales. Configurar timeouts en el adaptador.
- Si el servicio no ofrece escritura condicional e idempotencia durable, no anunciar una
  reserva segura. Mantener solo lectura/propuesta hasta resolver esa limitación.

### `WorkflowStore`

- Persistencia durable: no un diccionario en memoria para una integración real.
- Unicidad `(actor_id, idempotency_key)`. `create` devuelve la propuesta ya existente ante
  una carrera; el servicio comprueba que el payload solicitado coincida.
- `claim`: transacción atómica entre estado y evento. Solo un llamador recibe `claimed=True`.
  Debe comprobar vencimiento, resolver reenvíos y rechazar decisiones contradictorias.
- `finish`: únicamente `executing` → `executed/failed/uncertain`, estado y evento atómicos.
- `get`, `find`, `events`: reconstruyen contratos de dominio; orden de eventos estable.
- Los permisos se comprueban en el servicio. No exponer directamente el store a HTTP ni IA.

### `Identity`

Validar credenciales en el servidor y devolver `Actor(id, role)`. No aceptar roles del body,
query string o claims sin validar. Los únicos roles actuales son `viewer`, `clerk` y
`supervisor`. Usar errores genéricos 401; secretos en configuración privada. Añadir autorización
por sucursal/tenant requiere ampliar el dominio y sus pruebas, no solo el login.

### `Assistant`

`interpret(message)` solo puede devolver `SearchIntent(query)`. Puede conectarse a un modelo,
un buscador o continuar sin IA. La salida se valida por tipo/longitud y el dominio obtiene
los productos de `Commerce`; el modelo no fija precios ni reservas. No pasar al modelo
credenciales, APIs de aprobación o acceso directo al store. La interpretación clínica y
las recomendaciones terapéuticas no forman parte de este contrato.

Todos los puertos son síncronos. FastAPI ejecuta las rutas síncronas fuera del event loop;
cada adaptador es responsable de timeouts, conexiones y seguridad entre threads. Si el
proveedor exige trabajos prolongados, diseñar después un worker con reconciliación y
reutilizar el ID de operación, en vez de introducir reintentos ciegos.

## Pruebas antes de activar un proveedor

Usa como especificación los casos de `tests/test_adapters.py` y `tests/test_workflow.py`;
los tests actuales construyen SQLite, por lo que debes añadir fixtures para tu proveedor
en un sandbox autorizado. Comprobar:

1. Lectura y cotización sin cambios de estado; precios y unidades exactos.
2. Cambio de precio/stock después de cotizar; rechazo sin efectos.
3. Dos aprobaciones simultáneas; un único despacho.
4. Misma operación repetida y payload contradictorio.
5. Timeout antes y después de un posible commit; estado ambiguo y reconciliación.
6. Reinicio del proceso, persistencia, permisos y ausencia de secretos en respuestas/eventos.

No ejecutar pruebas contra sistemas comerciales reales sin autorización. CI no necesita
credenciales ni hace llamadas a proveedores externos.
