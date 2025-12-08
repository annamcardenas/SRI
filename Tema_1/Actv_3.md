# Ejercicios

## 1. Crea un directorio llamado "dir1" y otro llamado "dir2"

<img width="866" height="650" alt="image" src="https://github.com/user-attachments/assets/96258b8b-2757-49db-8b93-6e92032e3b27" />

Vamos a var/www/html y creamos dos directorios con el comando sudo mkdir dir1.

## 2. Explica qué diferencia existe entre ambos y muestra su equivalencia con la directiva Require:

<Directory /var/www/example1>
Order Deny,Allow
Deny from All
Allow from 192.168.1.100
</Directory>


<Directory /var/www/example1>
Order Allow,Deny
Deny from All
Allow from 192.168.1.100
</Directory>

La diferencia es el orden en que el servidor web Apache evalúa las reglas de control de acceso: el primer ejemplo, Order Deny,Allow, denegará todo acceso de forma predeterminada y luego permitirá el acceso $192.168.1.100. El segundo ejemplo, Order Allow,Deny, permitirá todo acceso de forma predeterminada y luego denegará el acceso a todos excepto $192.168.1.100. 

Con order Deny, Allow: Solo 198.162.1.100 puede acceder al directorio.
Con order Allow, Deny: Todos excepto 192.162.1.100 puede acceder 

<img width="1104" height="688" alt="image" src="https://github.com/user-attachments/assets/4dd40d88-0b2d-43b6-b7d0-1ffe88119e49" />

En el archivo principal de apache etc/apache2/apache2.conf/ ponemos:

<Directory /etc/apache2/>
	Require ip 192.168.1.100
</Directory>

## 3. Para dir1

## a) Permite el acceso de las peticiones provenientes de 10.3.0.100

<img width="696" height="492" alt="image" src="https://github.com/user-attachments/assets/b0672fee-dfd2-4a25-a288-85ea0b8e9fc3" />

## b) Permite el acceso desde "marisma.intranet"

<img width="691" height="495" alt="image" src="https://github.com/user-attachments/assets/aa079875-d660-4010-acb6-c93d9782446e" />

## c) Permite el acceso desde cualquier subdominio de "marisma.intranet"

<img width="707" height="507" alt="image" src="https://github.com/user-attachments/assets/03bd6bae-008c-427a-8495-2566f95d8e41" />

## d) Permite el acceso de las peticiones provenientes de "10.3.0.100" con máscara "255.255.0.0"

<img width="691" height="494" alt="image" src="https://github.com/user-attachments/assets/c60abe8e-7adb-40ce-ae44-405026c1278a" />

## 4. Modifica la configuración de forma que el acceso a dir1:

## a) Se permita a "marisma.intranet" y no se permita desde 10.3.0.101

<img width="690" height="500" alt="image" src="https://github.com/user-attachments/assets/d4315367-7a1c-45f9-b387-c09c370b12c5" />

## 5. Modifica la configuración de forma que el acceso a dir2:

## a)Se permita a "10.3.0.100/8" y no a "marisma.intranet"

<img width="692" height="500" alt="image" src="https://github.com/user-attachments/assets/b478793b-c593-445f-8722-7cd6d1da1536" />







