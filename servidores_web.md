# Práctica Servidores Web - 1º Trimestre
## Instalación y configuración de servidor web Apache con WordPress y aplicación Python

## 1. Introducción

En esta práctica vamos a configurar un servidor web Apache para un instituto, implementando:
- Dos dominios virtuales: `centro.intranet` (WordPress) y `departamento.centro.intranet` (Python)
- Instalación de WordPress con MySQL
- Aplicación Python usando WSGI
- Autenticación básica para la aplicación Python
- Instalación de AWStats
- Segundo servidor web (Nginx) en puerto 8080 con phpMyAdmin

**Entorno utilizado:** Ubuntu Desktop en máquina virtual Proxmox

## 2. Preparación del entorno

### 2.1 Actualización del sistema

sudo apt update && sudo apt upgrade -y

sudo apt install curl wget vim git -y

<img width="706" height="271" alt="image" src="https://github.com/user-attachments/assets/b95a4ccf-4f77-41cb-b186-bc734be8230b" />

Instalamos Apache2

sudo apt install apache2 -y

sudo systemctl start apache2

sudo systemctl enable apache2

<img width="694" height="107" alt="image" src="https://github.com/user-attachments/assets/6e3855c8-bfba-45a5-bd82-d45fcbb21571" />

Verificamos que el servicio esta activo

sudo systemctl status apache2

<img width="654" height="82" alt="image" src="https://github.com/user-attachments/assets/21981be2-962e-4c58-8f29-57ef99f2fb50" />

<img width="710" height="346" alt="image" src="https://github.com/user-attachments/assets/012e1cc6-fa8a-4054-b23f-7d609aa0bf50" />

Comprobamos en el navegador que apache esta funcionando.

Abrimos el navegador Firefox y navegamos a http://localhost.

<img width="1121" height="694" alt="image" src="https://github.com/user-attachments/assets/1dc1baf1-bb02-495f-a76d-991ceaae245e" />

## 2.2. Configurar VirtualHosts en apache

Los Virtual Hosts nos permiten alojar múltiples sitios web en un único servidor Apache. 
En nuestro caso, necesitamos dos dominios principales:

- centro.intranet para WordPress
  
- departamento.centro.intranet para la aplicación Python

Cada Virtual Host requiere su propio directorio raíz donde se almacenarán los archivos del sitio web. 
Creamos una estructura organizada que facilite el mantenimiento:

Crear directorios para cada dominio.

sudo mkdir -p /var/www/centro.intranet/public_html

sudo mkdir -p /var/www/departamento.centro.intranet/public_html

<img width="707" height="229" alt="image" src="https://github.com/user-attachments/assets/1c5bc0b3-5282-44f6-a7a5-45a6bb569a6a" />

Asignamos permisos adecuados:

sudo chown -R $USER:$USER /var/www/centro.intranet/public_html

sudo chown -R $USER:$USER /var/www/departamento.centro.intranet/public_html

sudo chmod -R 755 /var/www/

chown -R $USER:$USER: Establece al usuario actual como propietario, facilitando la gestión de archivos

chmod -R 755: Permisos que permiten lectura y ejecución para todos, pero escritura solo para el propietario

Estructura 755: Propietario (7: rwx), Grupo (5: r-x), Otros (5: r-x)

<img width="1014" height="210" alt="image" src="https://github.com/user-attachments/assets/59025d86-b988-442a-a693-ac94d536ba21" />

Para poder acceder a nuestros dominios virtuales desde el mismo servidor, 
debemos configurar la resolución de nombres local: 

sudo nano /etc/hosts

Y añadimos las siguientes entradas:

127.0.0.1    centro.intranet

127.0.0.1    departamento.centro.intranet

127.0.0.1    servidor2.centro.intranet

El archivo /etc/hosts actúa como un sistema de resolución de nombres local, priorizando estas entradas sobre los servidores DNS externos. 

Al apuntar a 127.0.0.1 (localhost), todas las peticiones a estos dominios se redirigen al servidor Apache local.

<img width="681" height="297" alt="image" src="https://github.com/user-attachments/assets/035f19d7-659b-4011-9753-acf92058e4b6" />

