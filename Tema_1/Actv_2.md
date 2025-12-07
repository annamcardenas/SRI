# Actividad #2
# Pon en marcha el servidor Apache y lleva a cabo los siguientes cambios en el archivo de configuración.

## 1. Añadimos el puerto 81 en el archivo ports dentro de apache2

Ponemos Listen 81 en el archivo ports.conf

<img width="1086" height="689" alt="image" src="https://github.com/user-attachments/assets/503c3d91-0007-4849-83dd-b5d6ad2d6d1e" />

Comprobamos que funciona el puerto 81:

<img width="1084" height="686" alt="image" src="https://github.com/user-attachments/assets/e8d71809-fe8a-48e5-8ee2-5beee5eddfd6" />

# 2. Añadir el dominio marisma.intranet en el fichero host

Con sudo nano hosts en el directorio de /etc y escribimos 127.0.0.1 marisma.intranet

<img width="1068" height="683" alt="image" src="https://github.com/user-attachments/assets/99d315ea-3b4f-4481-b374-0d7a05b4bb26" />

Comprobamos que funciona:

<img width="1037" height="653" alt="image" src="https://github.com/user-attachments/assets/8ebda353-ac9b-460a-a8cd-bcd798d148cf" />

# 3. Cambiar la directiva Servertokens para mostar el nombre del producto

Cambiamos el ServerTokens OS por ServerTokens Prod

<img width="1076" height="686" alt="image" src="https://github.com/user-attachments/assets/fe866c87-f4a0-47df-910b-750d8025cb34" />

Con curl comprobamos que en el server pone Apache.

<img width="1093" height="680" alt="image" src="https://github.com/user-attachments/assets/7bd3e565-7ef0-4b1b-a361-4de16cfc855c" />

# 4. Cambia el valor de la directiva “ServerSignature” y comprueba que funciona correctamente. 

Le ponemos el ServerSignature en Off para que no salga en el localhost lo de la firma deapache.

<img width="1089" height="684" alt="image" src="https://github.com/user-attachments/assets/f83a7a16-11dd-4682-a9e6-28617a27eb49" />

Asi se ve con la firma:

<img width="840" height="401" alt="image" src="https://github.com/user-attachments/assets/f685f610-90ba-4547-b18a-73ddb5f8e189" />

Y aqui con los cambios que hemos hecho para que no aparezca la firma:

<img width="696" height="321" alt="image" src="https://github.com/user-attachments/assets/b9f06e09-fcfe-4d4e-b4d3-44dda9a76935" />

# 5. Crea un directorio “prueba” y otro directorio “prueba2”. Incluye un par de páginas en cada una de ellas.

Creamos los directorios con mkdir

vamos a var/www/html

<img width="590" height="361" alt="image" src="https://github.com/user-attachments/assets/35e65091-75fb-4e78-8c01-c0d497fe6557" />

Y creamos dos archivos html en cada directorio.

<img width="1095" height="687" alt="image" src="https://github.com/user-attachments/assets/b4bea373-e31b-4e8e-ba7f-39e8834f68c2" />

# 6. Redirecciona el contenido de la carpeta “prueba” hacia “prueba2”

Redireccionamos el contenido de la carpeta en etc/apache2 vamos al archivo apache2.conf

<img width="1091" height="686" alt="image" src="https://github.com/user-attachments/assets/3f531ad7-536e-4dee-bce9-94fd373ba9af" />

Escribimos abajo redirect /prueba /prueba2

<img width="1078" height="682" alt="image" src="https://github.com/user-attachments/assets/d72319bf-c040-4336-ac87-2d27ca2cdfda" />

Ya podemos acceder a los directorios

<img width="885" height="477" alt="image" src="https://github.com/user-attachments/assets/7d403310-c3f4-479b-b4cf-b8c6e8d5f999" />

# 7. Es posible redireccionar tan solo una página en lugar de toda la carpeta.

<img width="1082" height="680" alt="image" src="https://github.com/user-attachments/assets/31ddef6a-8700-4d6c-97f2-007f9064cd82" />

Creamos un archivo test.html dentro de prueba2

<img width="1090" height="692" alt="image" src="https://github.com/user-attachments/assets/69c5534f-61d1-49a7-91ba-99cfbe79a042" />

Hacemos una redireccion a un archivo especifico dentro de una carpeta.

