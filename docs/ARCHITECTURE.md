# Arquitectura e Integraciones

## Decisiones Técnicas y Arquitectónicas
1. **Lógica de Sobrescritura en Memoria (Bisturí):** En vez de alterar el esquema de base de datos dividiendo Clientes y Facturas (lo cual habría roto la compatibilidad con el front-end existente), se optó por manejar actualizaciones "en cascada" iterando sobre todas las filas correspondientes al mismo NIT durante las cargas masivas.
2. **Formato de correos nativo:** Para habilitar múltiples correos sin crear tablas relacionales extra, se reutilizó la columna `String` nativa, separando correos con punto y coma (`;`). La interfaz (Jinja2) y el back-end (`clean_email`) operan transparentemente sobre esta cadena.
3. **Manejo Asíncrono de Interfaz:** En el proceso de "Cobro Masivo", para evitar que el navegador congele la experiencia del usuario (Timeout), se emplea JavaScript con `setTimeout` para renderizar el "spinner" antes de que el motor de Python bloquee el hilo enviando correos sincrónicamente.

## Integraciones
### 1. Outlook / SMTP
- **Finalidad:** Despacho de cartas de cobro.
- **Forma de conexión:** Librería estándar de Python `smtplib.SMTP`.
- **Autenticación:** STARTTLS mediante `SMTP_USER` y `SMTP_PASS` (Variables de entorno `.env`).
- **Posibles fallos:** `SMTPAuthenticationError` (si cambia la contraseña o Microsoft exige MFA/App Password).

## Despliegue (Deployment)
El sistema opera en un entorno de red local cerrado.
### Proceso de Publicación
1. El código reside en GitHub (rama `main`).
2. En el servidor local (físico o VM de la empresa), abrir consola.
3. Ejecutar `git pull` para obtener los cambios.
4. Reiniciar la tarea de Uvicorn (Ej: cerrar CMD y ejecutar `start_server.bat`).
### Backups
- Recomendado: Respaldar las carpetas `cartas_generadas` y `certificados_generados` periódicamente.
- El código está respaldado en la nube.
- La BD debe incluirse en los planes de respaldo diarios corporativos del servidor SQL.

## Roles y Permisos (Seguridad)
Actualmente, el sistema **no implementa una capa de autenticación ni roles por usuario a nivel de aplicación**.
- **Acceso:** Se asume que quien tiene acceso a la red local y a la IP/Puerto del servidor (ej. `127.0.0.1:8000`) es personal de confianza del área de Cartera.
- **INFORMACIÓN PENDIENTE DE VALIDAR:** No existe JWT, OAuth, ni sesiones en el código disponible. Si la empresa desea exponer esto a internet, **deberá implementarse obligatoriamente una capa de seguridad (Autenticación y Autorización).**
- **Sanitización:** Los correos y teléfonos pasan por funciones regex (ej. `clean_email`) para evitar inyección y malformaciones.