Para confirmar que la configuración del archivo hosts funciona correctamente, utilizamos el comando ping:

ping -c 3 centro.intranet

ping -c 3 departamento.centro.intranet

ping -c 3 servidor2.centro.intranet

Cada comando debería mostrar respuestas desde 127.0.0.1, confirmando que los nombres se resuelven correctamente a la dirección local.

<img width="1018" height="285" alt="image" src="https://github.com/user-attachments/assets/3fb8f278-6804-467a-8f64-c118b9525774" />

<img width="1011" height="305" alt="image" src="https://github.com/user-attachments/assets/bd1d935a-320e-4728-ad58-06a9a23fd948" />

<img width="1020" height="308" alt="image" src="https://github.com/user-attachments/assets/441fa775-39ab-4dc8-9ee7-fd9c40994e7b" />

## 3. Instalación de la pila LAMP

LAMP es el acrónimo de Linux, Apache, MySQL y PHP, una combinación estándar para servidores web dinámicos. 
En nuestro caso, como ya tenemos Apache instalado, procedemos con MySQL y PHP.

### 3.1 Instalación de MySQL y PHP
MySQL proporciona el sistema de gestión de bases de datos necesario para WordPress, 
mientras que PHP es el lenguaje de programación que ejecuta WordPress y otras aplicaciones web.

sudo apt install mysql-server php libapache2-mod-php php-mysql php-gd php-xml php-mbstring -y

<img width="1021" height="245" alt="image" src="https://github.com/user-attachments/assets/e90a55bf-eba4-4f19-b3db-8ce6eb3b647d" />

<img width="1018" height="308" alt="image" src="https://github.com/user-attachments/assets/c5f0c2eb-8025-4efd-9f73-2378af899434" />

### Componentes instalados:

mysql-server: Servidor de base de datos MySQL

php: Intérprete del lenguaje PHP

libapache2-mod-php: Módulo para integrar PHP con Apache

php-mysql: Extensión para conectar PHP con MySQL

php-gd: Biblioteca de procesamiento de imágenes

php-xml: Soporte para XML

php-mbstring: Soporte para cadenas multibyte (necesario para WordPress)

### 3.2. Comprobamos que se ha instalado correctamente y activamos el servicio de apache.

Tras la instalación, es fundamental verificar que todos los componentes funcionan correctamente y están configurados para iniciarse automáticamente.

Verificar versión de PHP instalada:

php --version

Asegurar que Apache se inicia automáticamente:

sudo systemctl enable apache2

<img width="1026" height="173" alt="image" src="https://github.com/user-attachments/assets/ece98024-8569-4533-b0d9-8e9ed20378d9" />

<img width="1008" height="519" alt="image" src="https://github.com/user-attachments/assets/67e42247-ed86-455b-8461-9ec741d2577e" />

Versión de PHP: Confirma que la instalación fue exitosa

Estado de Apache: Asegura que el servidor web esté activo y configurado para arranque automático

## 4. Instalación y configuración de WordPress

WordPress es el sistema de gestión de contenidos (CMS) más popular del mundo.

Lo instalaremos en el dominio centro.intranet para gestionar el contenido principal del instituto.

### 4.1 Configuración de la Base de Datos MySQL

WordPress requiere una base de datos MySQL para almacenar todo su contenido, configuraciones y datos de usuarios.

Acceso al servidor MySQL:

sudo mysql -u root -p

<img width="660" height="295" alt="image" src="https://github.com/user-attachments/assets/4ed8627e-eb7a-423f-bb36-1bc8a7366eb7" />

Dentro de mysql escribimos estos comandos:

CREATE DATABASE wordpress_centro;

CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'contraseña_segura';

GRANT ALL PRIVILEGES ON wordpress_centro.* TO 'wpuser'@'localhost';

FLUSH PRIVILEGES;

EXIT;

<img width="631" height="269" alt="image" src="https://github.com/user-attachments/assets/b7f9599d-8951-4fa2-9d50-ff91bfc8ef84" />

