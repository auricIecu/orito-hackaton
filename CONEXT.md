# Contexto para continuar — Mostrador Copilot

Actualizado: 7 de octubre de 2026. Este archivo conserva el nombre `CONEXT.md`
solicitado para el traspaso. Es el punto de entrada para quien no participó en la conversación.

## 1. Qué queremos construir

Un **copiloto comercial para el personal de mostrador de una farmacia**, orientado al
caso de uso de Farmaenlace. Debe ayudar a encontrar un producto, consultar disponibilidad
por sucursal y preparar una acción que una persona pueda revisar y aprobar.

La propuesta de valor a validar es reducir el tiempo de consulta y las ventas perdidas
por no encontrar disponibilidad, sin sustituir al dependiente ni al sistema comercial.
No hemos medido todavía ese impacto: es una hipótesis, no un resultado demostrado.

El usuario consideró también postventa/customer support de BYD. Se decidió enfocar esta
base en Farmaenlace; **no hay implementación BYD ni dos productos paralelos**. No ampliar
el alcance a ambas verticales antes de conseguir una demo sólida de mostrador.

Ejemplo de historia para la demo: una persona pide un producto concreto; el dependiente
consulta el mismo SKU en varias sucursales, prepara una reserva en la sucursal elegida,
revisa cantidad/precio y confirma. Si las condiciones cambiaron, la operación se rechaza
y se debe preparar una propuesta nueva. Todo lo mostrado actualmente es sintético.

## 2. Contexto del hackatón

Según el resumen de kickoff compartido por el usuario, el evento es el 8 de octubre,
con FarmEnlace y BYD como auspiciantes. Los retos abarcan eficiencia operativa, ventas
y atención al cliente. Se valoran propuesta de valor, calidad técnica, novedad,
viabilidad y claridad del pitch.

La referencia de planificación recibida es inicio a las 09:00 y entrega a las 15:45:
código, demo y presentación breve. **Confirmar la agenda final con la organización**;
este documento no es una comunicación oficial ni un calendario actualizado del evento.

Restricciones importantes para trabajar:

- No contamos con datos reales ni acceso confirmado a sistemas de las empresas.
- Preparar datasets propios/sintéticos, sin datos de pacientes o clientes reales.
- La organización puede ofrecer infraestructura y herramientas; no se han conectado
  cuentas ni asumido que una API específica estará disponible.
- Se indicó construir durante el evento y declarar herramientas/licencias. Esta base
  se preparó antes: confirmar expresamente si puede reutilizarse y declararla como tal.
- No compartir información no pública ni subir los materiales de revisión al repositorio.

## 3. Decisiones del usuario y propiedad intelectual

La petición fue crear un repo abierto, aprovechando **ideas de estructura y arquitectura**
de Orito y AHQ, pero no datos, servicios o conectores propietarios. Se implementó código
nuevo, no un fork ni una migración de esos proyectos.

