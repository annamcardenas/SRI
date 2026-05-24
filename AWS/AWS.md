# Práctica AWS: Despliegue de WordPress con VPC, RDS y EFS

## Introducción

En esta práctica se describe el proceso completo de despliegue de una aplicación WordPress en AWS, utilizando una arquitectura de red segmentada y servicios gestionados.

Los objetivos principales son:

- **Crear una VPC** con 2 subredes públicas y 2 privadas para simular un entorno realista.
- **Lanzar una instancia EC2** (Debian) en una subred pública para alojar el frontend (Apache + PHP).
- **Configurar una base de datos MySQL** gestionada con Amazon RDS, ubicada en las subredes privadas.
- **Implementar almacenamiento compartido** con Amazon EFS, montado en el directorio `wp-content` de WordPress.
- **Instalar y configurar WordPress** conectándolo a RDS y utilizando EFS para archivos multimedia.

## Creacion VPC

<img width="750" height="208" alt="image" src="https://github.com/user-attachments/assets/2c59247c-4b06-4138-be70-6afa7a81a832" />

---
Se asigna el bloque CIDR IPv4 10.2.0.0/16 y se deshabilita el IPv6. Se activa el etiquetado automático con el nombre proyecto_AWS

<img width="1303" height="485" alt="image" src="https://github.com/user-attachments/assets/d191e5f5-5ef1-4161-94f2-e00af58cc93d" />

---
Configuración de zonas de disponibilidad. Se eligen dos AZ para alta disponibilidad: eu-west-1a y eu-west-1b. La tenencia se mantiene como "Predeterminada".

<img width="448" height="378" alt="image" src="https://github.com/user-attachments/assets/52fecb25-9201-4c3c-8833-313e47b00149" />

---
Personalización de los bloques CIDR de las subredes. Se crean dos subredes públicas (10.2.0.0/24 y 10.2.1.0/24) y dos privadas (10.2.2.0/24 y 10.2.3.0/24), distribuidas entre las dos zonas de disponibilidad.

<img width="447" height="514" alt="image" src="https://github.com/user-attachments/assets/722a98a8-2d20-45cd-b1f6-2869fb6895c1" />

---
Se selecciona "Ninguna" para las puertas de enlace NAT, ya que no se necesitan por ahora. Los puntos de enlace de VPC también se dejan en "Ninguna". Se mantienen habilitadas las dos opciones de DNS: "Habilitar nombres de host DNS" y "Habilitar la resolución de DNS", necesarias para que las instancias puedan resolver nombres dentro de la VPC.
 
<img width="441" height="539" alt="image" src="https://github.com/user-attachments/assets/75583481-2530-4ee3-892b-f6cb5da9813d" />

---
Resumen del flujo de trabajo de creación de la VPC. Se muestran los recursos que se han creado automáticamente: la propia VPC con su ID, las cuatro subredes (dos públicas y dos privadas), una puerta de enlace de Internet (igw), y las tablas de enrutamiento correspondientes. Todo el proceso se ha completado correctamente.

<img width="590" height="522" alt="image" src="https://github.com/user-attachments/assets/7eab40d1-920f-4fae-ae38-6fe1680e5984" />


## Creación de instancias
Se inicia el asistente para lanzar una instancia EC2. Se asigna el nombre servidor_wordpress a la instancia. En la sección de sistema operativo, se selecciona la opción "Debian" dentro del catálogo de imágenes de máquina de Amazon (AMI).

<img width="888" height="535" alt="image" src="https://github.com/user-attachments/assets/071e7066-8418-41d1-9e4d-ea89dfe84d8d" />

---
Se elige el tipo de instancia t3.micro, que está dentro de la capa gratuita y dispone de 2 vCPUs y 1 GiB de memoria. También se selecciona el par de claves practica_AWS para poder conectarse por SSH de forma segura a la instancia una vez lanzada.

<img width="896" height="448" alt="image" src="https://github.com/user-attachments/assets/a80123f8-ea2e-45c9-9e62-45de7169eeb7" />

---

<img width="884" height="538" alt="image" src="https://github.com/user-attachments/assets/095aaef7-a84f-413e-a488-22ad383cd8d8" />

---
Se configuran dos reglas de entrada para el grupo de seguridad. La primera permite tráfico SSH (puerto 22) desde cualquier dirección IP (0.0.0.0/0) para poder conectarnos remotamente. La segunda permite tráfico HTTP (puerto 80) también desde cualquier IP, para que el servidor web sea accesible públicamente.

<img width="888" height="539" alt="image" src="https://github.com/user-attachments/assets/64653edf-183a-4758-b76a-f94a81b458fb" />

---
Se confirma que la instancia se ha lanzado correctamente con el ID i-0db5e4c77ec7002b7. El registro de lanzamiento muestra que todas las etapas se completaron con éxito: inicialización de solicitudes, creación de grupos de seguridad y reglas, e inicio del lanzamiento. 

