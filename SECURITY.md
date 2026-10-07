# Seguridad

Esta base es de demostración. No usar datos personales, clínicos ni credenciales de
producción, y no exponer a Internet el modo `demo`: sus tokens son públicos y fijos.

Las acciones requieren autenticación, permisos y confirmación; las entradas no permiten
enviar un rol o cambiar un precio. Esto no sustituye hardening de producción.

Antes de desplegar para usuarios reales hacen falta identidad real, HTTPS, gestión de
secretos, límites de tráfico, acceso por sucursal/organización cuando aplique, backups,
retención de datos, monitoreo, auditoría protegida y un proceso de reconciliación de
operaciones `executing`/`uncertain`. Validar las garantías del adaptador comercial.

No hay diagnóstico, dosificación ni sustitución de medicamentos. No añadir consejo
clínico o decisiones automáticas al asistente como extensión incidental de esta demo.

No se guardan prompts ni tokens en eventos. Evitar añadir logging de bodies o headers
de autenticación. Los adaptadores deben devolver códigos de error y recibos saneados;
son código confiable, no plugins descargados durante una conversación.

Para reportar un problema, contactar al mantenedor por un canal privado. Al publicar,
habilitar el reporte privado de vulnerabilidades en GitHub. No adjuntar secretos o
datos personales a issues públicos. Actualmente no hay SLA de respuesta definido.
