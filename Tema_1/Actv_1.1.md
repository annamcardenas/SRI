# Actividad #1

## La arquitectura Web es un modelo compuesto de tres capas, ¿cuáles son y cuál es  la función de cada una de ellas?

- Capa de presentación - Interfaz usuario frontend con **HTML CSS JavaScript** muestra información navegador y captura interacciones usuario responsable experiencia visual y usabilidad.
- Capa de aplicación - Lógica negocio backend con **PHP Python Java** procesa peticiones aplica reglas negocio coordina operaciones entre presentación y datos gestiona seguridad y autenticación.
- Capa de datos - Almacenamiento persistente con **MySQL MongoDB PostgreSQL** almacena recupera gestiona información base datos asegura integridad datos y acceso eficiente.

## Una plataforma web es el entorno de desarrollo de software empleado para  diseñar y ejecutar un sitio web; destacan dos plataformas web, LAMP y WISA. Explica en qué consiste cada una de ellas.

- **LAMP** - Stack open source Linux sistema operativo Apache servidor web MySQL base datos PHP lenguaje programación usado desarrollo web dinámico hosting compartido y aplicaciones web.

- **WISA** - Stack Microsoft Windows sistema operativo IIS servidor web SQL Server base datos ASP.NET framework desarrollo aplicaciones web usado entornos empresariales Windows Server.

## Lee el siguiente artículo e instala Apache en Ubuntu:

Se necesita crear una máquina virtual en Proxmox con Ubuntu, una vez instalado el sistema operativo se procede a descargar y instalar Apache.

Vamos al link de la descarga.

<img width="1443" height="718" alt="image" src="https://github.com/user-attachments/assets/53bcbdb4-cf3b-468e-b2eb-0c427411c90b" />


Descargamos los archivos en el link de download server.

<img width="1437" height="718" alt="image" src="https://github.com/user-attachments/assets/25479804-ee6a-404e-bee3-3d71c916afc0" />

También se puede instalar mediante comandos en la consola de comandos.

Primero actualizamos los paquetes del índice con este comando:

sudo apt update

<img width="1431" height="721" alt="image" src="https://github.com/user-attachments/assets/2f9bb32c-7e33-40f5-aad7-f229a12dc253" />

Después instalamos el paquete de Apache con el comando:

sudo apt install apache2

<img width="714" height="509" alt="image" src="https://github.com/user-attachments/assets/25915918-4469-4f84-a5b8-8e0d341d5f88" />

Ahora vamos a ajustar el Firewall, ya que es necesario modificar la configuración para permitir el acceso externo a los puertos web predeterminados. 

Usamos el comando sudo ufw app list.
Aparecerá una lista de las aplicaciones disponibles.

<img width="327" height="141" alt="image" src="https://github.com/user-attachments/assets/ea46e085-1cb9-4b49-b3a8-4ed33bb0071f" />

Aparecen tres perfiles disponibles para Apache:

- apache: Este perfil abre solo el puerto 80 (tráfico web normal y no cifrado)
- Apache completo: Este perfil abre tanto el puerto 80 (tráfico web normal y no cifrado) como el puerto 443 (tráfico cifrado TLS/SSL)
- Apache seguro: Este perfil abre solo el puerto 443 (tráfico cifrado TLS/SSL)


Como aún no hemos configurado SSL para nuestro servidor en esta guía, solo necesitaremos permitir el tráfico en el puerto 80:

sudo ufw allow 'Apache'

<img width="333" height="83" alt="image" src="https://github.com/user-attachments/assets/b4ebe1a1-4af2-42c3-b15f-c4d3c23a5486" />

Procedemos a instalar curl.

<img width="747" height="420" alt="image" src="https://github.com/user-attachments/assets/d1325d85-e0a4-431e-997d-9ab7d04e6645" />

Comprobamos que ha funcionado escribiendo curl http://localhost o escribiéndolo en el navegador y aparezca esta ventana.

<img width="1094" height="682" alt="image" src="https://github.com/user-attachments/assets/a2d58c34-48dc-40e5-893f-7ce0e7e17d75" />

Ahora vamos a descargar MySQL con el comando:

sudo apt install mysql-server

<img width="699" height="507" alt="image" src="https://github.com/user-attachments/assets/65d4640c-851c-421e-844a-f1fce70ceaf2" />

Configuramos la instalación con el comando:

sudo mysql_secure_installation

<img width="713" height="499" alt="image" src="https://github.com/user-attachments/assets/5daf42cc-cb56-4c9b-a529-8b64a423a6ad" />

Una vez instalado Apache y MySQL vamos a crear un host virtual para nuestro sitio web.

Creamos el directorio para nuestro dominio:

sudo mkdir var/www/midominio

<img width="716" height="499" alt="image" src="https://github.com/user-attachments/assets/1ed866cc-3e9d-41eb-a6a5-8a051e897377" />

Ahora asignamos la propiedad del directorio con la variable de entorno $USER, que hará referencia a su usuario de sistema actual:

<img width="728" height="127" alt="image" src="https://github.com/user-attachments/assets/f72cfc45-b01c-45f9-9544-4509abf700f9" />

Creamos un fichero.

<img width="778" height="58" alt="image" src="https://github.com/user-attachments/assets/c9358886-983c-4168-9474-bb3b210ac48c" />

Y escribimos este texto poniendo nuestro nombre de dominio.

<img width="694" height="495" alt="image" src="https://github.com/user-attachments/assets/d3829ef4-d534-4ed4-a367-ceb649748411" />

Ahora vamos a habilitar el nuevo host virtual con el comando:

sudo a2ensite midominio

<img width="406" height="127" alt="image" src="https://github.com/user-attachments/assets/679969fd-f2a1-460b-990d-17736bcfd2bf" />

Para deshabilitar el sitio web predeterminado de Apache escribimos:

sudo a2dissite 000-default

<img width="558" height="160" alt="image" src="https://github.com/user-attachments/assets/4bbb224b-0256-46ed-9482-a69178e0bb58" />

Para asegurarnos de que el archivo de configuración no contenga errores de sintaxis escribimos:

sudo apache2ctl configtest

<img width="840" height="154" alt="image" src="https://github.com/user-attachments/assets/e0c51693-7bf4-44bd-89f3-3a7f08dd6629" />

Volvemos a cargar Apache para que estos cambios surtan efecto:

<img width="439" height="60" alt="image" src="https://github.com/user-attachments/assets/4006e57d-32d4-4283-886b-620d5726a55c" />

Ahora, su nuevo sitio web está activo, pero el directorio root web todavía está vacío. Cree un archivo en esa ubicación para poder probar que el host virtual funcione según lo previsto:/var/www/your_domainindex.html

<img width="403" height="46" alt="image" src="https://github.com/user-attachments/assets/1a5e7234-e230-4a0e-b604-07a19a16f65f" />


<img width="838" height="575" alt="image" src="https://github.com/user-attachments/assets/4e4eb018-635e-41d8-8d5a-7567b1a4fc2c" />

En el navegador ponemos localhost y aparecerá nuestra página web creada.

<img width="759" height="320" alt="image" src="https://github.com/user-attachments/assets/44fc159a-20da-44dd-aca7-fb5fc2ed13ec" />