<img width="1081" height="681" alt="image" src="https://github.com/user-attachments/assets/5faff723-93b7-4f78-899e-b3a8521b83d4" />

Comprobamos que se ha redireccionado correctamente

<img width="791" height="325" alt="image" src="https://github.com/user-attachments/assets/87f6f1c0-4414-4b05-82d7-ad7dc67aa928" />

# 8. Usa la directiva userdir

Primero se habilita el módulo userdir con el comando "sudo a2enmod userdir"

<img width="1093" height="687" alt="image" src="https://github.com/user-attachments/assets/0ec1f784-2f7a-490c-a165-86215e718c3d" />

Vamos al directorio de nuestro usuario (ana) y creamos una carpeta llamada public_html y dentro creamos un archivo index.html

<img width="694" height="502" alt="image" src="https://github.com/user-attachments/assets/494e24b6-348f-48fd-af14-f1b7b6b651e2" />

El contenido de index.html

<img width="696" height="497" alt="image" src="https://github.com/user-attachments/assets/5c72c3c4-665b-494c-be1a-df9e04498e0e" />

Una vez creado el archivo, procedemos a cambiarle los permisos al directorio personal (ana) con el comando chmod 755 ana/

<img width="687" height="491" alt="image" src="https://github.com/user-attachments/assets/2c70ecd0-e701-411f-886d-a05b06d7d06e" />

Comprobamos en el navegador mediante localhost/~ana/ se muestra correctamente la pagina de index.html que hemos creado anteriormente.

<img width="927" height="463" alt="image" src="https://github.com/user-attachments/assets/53ba03f4-0585-4bf3-961e-780d1fe06f97" />

# 9. Usa la directiva alias para redireccionar a una carpeta dentro del directorio de usuario.

Primero accedemos a etc/apache2 y abrimos el archivo de configuración apache2.conf

<img width="691" height="495" alt="image" src="https://github.com/user-attachments/assets/9b551f3c-d502-4764-86b8-fe998050663a" />

Escribimos en el archivo el Alias que apunta a la ruta que hemos creado anteriormente. Guardamos el archivo y lo cerramos.

<img width="685" height="492" alt="image" src="https://github.com/user-attachments/assets/2af3036d-464f-4d41-9f9e-7909390ad7d5" />

Vamos a crear otro archivo, por ejemplo un archivo de texto, dentro de la carpeta que hemos creado (Musiquita). Recargamos apache2

<img width="688" height="491" alt="image" src="https://github.com/user-attachments/assets/ff6d52dd-c55b-42e1-8510-33acb75d507f" />

Ahora para comprobar que ha funcionado la redirección vamos al navegador y escriimos localhost/Musiquita/

Aparecerá como se muestra en la imagen.

<img width="910" height="325" alt="image" src="https://github.com/user-attachments/assets/d9c92171-5dfa-41f5-abb2-a95b1bda8cac" />

10. ¿Para qué sirve la directiva Options y dónde aparece. Comprueba si apache indexa los directorios. Si es así, ¿cómo lo desactivamos?

La directiva Options de Apache sirve para activar o desactivar características específicas en un directorio. Permite controlar funciones como el seguimiento de enlaces simbólicos (FollowSymLinks), la generación automática de listados de directorios (Indexes) o la ejecución de scripts CGI (ExecCGI). Esta directiva se usa dentro de bloques como <Directory> o en archivos .htaccess para definir el comportamiento del servidor para esa sección del sitio web. 

Sintaxis: Options [+|-]opción1 [opción2] .... 

Contexto de uso: Se aplica en la configuración del servidor (server config), en hosts virtuales (virtual host), en directorios específicos (directory) o en archivos .htaccess

Vamos al archivo apache2.conf y eliminamos la palabra Indexez para desactivar el indexado del directorio.

<img width="699" height="501" alt="image" src="https://github.com/user-attachments/assets/75ea43d1-1040-4276-b2ed-1d618696f683" />

Aqui con el indexes eliminado.

<img width="697" height="505" alt="image" src="https://github.com/user-attachments/assets/5d4d77fc-cbeb-474e-aa2d-984e77773a47" />

Comprobamos que la redirección ya no funciona.

<img width="914" height="340" alt="image" src="https://github.com/user-attachments/assets/60e76153-e7ba-4db2-bb82-5cd797a1a4f7" />