Primero creamos la base de datos con CREATE DATABASE, después creamos al usuario con CREATE USER,
le damos privilegios con GRANT ALL PRIVILEGES ON y aplicamos cambios con FLUSH PRIVILEGES.
Para salir escribimos EXIT al final.

### 4.2. Descargamos Wordpress.

Vamos al diccionario de centro.intranet y descargamos wordpress.

cd /var/www/centro.intranet/public_html

sudo wget https://wordpress.org/latest.tar.gz

<img width="1008" height="294" alt="image" src="https://github.com/user-attachments/assets/abdb9ff5-603d-4e28-937c-7df346e2e7ce" />

Descomprimimos.

sudo tar -xzvf latest.tar.gz

Este comando extrae todos los archivos de WordPress en un directorio llamado wordpress.

<img width="900" height="235" alt="image" src="https://github.com/user-attachments/assets/f9de7222-5aa0-4c6a-b9e7-811b5ea27a03" />

Movemos los archivos al directorio actual y borramos la carpeta wordpress que quedará vacía.

sudo mv wordpress/* .

sudo rmdir wordpress

sudo rm latest.tar.gz

<img width="850" height="122" alt="image" src="https://github.com/user-attachments/assets/ec74d3f5-4dd3-4f50-97ca-700801cef172" />

Damos los permisos necesarios.

Para que Apache pueda acceder y modificar los archivos de WordPress:

sudo chown -R www-data:www-data /var/www/centro.intranet/public_html

sudo find /var/www/centro.intranet/public_html -type d -exec chmod 755 {} \;

sudo find /var/www/centro.intranet/public_html -type f -exec chmod 644 {} \;

Propietario www-data: Usuario bajo el cual corre Apache

Directorios 755: Permite a Apache listar y acceder a directorios

Archivos 644: Permite lectura por Apache, escritura solo por propietario

<img width="1014" height="214" alt="image" src="https://github.com/user-attachments/assets/f7082618-013b-4025-826c-3ba1d4a0d39d" />

Creamos archivos de configuración.

sudo nano /etc/apache2/sites-available/centro.intranet.conf

Para centro.intranet (WordPress)

<VirtualHost *:80>
    ServerName centro.intranet
    ServerAlias www.centro.intranet
    DocumentRoot /var/www/centro.intranet/public_html
    
    <Directory /var/www/centro.intranet/public_html>
        AllowOverride All
        Require all granted
        Options Indexes FollowSymLinks
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/centro_error.log
    CustomLog ${APACHE_LOG_DIR}/centro_access.log combined
</VirtualHost>

- ServerName: Nombre principal del dominio

- ServerAlias: Alias adicionales (incluye versión con www)

- DocumentRoot: Ubicación de los archivos del sitio

- AllowOverride All: Permite el uso de archivos .htaccess

- Require all granted: Permite acceso a todos los visitantes

<img width="1011" height="79" alt="image" src="https://github.com/user-attachments/assets/1445f935-9fc4-4126-a5d3-f7aa3c4c4feb" />

<img width="780" height="346" alt="image" src="https://github.com/user-attachments/assets/45c4e90b-9fca-4651-855a-618d0f79d659" />

Y para departamento.centro.intranet (Python)

Creamos la configuración para el dominio que alojará la aplicación Python:

sudo nano /etc/apache2/sites-available/departamento.centro.intranet.conf

<img width="1042" height="42" alt="image" src="https://github.com/user-attachments/assets/d9c78505-38d8-480d-b45e-4ca25efb9500" />

<img width="816" height="405" alt="image" src="https://github.com/user-attachments/assets/f71a579d-8708-4b1a-9f71-cf6820d50637" />

Confirmamos que se hayan creado

ls -la /etc/apache2/sites-available/

<img width="955" height="177" alt="image" src="https://github.com/user-attachments/assets/e36a8b49-dd45-41d2-9212-dadac197d1b2" />

Desactivamos el sitio por defecto para que apache no tenga dos configuraciones para el puerto 80.

sudo a2dissite 000-default.conf

<img width="919" height="100" alt="image" src="https://github.com/user-attachments/assets/8fd98d1a-adec-4c21-8b1d-3f2ff77836be" />

Activamos nuestros sitios.

sudo a2ensite centro.intranet.conf

sudo a2ensite departamento.centro.intranet.conf

<img width="1019" height="202" alt="image" src="https://github.com/user-attachments/assets/3a912194-20c9-4041-bef4-eb512d95400f" />

Activamos el módulo rewrite.

WordPress requiere el módulo de reescritura de URLs para funcionar correctamente:

sudo a2enmod rewrite

<img width="826" height="111" alt="image" src="https://github.com/user-attachments/assets/3c057a26-fc22-4717-9069-9c6d24574c51" />

Recargamos Apache:

sudo systemctl reload apache2

<img width="684" height="43" alt="image" src="https://github.com/user-attachments/assets/a0842ef3-2a66-4157-859e-28052f39a717" />

En el navegador comprobamos si funciona buscando centro.intranet

Abrimos el navegador y navegamos a http://centro.intranet para iniciar el proceso de instalación web.

<img width="1088" height="673" alt="image" src="https://github.com/user-attachments/assets/21615322-457e-4d70-b7e0-4f139eac689d" />

Seleccionamos el idioma y procedemos a rellenar el formulario para conectarnos a la base de datos.

<img width="1067" height="666" alt="image" src="https://github.com/user-attachments/assets/1bb76efc-4a03-4fef-a2dd-f5698fc3fdc1" />

Ponemos los datos que correspondan.

<img width="1014" height="639" alt="image" src="https://github.com/user-attachments/assets/c4827877-3889-4e6a-a712-984eb52926a8" />

<img width="915" height="349" alt="image" src="https://github.com/user-attachments/assets/d63a4a02-96a1-436e-a747-aa2df95e4ab4" />

Rellenamos con la información correspondiente.

<img width="966" height="637" alt="image" src="https://github.com/user-attachments/assets/fb9d19d9-dee2-4514-b4e6-015c9c3765c9" />

<img width="738" height="463" alt="image" src="https://github.com/user-attachments/assets/4af7af12-e82f-4f2f-82bf-48a37054eda3" />

Comprobamos que se ha configurado correctamente.

<img width="1067" height="659" alt="image" src="https://github.com/user-attachments/assets/a92f5ac1-1e49-48b8-934d-10951d59c935" />

### 5. Aplicación Python con WSGI

Instalamos el módulo WSGI para Python 3 y verificamos su instalación.

<img width="992" height="449" alt="image" src="https://github.com/user-attachments/assets/8ceb2353-9128-4ca0-a392-5e7c36d706d8" />

Creamos el directorio y el archivo para la aplicación Python

<img width="1018" height="60" alt="image" src="https://github.com/user-attachments/assets/27432a89-f81d-4cd3-8881-f8277632e413" />

<img width="695" height="42" alt="image" src="https://github.com/user-attachments/assets/4d67a602-9ea8-4935-b729-d089d3bd9cbb" />

<img width="959" height="588" alt="image" src="https://github.com/user-attachments/assets/cabb99bc-5312-4ff9-8c77-62d5fb121845" />

<img width="1018" height="640" alt="image" src="https://github.com/user-attachments/assets/f5c1c1a7-cda6-4663-b630-ed04847899bd" />

Configuramos Apache para Python (WSGI)

<img width="691" height="80" alt="image" src="https://github.com/user-attachments/assets/6296c9ce-06cc-4523-9dbb-f3f8a66696e3" />

Editamos la configuración del virtual host

<img width="947" height="558" alt="image" src="https://github.com/user-attachments/assets/394f6f8f-27e8-40eb-9690-7eea9a5e1d49" />

Damos permisos a python

<img width="683" height="101" alt="image" src="https://github.com/user-attachments/assets/59cff864-7038-413a-a39e-42cb18125459" />

Al comprobarlo ha dado un error que no pudimos solucionar.

### 6. Proteger Python con Autenticación 

Al no completar el anterior no se pudo realizar este.

### 7. Instalar y Configurar AWStats

AWStats (Advanced Web Statistics) es una potente herramienta de análisis de logs de acceso que nos permitirá monitorizar el tráfico de nuestros sitios web. A diferencia de soluciones basadas en JavaScript como Google Analytics, AWStats analiza directamente los archivos de log del servidor, proporcionando estadísticas más precisas sobre el tráfico real del servidor, incluyendo robots de búsqueda y accesos que no ejecutan JavaScript.

Ventajas de AWStats:

- Análisis de logs en tiempo real

- Detección de bots y spiders

- Estadísticas de ancho de banda consumido

- Información geográfica de visitantes

- Bajo consumo de recursos del servidor

### 7.1  Instalación de AWStats

Comenzamos instalando el paquete AWStats desde los repositorios oficiales de Ubuntu:

sudo apt install awstats -y

<img width="1001" height="640" alt="image" src="https://github.com/user-attachments/assets/bf069246-fa03-48f0-98a6-fcc6b9780dfb" />

Durante la instalación, se instalan automáticamente varias dependencias necesarias, incluyendo herramientas de análisis de logs y módulos de Perl. El sistema mantiene una gestión limpia de paquetes, eliminando automáticamente aquellos que ya no son necesarios.

Creación del Archivo de Configuración Específico
Cada sitio web requiere su propia configuración de AWStats. Creamos una copia del archivo de configuración por defecto adaptada a nuestro dominio:

sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.centro.intranet.conf

<img width="1014" height="47" alt="image" src="https://github.com/user-attachments/assets/1aec1c7a-8dc3-44c7-9990-bd408d60c1b3" />

Esta acción crea un archivo de configuración específico para centro.intranet basado en la plantilla por defecto.

<img width="937" height="46" alt="image" src="https://github.com/user-attachments/assets/816d90fb-c2c0-47fc-9fcb-25e6fe494484" />

Realizamos las siguientes modificaciones:

Línea 50: Especificamos el archivo de log de Apache

LogFile="/var/log/apache2/centro_access.log"

Línea 192: Definimos el dominio del sitio

SiteDomain="centro.intranet"

Línea 209: Nombre descriptivo para el informe

HostAliases="centro.intranet www.centro.intranet localhost 127.0.0.1"

<img width="1010" height="636" alt="image" src="https://github.com/user-attachments/assets/3a6209e1-f066-4720-9105-3c0f6c1d8eae" />

<img width="1001" height="636" alt="image" src="https://github.com/user-attachments/assets/c0cc7da7-9af1-4160-8e90-2104e6036517" />

<img width="968" height="632" alt="image" src="https://github.com/user-attachments/assets/8c6a7ab9-1a89-4513-ad7f-e1de9f3637c6" />

Explicación de los parámetros configurados:

- LogFile: Ruta exacta del archivo de log de acceso de Apache para este dominio

- SiteDomain: Nombre canónico del sitio para generar estadísticas correctamente

- HostAliases: Lista de alias que deben ser considerados como parte del mismo sitio

- Consideraciones importantes:

- Asegurarse de que la ruta del log coincide exactamente con la configuración de Apache

- Verificar permisos de lectura sobre los archivos de log

- Configurar la zona horaria correcta si el servidor está en una ubicación diferente

### 7.2  Configuración del Módulo CGI en Apache

AWStats utiliza CGI (Common Gateway Interface) para generar informes dinámicos a través del navegador web. Activamos los módulos necesarios:

Activación del Módulo CGI

sudo a2enmod cgi

El módulo CGI permite ejecutar scripts Perl desde el servidor web, necesario para AWStats.

Configuración del Directorio CGI

sudo a2enconf serve-cgi-bin

Esta configuración habilita el directorio estándar /usr/lib/cgi-bin/ para la ejecución de scripts CGI.

Aplicación de los Cambios

sudo systemctl restart apache2

Reiniciamos Apache para que los cambios en la configuración de CGI surtan efecto.

<img width="874" height="162" alt="image" src="https://github.com/user-attachments/assets/b722a177-e480-48e7-8469-4362f20ffdb3" />

Generación de Estadísticas Iniciales

Ejecución del Proceso de Análisis

Ejecutamos AWStats por primera vez para procesar los logs existentes:

sudo /usr/lib/cgi-bin/awstats.pl -config=centro.intranet -update































