<img width="430" height="412" alt="image" src="https://github.com/user-attachments/assets/46c561c2-099e-4731-a86d-3a159d10b9a8" />

---

<img width="998" height="539" alt="image" src="https://github.com/user-attachments/assets/28f24c3a-91d6-4827-a950-687cecbf92c6" />

---

<img width="1345" height="526" alt="image" src="https://github.com/user-attachments/assets/b0cb862e-0d7b-4a3d-970f-a8fa21de4022" />

---

### Nos conectamos por ssh
Desde el terminal local se establece la conexión SSH a la instancia EC2 utilizando el par de claves practica_AWS.pem y el usuario admin.

<img width="886" height="174" alt="image" src="https://github.com/user-attachments/assets/9c21ce0f-d072-466e-8ca8-96662dce252f" />

---

## Instalación de Apache y PHP
Se inicia el servicio de Apache con sudo systemctl start apache2 y se configura para que arranque automáticamente al lanzar la instancia con sudo systemctl enable apache2. A continuación, se accede a la IP pública de la instancia desde un navegador, mostrando la página por defecto de Apache ("It works!"), lo que verifica que el servidor web está correctamente instalado y funcionando.

<img width="819" height="319" alt="image" src="https://github.com/user-attachments/assets/7fb4a695-2940-4c59-b933-f93a667a9f72" />

---

<img width="898" height="83" alt="image" src="https://github.com/user-attachments/assets/9a29132c-7482-45f0-8873-ac636adf972d" />

---
Se instalan los paquetes necesarios para ejecutar PHP en el servidor: php, libapache2-mod-php (módulo de PHP para Apache) y php-cli (interfaz de línea de comandos). La instalación descarga e instala PHP 8.4.21 junto con sus dependencias. Finalmente, se ejecuta php -v para comprobar que PHP está correctamente instalado y se muestra la versión confirmando el éxito de la instalación.

<img width="1108" height="716" alt="image" src="https://github.com/user-attachments/assets/38c1dae8-2d69-4fe9-baf0-8bfccc899c1c" />

---
Se instala el paquete php-mysql, que permite a PHP conectarse y comunicarse con bases de datos MySQL. Como dependencia se instala también php8.4-mysql. Este paso es necesario para que WordPress pueda interactuar con la base de datos RDS que se creará más adelante.

<img width="967" height="261" alt="image" src="https://github.com/user-attachments/assets/e260d32f-c25f-4ff4-8b9f-28bc3d5e4e39" />

---

<img width="562" height="111" alt="image" src="https://github.com/user-attachments/assets/614c86cc-67b2-4efc-af17-1d0fed68e25d" />

---

<img width="599" height="305" alt="image" src="https://github.com/user-attachments/assets/4e858ca2-dc5d-4df7-b3f8-74440ade6bdf" />

---


<img width="418" height="80" alt="image" src="https://github.com/user-attachments/assets/2216fe5d-39c7-4032-a457-a1f492e02b2c" />

---

## Creación de base de datos en RDS
Se inicia la creación de una base de datos en el servicio RDS. Se asigna el identificador bbdd-wordpress a la instancia de base de datos. El nombre de usuario maestro se establece como admin y se elige la opción "Autoadministrado".

<img width="1344" height="153" alt="image" src="https://github.com/user-attachments/assets/c5b4df60-63d5-42d2-8f1f-18af324bb841" />

---
Se elige el motor de base de datos. Se selecciona "Configuración completa" para tener control sobre todos los parámetros. Entre las opciones disponibles, se marca MySQL como el motor a utilizar, que será compatible con WordPress.

<img width="1319" height="536" alt="image" src="https://github.com/user-attachments/assets/9f6b313a-f781-4933-a556-d2122906049a" />

---
Se configura la clase de instancia para la base de datos. Se elige el tipo db.t4g.micro, que dispone de 2 vCPUs y 1 GiB de RAM. Esta instancia está dentro de la capa gratuita y es suficiente para los requisitos de la práctica.

<img width="1349" height="540" alt="image" src="https://github.com/user-attachments/assets/5ef99fff-e210-469d-9c4b-3a354305968c" />

---
Se configura el almacenamiento de la base de datos. Se asignan 20 GiB de capacidad, que es el mínimo permitido. Las IOPS se establecen en 3000 y el rendimiento en 125 MiB/s, valores por defecto adecuados para esta práctica.

<img width="993" height="418" alt="image" src="https://github.com/user-attachments/assets/b69271b8-3625-447a-8a20-7ea61a125368" />

---
Se configura la conectividad de la base de datos. Se selecciona la VPC proyecto_AWS-vpc creada anteriormente. El acceso público se establece en "No" por seguridad, ya que la base de datos solo debe ser accesible desde recursos dentro de la VPC (como la instancia EC2). Se creará un nuevo grupo de seguridad específico para la base de datos.

