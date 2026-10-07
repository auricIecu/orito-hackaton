# Procedencia y preparación para publicación

## Código original

Esta base se escribió desde cero para explorar un copiloto comercial de mostrador.
No se copiaron archivos, fragmentos, prompts, conectores, pruebas, esquemas de datos,
assets ni datasets de Orito o AHQ. Las referencias son únicamente patrones generales:

- Separación de presentación, coordinación y operaciones de negocio.
- Cotización determinista y confirmación explícita de una acción.
- Propuestas, autorización, registro de eventos e idempotencia.
- Tratamiento visible de fallos y resultados inciertos.

[AHQ](https://github.com/nathanjcx/ahq) fue una referencia de arquitectura, no una dependencia.
La revisión no identificó una licencia que autorizara reutilizar su implementación;
no se redistribuye código suyo. Orito y sus integraciones permanecen fuera de este repo.

Los productos `DEMO-*`, sucursales, precios y existencias son inventados. No se incorporan
datos de clientes, precios reales, documentación interna ni materiales del hackatón.
Farmaenlace se menciona como contexto del caso de uso; no se afirma afiliación, aprobación
ni compatibilidad con sus sistemas. No se utilizan logos ni marcas gráficas.

## Licencias

`LICENSE` aplica MIT a esta implementación original. Los paquetes de terceros mantienen
sus propios términos, documentados en sus distribuciones. `uv.lock` fija versiones para
reproducir la instalación; no otorga derechos sobre software ajeno.

## Checklist antes de hacer público el remoto

- Revisar archivos y también el historial Git que vaya a publicarse.
- Verificar que `.context/`, adjuntos, `.env`, bases SQLite y credenciales no están tracked.
- Confirmar autoría y aceptación de MIT de todos los colaboradores.
- Revisar licencias de cualquier dependencia, asset o conector añadido después.
- Confirmar con la organización las reglas para reutilizar una base previa al evento.
- Habilitar detección de secretos y la política de reporte de vulnerabilidades al publicar.

Un `.gitignore` evita añadidos accidentales, pero no elimina secretos previamente versionados.
Cambiar visibilidad o publicar en GitHub es una operación independiente del scaffolding local.
