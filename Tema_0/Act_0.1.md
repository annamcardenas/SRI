# Actividad 0.1- HTTP Introduction

## ¿Quién, dónde y cuándo se crea el primer servidor web?

Tim Berners-Lee en 1990 fue quien creó el primer servidor web en el  CERN (Organización Europea para la Investigación Nuclear), en Ginebra, Suiza.

## ¿Qué es pila de protocolos usados por http?

La pila de protocolos usada por HTTP es un conjunto de protocolos organizados en capas que permiten la comunicación de datos en internet, con HTTP operando en la capa superior, conocida como la capa de aplicación.
HTTP depende de otros protocolos para funcionar, principalmente TCP (Protocolo de Control de Transmisión) y IP (Protocolo de Internet), que forman parte del modelo TCP/IP

## ¿Componentes de una URL?

Una URL (Localizador Uniforme de Recursos) actúa como la dirección única de un recurso en la web y se compone de varias partes estructuradas. Usando el ejemplo https://www.ejemplo.com:443/carpeta/pagina.html?busqueda=hola#intro:

**Protocolo o Esquema**: https:// Indica el protocolo de comunicación a usar (HTTP o su versión segura, HTTPS).

**Subdominio**: www Es una subdivisión del dominio principal (aunque "www" es común, puede ser cualquier palabra como "blog" o "tienda").

**Dominio**: ejemplo.com El nombre único que identifica al sitio web.

**Puerto**: :443 Número que especifica la "puerta" de acceso en el servidor. Suele ser opcional porque los navegadores usan el puerto por defecto según el protocolo (80 para HTTP, 443 para HTTPS).

**Ruta**: /carpeta/pagina.html Indica la ubicación específica del recurso (como un archivo o página) dentro del servidor.

**Parámetros de Consulta (Query String)**: ?busqueda=hola Información adicional que se envía al servidor, normalmente precedida por ? y formada por pares clave=valor separados por &.

**Fragmento**: #intro Una referencia a una sección específica dentro de la misma página. El navegador se desplaza hasta ese punto, pero este componente no se envía al servidor.

## ¿Pasos en la recuperación de una página web mediante HTTP?

El proceso es una secuencia coordinada de pasos:

**Resolución DNS**: El navegador toma el nombre de dominio (ej: www.ejemplo.com) y consulta a un servidor DNS (Sistema de Nombres de Dominio) para traducirlo a una dirección IP numérica, que es la identificación real del servidor en la red.

**Establecimiento de Conexión TCP**: El navegador inicia un "apretón de manos" TCP (3-way handshake) con la dirección IP obtenida, en el puerto correspondiente (generalmente 80 o 443), para establecer una conexión confiable.

**Envío de la Solicitud HTTP**: Una vez conectado, el navegador envía un mensaje de petición HTTP. Este mensaje incluye el método (ej: GET), la ruta del recurso, la versión del protocolo y headers importantes como Host: www.ejemplo.com.

**Procesamiento y Respuesta del Servidor**: El servidor web recibe la solicitud, la procesa (busca el archivo, ejecuta un script, etc.) y genera una respuesta HTTP. Esta respuesta contiene un código de estado (ej: 200 OK si todo va bien), sus propios headers (información sobre la respuesta) y, si es exitosa, el cuerpo con el contenido solicitado (como el código HTML).

**Renderizado en el Navegador**: El navegador recibe la respuesta, interpreta el HTML, solicita recursos adicionales vinculados (como CSS, JavaScript e imágenes) mediante nuevas peticiones HTTP, y finalmente renderiza la página completa para el usuario.

**Cierre o Reutilización de la Conexión**: La conexión TCP se puede cerrar o, en versiones modernas de HTTP, mantenerse abierta (Connection: keep-alive) para agilizar la solicitud de recursos posteriores.

## Diferencia entre páginas dinámicas y estáticas

La diferencia fundamental radica en cuándo y cómo se genera el contenido que ve el usuario.

**Páginas Estáticas**: El contenido es fijo y preexistente. Cada usuario que visita la URL recibe exactamente el mismo archivo HTML almacenado en el servidor. Son fáciles de almacenar en caché y muy rápidas de servir. Son ideales para sitios de contenido que no cambia frecuentemente, como un folleto digital o un currículum en línea.

