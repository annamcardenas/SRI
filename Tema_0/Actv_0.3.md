# Actividad 0.4 - Usando cUrl

## Busca información sobre el comando curl y muestra al menos cinco ejemplos de uso

cURL es una herramienta de línea de comandos para transferir datos con sintaxis de URL. Soporta múltiples protocolos (HTTP, HTTPS, FTP, FTPS, SCP, SFTP) y es ampliamente utilizado para testing de APIs, descargas automáticas y debugging de conexiones.

## Ejemplos esenciales de uso:

**Descarga básica de contenido web**

curl https://www.ejemplo.com

Descarga y muestra el contenido HTML de la página en la terminal.

**Guardar contenido en archivo local**

curl -o pagina.html https://www.ejemplo.com

Almacena el contenido descargado en un archivo específico en lugar de mostrarlo en pantalla.

**Solicitud con verificación de headers**

curl -I https://www.ejemplo.com

Solo muestra los encabezados HTTP de la respuesta, útil para verificar códigos de estado y metadatos.

**Envío de datos mediante POST**

curl -X POST -d "usuario=ana&password=123" https://api.ejemplo.com/login

Simula el envío de datos de formulario mediante método POST, común en APIs REST.

**Descarga conservando nombre original**

curl -O https://www.ejemplo.com/informe.pdf

Descarga archivos manteniendo el nombre que tienen en el servidor.

**Funcionalidades avanzadas**

cURL permite autenticación básica con -u usuario:clave, seguir redirecciones con -L, modo verbose con -v para debugging detallado, y trabajo con cookies mediante -c y -b. Es particularmente útil para testing de servicios web y automatización de tareas de transferencia de datos.

La herramienta está disponible por defecto en la mayoría de sistemas Unix/Linux y puede instalarse en Windows, siendo una alternativa ligera a navegadores gráficos para operaciones web automatizadas.

