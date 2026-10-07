# Mostrador Copilot

Base abierta e independiente de proveedores para un **copiloto comercial de mostrador**,
orientado al caso de uso de Farmaenlace. El personal consulta productos y disponibilidad,
prepara una reserva, revisa sus condiciones y decide si ejecutarla.

**Estado:** arquitectura ejecutable con API y demostración local. No es una integración
oficial con Farmaenlace, un sistema clínico ni un producto listo para producción.

Si te incorporas al proyecto, empieza por [CONEXT.md](CONEXT.md): contexto del hackatón,
decisiones, estado real y guía de continuación para colaboradores.

## Qué incluye

- Dominio separado de la API y de los servicios externos mediante cuatro contratos Python.
- Catálogo, existencias y precios totalmente inventados; cantidades enteras y dinero en centavos.
- Propuestas con vencimiento, aprobación/rechazo explícitos y comprobación de cambios de stock/precio.
- Identidad y permisos, control de duplicados, persistencia y eventos por operación.
- Estado `uncertain` cuando no puede saberse si un proveedor completó una operación.
- API FastAPI con documentación interactiva y contratos OpenAPI para un futuro frontend.
- Adaptadores SQLite, identidad de demostración y búsqueda por palabras, sin cuentas externas.

No incluye datos, código, prompts ni conectores de Orito o AHQ. Tampoco incluye modelos
de IA, servicios cloud, WhatsApp, ERP/POS, pagos, historias clínicas ni frontend propio.
La implementación se escribió desde cero; ver [procedencia](docs/provenance.md).

## Arranque local

Requisitos: Python 3.11+ y [uv](https://docs.astral.sh/uv/getting-started/installation/).

```sh
uv sync --locked
COPILOT_MODE=demo uv run uvicorn mostrador.api:create_app --factory --host 127.0.0.1 --port 8000
```

Abre <http://127.0.0.1:8000/docs>. En **Authorize**, introduce `demo-clerk` como token
(sin escribir `Bearer`). `demo-supervisor` permite revisar propuestas de otros actores;
`demo-viewer` únicamente consulta. **Son tokens públicos de demo, no credenciales seguras.**
No expongas este modo a Internet ni cargues información de personas reales.

La base se crea en `.local/mostrador.sqlite`, ignorada por Git. Para una sesión nueva,
usa otra ruta con `COPILOT_DB_PATH=.local/otra-demo.sqlite`; reiniciar no repone el stock.
`.env.example` documenta las opciones; copiarlo no carga las variables automáticamente.

### Recorrido de demostración

1. `GET /offers?q=gasas`: compara el mismo SKU en las sucursales ficticias `centro` y `norte`.
2. `POST /proposals`: envía el cuerpo de abajo. Todavía **no** se descuenta stock.
3. Revisa `quote`, `quote.total_cents`, `expires_at` y `status` en la respuesta.
4. `POST /proposals/{id}/decision` con `{"decision":"approve"}` o `{"decision":"reject"}`.
5. Consulta `GET /proposals/{id}/events` para ver quién propuso y aprobó la operación.

```json
{
  "sku": "DEMO-001",
  "branch_id": "centro",
  "quantity": 2,
  "idempotency_key": "demo-reserva-001"
}
```

Ejemplo desde terminal:

```sh
curl -H 'Authorization: Bearer demo-clerk' 'http://127.0.0.1:8000/offers?q=gasas'
curl -X POST http://127.0.0.1:8000/proposals \
  -H 'Authorization: Bearer demo-clerk' -H 'Content-Type: application/json' \
  -d '{"sku":"DEMO-001","branch_id":"centro","quantity":2,"idempotency_key":"demo-reserva-001"}'
```

`POST /assistant` con `{"message":"buscar gasas"}` demuestra el contrato del asistente:
solo genera una consulta comercial. **El adaptador actual no usa IA ni comprende lenguaje
natural general.** Un futuro modelo tampoco recibirá autorización para aprobar reservas.

## Arquitectura

```text
Frontend futuro / Swagger / otro cliente
                  │ HTTP + identidad
             API FastAPI
                  │
          Servicio de dominio
       ┌──────────┼───────────┬─────────────┐
    Identity   Assistant   Commerce   WorkflowStore
       │          │           │             │
  Demo tokens  Palabras    SQLite demo   SQLite demo
       └──── reemplazables por contratos ────┘
```

| Directorio | Responsabilidad |
| --- | --- |
| `src/mostrador/domain.py` | Entidades y reglas básicas, sin SDK de proveedor |
| `src/mostrador/service.py` | Flujo de consulta, propuesta y aprobación |
| `src/mostrador/ports.py` | Contratos para integrar servicios |
| `src/mostrador/adapters/` | Implementaciones exclusivamente locales |
| `src/mostrador/api.py` | HTTP, validación de entradas y OpenAPI |
| `src/mostrador/bootstrap.py` | Selección e inyección de adaptadores |
| `tests/` | Pruebas de dominio, concurrencia, adaptadores y API |

No se fija proveedor de nube, IA, identidad o base de datos externa. Para conectarlos
se implementan los contratos y se configura una fábrica; **no basta con añadir una API key**.
El modo `external` exige esa configuración y no cae silenciosamente en demo.

- [Arquitectura y garantías](docs/architecture.md)
- [Guía para integrar servicios](docs/providers.md)
- [Diseño y alcance](docs/design.md)
- [Seguridad y límites](SECURITY.md)

## Validación

```sh
uv run pytest -q
uv run ruff check .
uv run ruff format --check src tests examples
```

Se comprueban, entre otros casos, aprobaciones concurrentes, reenvíos idempotentes,
precios modificados, vencimiento, permisos, persistencia y respuestas inciertas.
Estas pruebas usan los adaptadores locales; no certifican servicios que se integren después.

### Contenedor opcional

```sh
docker build -t mostrador-copilot .
docker volume create mostrador-demo
docker run --rm -p 127.0.0.1:8000:8000 -e COPILOT_MODE=demo \
  -e COPILOT_DB_PATH=/app/data/demo.sqlite -v mostrador-demo:/app/data mostrador-copilot
```

El contenedor ejecuta un usuario sin privilegios. Revisa permisos si utilizas un bind mount.
No contiene `.context`, adjuntos, credenciales ni bases locales. Docker es opcional: el
recorrido de referencia y las pruebas se ejecutan con `uv`.

## Siguiente incremento para el hackatón

1. Interfaz de mostrador con productos, sucursales, propuesta visible y confirmación explícita.
2. Adaptadores para los servicios autorizados por la organización; mantener este dominio estable.
3. Asistente de IA opcional para interpretar búsquedas y explicar resultados verificables.
4. Evaluar latencia, exactitud del catálogo y tareas completadas con escenarios sintéticos.

Antes de participar, confirmar con la organización si permite reutilizar esta base preparada
antes del evento. Declarar la base, herramientas y licencias; no presentarla como trabajo
íntegramente construido durante el hackatón si no lo fue.

## Licencia y publicación

Código original bajo [MIT](LICENSE). Las dependencias conservan sus licencias.
Este repositorio no distribuye materiales privados del hackatón ni de los proyectos revisados.
Consulta [CONTRIBUTING.md](CONTRIBUTING.md) antes de añadir datos o conectores.
La configuración pública/privada del repositorio remoto se administra por separado;
añadir una licencia no cambia esa visibilidad.