<img width="1007" height="511" alt="image" src="https://github.com/user-attachments/assets/ced80821-d447-4573-b0a8-3db3bfc3d4fb" />

---
Se define el grupo de seguridad de VPC para la base de datos. Se elige la opción "Crear nuevo" y se asigna el nombre Seguridad-wordpress. Este grupo de seguridad controlará qué tráfico puede llegar a la base de datos.

<img width="1337" height="515" alt="image" src="https://github.com/user-attachments/assets/345e41d6-e975-4e6a-abe2-c681d3716a0e" />

---

<img width="1338" height="508" alt="image" src="https://github.com/user-attachments/assets/f062e22d-4a9b-45b7-a055-a28cd58af156" />

---
En la configuración adicional se especifica el nombre de la base de datos inicial: bbdd_wordpress. Se mantienen los valores por defecto para el grupo de parámetros (default.mysql8.4) y el grupo de opciones (default-mysql-8-4). 

<img width="981" height="519" alt="image" src="https://github.com/user-attachments/assets/030f7de0-d139-474f-829e-3b62d09d6bb9" />

---

### Dejamos creando la base de datos

Se muestra el estado de creación de la base de datos bbdd-wordpress. Inicialmente aparece con estado "Creando", y posteriormente cambia a "Configuring-enhanced..." mientras RDS aplica la configuración adicional. La instancia seleccionada es db.t4g.micro con motor MySQL Community. Este proceso puede tardar varios minutos en completarse.

<img width="1361" height="298" alt="image" src="https://github.com/user-attachments/assets/17a19d24-32d3-4750-a1ab-5ddddbe2478c" />

---

<img width="1136" height="267" alt="image" src="https://github.com/user-attachments/assets/21ababc1-9b08-4089-99c9-3ba2b968afd2" />

---
RDS proporciona un asistente para facilitar la conexión entre la instancia EC2 y la base de datos. Se selecciona la instancia servidor_wordpress (i-0db5e4c77ec7002b7), que se encuentra en la misma VPC que la base de datos. El asistente configurará automáticamente los permisos necesarios en los grupos de seguridad.

<img width="1358" height="444" alt="image" src="https://github.com/user-attachments/assets/f2930f2c-62eb-4b06-9eeb-1babb1c713a6" />

---
Se confirma que la conexión entre la base de datos RDS bbdd-wordpress y la instancia EC2 se ha establecido correctamente. 

<img width="1359" height="291" alt="image" src="https://github.com/user-attachments/assets/a608c6fd-e78d-4a7b-80e9-76c91d115730" />

---

## Creación de un EFS
Se accede al servicio Amazon Elastic File System (EFS) desde la consola de AWS. EFS proporciona un sistema de archivos NFS elástico y escalable que puede ser montado por múltiples instancias EC2 simultáneamente. Se pulsa el botón "Crear un sistema de archivos" para comenzar la configuración.

<img width="1277" height="258" alt="image" src="https://github.com/user-attachments/assets/3ce9ceaf-4a00-40db-b563-13ea1346d0cb" />

---
Se crea un nuevo sistema de archivos EFS con el nombre almacenamiento-wordpress. Se selecciona la VPC proyecto_AWS-vpc para que sea accesible desde la instancia EC2. 

<img width="669" height="596" alt="image" src="https://github.com/user-attachments/assets/a2bfbd7b-ba76-4cad-855f-c854bd0fffc3" />

---
Se editan las reglas de entrada del grupo de seguridad de la instancia EC2.

<img width="1361" height="394" alt="image" src="https://github.com/user-attachments/assets/14ad39df-1a58-4d2c-97cc-6bac2bab9541" />

---
El asistente de EFS proporciona los comandos necesarios para montar el sistema de archivos en la instancia.

<img width="1365" height="479" alt="image" src="https://github.com/user-attachments/assets/98c14a0e-da4b-442f-b4fc-a067381618f5" />

---

<img width="1342" height="476" alt="image" src="https://github.com/user-attachments/assets/21253d99-372e-48f8-908d-9385fa4def62" />

---
Se ejecuta el comando mount para montar el sistema de archivos EFS en el directorio /home/admin/efs de la instancia. A continuación, se utiliza df -h para comprobar que el montaje se ha realizado correctamente. Se puede observar que el EFS aparece montado con origen 10.2.0.215:/ y un tamaño muy grande (8.0E), lo que confirma su naturaleza elástica.

<img width="975" height="302" alt="image" src="https://github.com/user-attachments/assets/7b7920cc-2857-4cc6-afd0-bc35280d7b65" />

---

