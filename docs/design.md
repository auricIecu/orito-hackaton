# Mostrador: alcance de la base abierta

Implementación original de una arquitectura de copiloto de mostrador. Se toma
como referencia conceptual la separación entre conversación y operaciones y el
flujo de acciones supervisadas. No se importan código, prompts, datos, recursos
gráficos, credenciales ni conectores de proyectos anteriores o de terceros.

## Entrega

API FastAPI ejecutable, dominio en Python estándar, contratos de proveedores,
adaptadores locales SQLite/identidad demo/búsqueda por palabras, pruebas,
contenedor y documentación. OpenAPI sirve como interfaz de exploración. La UI
de producto y los proveedores reales quedan fuera de esta entrega arquitectónica.

La demo consulta productos sintéticos, prepara una reserva, permite aprobarla
o rechazarla y registra sus transiciones. Una consulta nunca reserva inventario.
El asistente local es determinista; no se presenta como un modelo de IA.

## Límites y decisiones

- Python >=3.11; FastAPI sólo en transporte. Sin SDK de proveedores en el núcleo.
- Un despliegue representa una organización; no se afirma aislamiento multitenant.
- Importes en centavos enteros USD; cantidades enteras de la unidad comercial.
- No recomendaciones clínicas, sustituciones terapéuticas ni datos de pacientes.
- Identidad autenticada por un puerto, nunca por actor/rol enviados en un body.
- Lectores consultan; dependientes proponen y deciden sobre sus propias propuestas;
  supervisores también deciden sobre propuestas de otros.
- Propuestas inmutables con vigencia de cinco minutos. Aprobar consume la versión
  exacta de oferta revisada; cambios de precio/stock invalidan esa propuesta.
- Doble aprobación no duplica la reserva. El proveedor recibe una clave estable.
- Timeout después de despacho implica resultado incierto, sin reintento automático.
- Una ejecución interrumpida permanece visible como executing para reconciliación.
- Auditoría persistente de transiciones, con actor y tiempo, sin tokens ni prompts.
- Adaptadores demo sólo en modo demo explícito; modo external requiere composición
  propia y falla al arrancar si falta. Ninguna caída activa un fallback demo.
- Licencia MIT para este código nuevo; sin sublicenciar material de terceros.