| Referencia conceptual | Qué se conserva como idea | Qué NO se incorporó |
| --- | --- | --- |
| Orito | Separar presentación, coordinación del asistente y operaciones; cotización y confirmación | Código, prompts, datos, conectores, credenciales, reglas de su negocio |
| [AHQ](https://github.com/nathanjcx/ahq) | Propuestas, aprobación, permisos, eventos, idempotencia y estados inciertos | Código, assets, stack, SDKs ni implementación de sus herramientas |

No se identificó en la revisión una licencia de AHQ que autorizara copiar su implementación.
Su disponibilidad en GitHub no se interpreta como autorización de reutilización.

Esta implementación original tiene licencia MIT. No usar logos ni insinuar afiliación o
integración oficial con Farmaenlace. Ver [procedencia](docs/provenance.md) y
[reglas de contribución](CONTRIBUTING.md).

**No necesitamos los repos originales para ejecutar o continuar este proyecto.**
Los adjuntos de kickoff, el PDF y los materiales de análisis se mantienen fuera de Git.
No copiar `.context/`, `.env`, bases locales ni documentos de terceros al repositorio.

## 4. Estado real al entregar esta base

### Implementado

- Backend Python 3.11+, FastAPI y dominio sin SDKs externos.
- API con OpenAPI/Swagger y validación estricta de solicitudes.
- Catálogo sintético con tres SKU y dos sucursales, precios en centavos USD y stock local.
- Consulta, propuesta de reserva, aprobación/rechazo y consulta del historial de eventos.
- Identidad demo, roles y permisos por propietario de propuesta.
- Idempotencia, control de concurrencia, comprobación de versión/precio/stock y expiración.
- Persistencia SQLite; reiniciar el servicio no reinicia inventario ni propuestas.
- Cuatro contratos reemplazables y una fábrica de composición configurable.
- Pruebas, lockfile, documentación, Dockerfile y workflow de CI.

### No implementado

- Interfaz visual de mostrador: Swagger es una herramienta de exploración, no la UI final.
- IA conectada: el asistente demo solo interpreta búsquedas sencillas por palabras.
- Integraciones con sistemas de Farmaenlace, nube, autenticación real, ERP/POS o mensajería.
- Historial 360 de clientes, promociones, listas segmentadas de precios o fidelización.
- Pagos, cancelación de reservas, reconciliación automática o panel de gestión de incidencias.
- Diagnóstico, recetas, dosificación o sustitución de medicamentos.
- Despliegue público, multitenancy o preparación completa para producción.

No presentar la base como una integración terminada ni el buscador demo como un LLM.

## 5. Arquitectura y archivos que leer

```text
Cliente futuro / Swagger
         │ HTTP
api.py: autenticación, DTO y serialización
         │
service.py: consulta → propuesta → decisión → ejecución
         │
ports.py: Identity | Assistant | Commerce | WorkflowStore
         │
adapters/: tokens demo | búsqueda por palabras | SQLite

bootstrap.py construye e inyecta los adaptadores.
domain.py define entidades y reglas básicas sin depender de infraestructura.
```

Orden recomendado de lectura:

1. [README.md](README.md): instalar, ejecutar y probar la demo.
2. [docs/architecture.md](docs/architecture.md): garantías, límites y estados.
3. [src/mostrador/ports.py](src/mostrador/ports.py): contrato de cada proveedor.
4. [src/mostrador/service.py](src/mostrador/service.py): comportamiento del copiloto.
5. [tests/test_workflow.py](tests/test_workflow.py): ejemplos ejecutables de las reglas.
6. [docs/providers.md](docs/providers.md): integración y pruebas exigidas a un adaptador.

Se eligió un único servicio para mantener bajo el coste de implementación. No hace falta
replicar la infraestructura completa de Orito/AHQ ni añadir microservicios, colas o MCP
para completar la historia de demo actual. El frontend y los proveedores siguen abiertos.

## 6. Invariantes que no debemos romper

- **Consultar o proponer no reserva stock.** Solo una decisión explícita y autorizada ejecuta.
- **El servidor obtiene precios y stock.** No aceptar un precio calculado por el cliente o el LLM.
- Cantidades enteras positivas, máximo 10.000; importes en centavos, no floats monetarios.
- La propuesta conserva el payload cotizado. Para cambiarlo se crea otra propuesta.
- La propuesta vence a los 300 segundos; se marca `expired` al intentar decidir, sin scheduler.
- La reserva valida versión, precio y stock atómicamente. Una versión distinta invalida
  la propuesta incluso si aún queda stock suficiente.
- Una clave idempotente identifica la solicitud de un actor; no se reutiliza para otro payload.
- El ID de propuesta es también el ID estable de operación enviado a comercio.
- Aprobar dos veces no debe duplicar la reserva. El store reclama la ejecución atómicamente
  y el proveedor debe ofrecer también idempotencia durable.
- Una respuesta perdida puede significar que el proveedor sí reservó: marcar `uncertain`,
  no reintentar a ciegas. Un proceso interrumpido también puede dejar `executing`.
- `DomainError` desde comercio significa rechazo **sin efecto confirmado**; no usarlo para
  camuflar un timeout o devolver mensajes privados del proveedor.
- El asistente solo devuelve `SearchIntent`. No tiene acceso a la aprobación ni a SQL.
- La autorización reside en el servidor, no en botones deshabilitados o instrucciones al modelo.

Los roles son `viewer`, `clerk` y `supervisor`. El dependiente puede aprobar su propia
propuesta; no hay doble aprobación obligatoria. Un supervisor puede revisar las de otros.
El despliegue es de una sola organización y no hay permisos por sucursal.

El historial registra transiciones con actor y tiempo, no prompts ni tokens. No es un log
criptográficamente inmutable ni una auditoría de todas las lecturas. No se promete ejecución
"exactly once" entre sistemas externos. Ver los límites completos en [SECURITY.md](SECURITY.md).

## 7. Levantarlo desde cero

El remoto de trabajo es `https://github.com/auricIecu/orito-hackaton.git` y esta entrega
se sube a la rama `hackaton-farmenlace-byd`, sin fusionarla automáticamente con `master`.
Mientras el repo sea privado, el colaborador necesita acceso concedido por su propietario.

```sh
git clone --branch hackaton-farmenlace-byd https://github.com/auricIecu/orito-hackaton.git
cd orito-hackaton
uv sync --locked
COPILOT_MODE=demo uv run uvicorn mostrador.api:create_app --factory --host 127.0.0.1 --port 8000
```

Abrir `http://127.0.0.1:8000/docs`, pulsar **Authorize** e introducir `demo-clerk`.
Los otros tokens demo son `demo-supervisor` y `demo-viewer`. Son públicos y predecibles;
**no exponer el modo demo a Internet** ni reutilizarlos como credenciales reales.

La base aparece en `.local/mostrador.sqlite`. Para una nueva sesión sin datos anteriores,
arrancar con otra ruta `COPILOT_DB_PATH=.local/sesion-nueva.sqlite`.
`.env.example` es documentación de variables: no se carga automáticamente.

El modo por defecto es `external`, que falla si no se configura un proveedor. Por eso
el comando de demo incluye `COPILOT_MODE=demo` explícitamente.

## 8. Recorrido API para construir la interfaz

Todas las rutas comerciales requieren `Authorization: Bearer <token>`.

| Ruta | Uso |
| --- | --- |
| `GET /health` | Estado básico y modo; pública |
| `GET /offers?q=gasas` | Buscar productos y disponibilidad por sucursal |
| `POST /assistant` | Interpretar `{"message":"buscar gasas"}` como búsqueda |
| `POST /proposals` | Preparar propuesta con SKU, sucursal, cantidad y clave |
| `GET /proposals/{id}` | Consultar cotización y estado |
| `POST /proposals/{id}/decision` | Enviar `{"decision":"approve"}` o `{"decision":"reject"}` |
| `GET /proposals/{id}/events` | Consultar eventos autorizados de la propuesta |
| `GET /openapi.json` | Contrato de la API; público |

Body de ejemplo para crear una propuesta:

```json
{
  "sku": "DEMO-001",
  "branch_id": "centro",
  "quantity": 2,
  "idempotency_key": "demo-001"
}
```

Con una base nueva, muestra dos paquetes a 350 centavos cada uno: total 700 centavos.
El cliente debe mostrar ese total como USD 7,00, no como 700 dólares. Mostrar también
unidad, sucursal, fuente y vigencia antes de pedir aprobación.

El frontend debe conservar la clave cuando reenvía la misma solicitud tras un corte de red.
Para `executing`, consultar el estado; para `uncertain`, indicar revisión pendiente.
No convertir esos estados en una nueva reserva automática. No hay endpoint de listado
de propuestas ni de resolución de incidencias: habría que diseñarlos si la UI los necesita.
No hay CORS habilitado; usar el mismo origen o configurar orígenes autorizados explícitos.

## 9. Cómo integrar los servicios que habiliten

`Adapters` agrupa cuatro interfaces. Cada servicio concreto se implementa en el borde,
sin agregar un SDK de proveedor a `domain.py` o `service.py`.

- `Commerce`: búsqueda, cotización y reserva contra una fuente comercial autorizada.
- `WorkflowStore`: propuestas y eventos, con transacciones/claims y unicidad.
- `Identity`: valida credenciales y devuelve actor/rol confiables.
- `Assistant`: interpreta búsquedas; no ejecuta acciones ni inventa resultados comerciales.

Una fábrica Python confiable devuelve el bundle; se indica con
`COPILOT_ADAPTER_FACTORY=paquete:build_adapters`. Para servicios reales usar
`COPILOT_MODE=external`. No hay un menú cerrado de marcas ni conectores ya implementados.
No basta con colocar una API key para que aparezca la integración.

[examples/local_factory.py](examples/local_factory.py) muestra la inyección funcionando,
pero sigue siendo demo. No cambiar simplemente `is_demo` a `False` para aparentar producción.
Aplicar las pruebas de contrato descritas en [docs/providers.md](docs/providers.md).

## 10. Verificación y pendientes técnicos conocidos

```sh
uv run pytest -q
uv run ruff check .
uv run ruff format --check src tests examples
uv build
```

La base se verificó localmente con Python 3.12: 40 pruebas pasaron, lint/formato correctos,
wheel/sdist construidos y recorrido HTTP real completado. Volver a ejecutar estos comandos
al modificarla; este resultado no certifica integraciones futuras.

- Hay una advertencia de deprecación upstream por `httpx` en Starlette TestClient;
  no impide las pruebas actuales, pero revisar al actualizar dependencias.
- Dockerfile incluido; no se pudo ejecutar el contenedor porque el daemon local estaba apagado.
- CI está configurado para Python 3.11 y 3.12; revisar su primera ejecución remota después del push.
- No se han probado proveedores reales ni carga de producción.
- El paquete fuente usa una lista de archivos permitidos para excluir carpetas de análisis.
  Mantener esa protección cuando se añadan directorios.

## 11. Próximo incremento recomendado

Esta es una propuesta de continuación, no trabajo ya implementado ni decisiones cerradas:

1. **Confirmar reglas y servicios del evento.** Resolver permiso para reutilizar la base,
   acceso del colaborador y cuentas disponibles. No empezar suponiendo acceso a un ERP.
2. **Construir UI de mostrador.** Buscador, resultados por sucursal, detalle de propuesta,
   confirmación, resultado e historial. Conectar primero a esta API demo estable.
3. **Integrar un proveedor útil.** Elegirlo según disponibilidad real; respetar contratos
   y probar fallos/concurrencia. Si no hay backend empresarial, declarar la simulación.
4. **Agregar IA solo donde aporte.** Interpretar la intención de búsqueda; cualquier
   expansión de sus capacidades debe mantener cotización y autorización fuera del modelo.
5. **Cerrar demo y pitch.** Mostrar tanto el camino exitoso como stock cambiado o doble clic;
   explicar ahorro de tiempo esperado, límites e integración necesaria para un piloto.

Si trabajan dos personas: una puede construir UI sobre OpenAPI y otra integrar/adaptar
el backend y preparar pruebas/demo. Acordar cambios de contrato antes de implementarlos.
Evitar iniciar a la vez voz, WhatsApp, fidelización, BYD y análisis clínico.

Decisiones todavía abiertas: framework de frontend, proveedor de IA/nube/identidad,
infraestructura para despliegue, datasets sintéticos ampliados, experiencia visual y
cuándo cambiar la visibilidad del remoto. Subir código no vuelve público el repositorio.

## 12. Cómo mantener este contexto

Actualizar este archivo cuando cambien alcance, contratos, proveedores o resultados de
verificación. Separar siempre hechos implementados, hipótesis y pendientes. Mantener
secretos y documentación no pública fuera de Git, aunque el remoto siga siendo privado.
No atribuir a Farmaenlace validación o resultados que aún no ha dado.