## Instalación de Wordpress
Desde el directorio /var/www/html, se descarga el paquete de WordPress utilizando wget http://wordpress.org/latest.tar.gz. La descarga se realiza correctamente y el archivo latest.tar.gz de aproximadamente 28 MB se guarda en el directorio.

<img width="965" height="284" alt="image" src="https://github.com/user-attachments/assets/01303f8e-6e4a-4bbc-92fc-3312b02e822f" />

---
Se descomprime el archivo latest.tar.gz con sudo tar -xf, generando el directorio wordpress con todos los archivos necesarios. También se instala el cliente de MySQL (default-mysql-client), que incluye el comando mysql para poder conectarse a la base de datos.

<img width="956" height="374" alt="image" src="https://github.com/user-attachments/assets/7a169c9d-9844-49db-8c57-889db5d00b2e" />

---
Desde la instancia EC2, se conecta a la base de datos RDS utilizando el cliente MySQL. Se especifica el endpoint de la base de datos (bbdd-wordpress.cti604ac8tfx.eu-west-1.rds.amazonaws.com), el puerto 3306, el usuario admin y el certificado SSL para la conexión segura. Una vez conectado, se ejecutan las sentencias SQL para crear la base de datos wordpress, el usuario wp_user con contraseña admin, y se le conceden todos los privilegios sobre la base de datos.

<img width="967" height="215" alt="image" src="https://github.com/user-attachments/assets/6faa5a7c-22d5-41fb-8ad0-ef8c549c7434" />

---

<img width="596" height="215" alt="image" src="https://github.com/user-attachments/assets/4b7958d4-3460-4244-9408-b1f54f8fb5a1" />

---
Se accede desde el navegador a la URL http://54.217.68.53/wordpress/wp-admin/setup-config.php. Aparece la pantalla de bienvenida del asistente de configuración de WordPress, que solicita los datos necesarios para la conexión con la base de datos: nombre de la base de datos, usuario, contraseña, host y prefijo de las tablas.

<img width="1082" height="672" alt="image" src="https://github.com/user-attachments/assets/f86c0371-de0e-4829-a9a0-1e59f3913d75" />

---
Se introducen los datos de conexión a la base de datos en el formulario. El nombre de la base de datos es wordpress, el usuario es wp_user, la contraseña es la establecida anteriormente, y el host es el endpoint de RDS (bbdd-wordpress.cti604ac8tfxe.eu-west-1.rds.amazonaws.com). El prefijo de las tablas se deja como wp_ por defecto.

<img width="1051" height="703" alt="image" src="https://github.com/user-attachments/assets/2e4bc88c-6d1f-465f-86d2-d0a2b4650ef3" />

---
Una vez validada la conexión con la base de datos, se procede a configurar el sitio. Se establece el título del sitio como prueba-aws, el usuario administrador como admin, se genera una contraseña segura automáticamente, y se introduce un correo electrónico de contacto. 

<img width="990" height="718" alt="image" src="https://github.com/user-attachments/assets/52e7a343-fb21-40ca-8b8f-d8dce074f631" />

---
Se confirma que WordPress se ha instalado correctamente. Se muestra un resumen con el nombre de usuario (admin) y se informa que la contraseña es la que se eligió en el paso anterior.

<img width="998" height="443" alt="image" src="https://github.com/user-attachments/assets/436c17f0-6472-4fe1-bfbd-6466671ec4b6" />

## Conexión de EFS a directorio WP-Content
Se accede al directorio de instalación de WordPress (/var/www/html/wordpress). Primero se renombra el directorio wp-content original a wp-content-old para hacer una copia de seguridad. A continuación, se monta el sistema de archivos EFS en la nueva ubicación wp-content, utilizando la dirección 10.2.0.215:/wp-content. Finalmente, se copia el contenido de la copia de seguridad al nuevo directorio montado y se asigna la propiedad a www-data, el usuario del servidor web.

<img width="659" height="57" alt="image" src="https://github.com/user-attachments/assets/6cca1366-7ce7-4521-8b60-3fc402c21a0a" />

---

<img width="972" height="65" alt="image" src="https://github.com/user-attachments/assets/770667cd-dec8-4897-bca1-f82cbcacfa16" />

---

<img width="968" height="64" alt="image" src="https://github.com/user-attachments/assets/216141b9-0397-4d26-a522-a571b947a4b8" />

---
Se ejecuta df -h para comprobar que el sistema de archivos EFS está correctamente montado. Se puede observar que el EFS aparece montado en /var/www/html/wordpress/wp-content con origen 10.2.0.215:/ y un tamaño elástico de 8.0E (prácticamente ilimitado). Esto confirma que los archivos multimedia y plugins de WordPress se almacenarán ahora en el EFS, permitiendo persistencia y escalabilidad.

<img width="679" height="285" alt="image" src="https://github.com/user-attachments/assets/7550c9c1-8e0f-4e08-8020-7360481a2696" />