**Páginas Dinámicas**: El contenido se genera en el momento de la solicitud. El servidor ejecuta scripts (usando lenguajes como PHP, Python, Node.js) que pueden interactuar con bases de datos, APIs o información del usuario para producir una página HTML personalizada. Esto permite funcionalidades complejas como tiendas online, redes sociales o búsquedas, donde el contenido es único para cada usuario o situación.

## ¿Cómo usar telnet para acceder a un servidor web?

Para acceder a un servidor web utilizando Telnet, debes utilizar el comando telnet seguido de la dirección IP o el nombre de dominio del servidor web y el número de puerto correspondiente al servicio HTTP, que por defecto es el puerto 80. Por ejemplo, si el servidor web tiene la dirección IP 192.168.1.100, el comando sería telnet 192.168.1.100 80.
Si el puerto está abierto y el servicio está activo, Telnet establecerá la conexión y podrás ver una respuesta del servidor, como un mensaje de bienvenida o el encabezado HTTP, aunque no se mostrará el contenido de la página web como lo haría un navegador.

## Request. Métodos principales

Los métodos principales de petición HTTP son GET, POST, PUT, DELETE y HEAD, que constituyen la base de la comunicación entre el cliente y el servidor web.

El método **GET** se utiliza para solicitar una representación de un recurso específico, y las peticiones con este método solo deben recuperar datos sin modificar el estado del servidor.

El método **POST** se emplea para enviar una entidad a un recurso específico, provocando a menudo un cambio en el estado o efectos secundarios en el servidor, como el registro de información.

El método **PUT** reemplaza todas las representaciones actuales del recurso de destino con la carga útil de la petición, y es idempotente, lo que significa que realizar la misma solicitud múltiples veces tiene el mismo efecto que hacerlo una sola vez.

El método **DELETE** se utiliza para eliminar un recurso específico y también es idempotente.

Por último, el método **HEAD** es similar al GET, pero la respuesta no incluye el cuerpo de la respuesta, solo los encabezados, lo que permite verificar metadatos como el tamaño de un recurso sin descargarlo completamente.

Estos métodos son fundamentales en el diseño de aplicaciones web y APIs REST, permitiendo operaciones de lectura, creación, actualización y eliminación de recursos.

## Response. Códigos

Los códigos de respuesta HTTP son valores numéricos que un servidor devuelve al cliente para indicar el resultado de una solicitud realizada a través del protocolo HTTP. Estos códigos se agrupan en cinco clases principales según su primer dígito, cada una representando un tipo de respuesta general.

Hay distintas clases: 

**1xx**: Informativo (ej: 100 Continue).

**2xx**: Éxito (200 OK, 201 Created, 204 No Content).

**3xx**: Redirección (301 Moved Permanently, 304 Not Modified).

**4xx**: Error del cliente (400 Bad Request, 403 Forbidden, 404 Not Found).

**5xx**: Error del servidor (500 Internal Server Error, 503 Service Unavailable).

## Content type. Tipos principales

Los tipos principales de Content-Type, también conocidos como MIME types (Multipurpose Internet Mail Extensions), se clasifican según el tipo de contenido que representan. Los tipos más comunes incluyen:

**Text**: Usados para texto plano, HTML, CSS y JavaScript. Ejemplos son text/plain, text/html, text/css y text/javascript.

**Image**: Para formatos de imágenes como JPEG, PNG, GIF y SVG. Ejemplos incluyen image/jpeg, image/png, image/gif y image/svg+xml.

**Audio y Video**: Para archivos de audio y video. Ejemplos son audio/mpeg, audio/wav, video/mp4, video/quicktime y video/webm.

**Application**: Para datos binarios y documentos como JSON, XML, PDF y archivos de aplicaciones. Ejemplos son application/json, application/xml, application/pdf, application/zip y application/octet-stream.

**Multipart**: Usado para datos divididos en partes, común en cargas de archivos. Ejemplos incluyen multipart/form-data y multipart/byteranges.

**VND**: Un subtipo especial para formatos de documentos de aplicaciones específicas, como application/vnd.ms-excel, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet y application/vnd.oasis.opendocument.text.

Estos tipos se especifican en el encabezado HTTP Content-Type y son fundamentales para que los servidores y navegadores interpreten correctamente los datos transmitidos.