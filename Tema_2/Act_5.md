# Instalar Bind en el servidor DNS

## Primero actualizamos los paquetes 

<img width="709" height="161" alt="image" src="https://github.com/user-attachments/assets/bf6a8fb5-a0c0-4924-a18c-de2da8250729" />

Instalamos los componentes bind.

<img width="882" height="107" alt="image" src="https://github.com/user-attachments/assets/c3880fac-3f22-4826-8a56-85d1548f2f52" />

## Configurar como servidor DNS de almacenamiento en caché

Primero configuramos Bind para que actúe como un servidor DNS de almacenamiento en caché. Esta configuración obligará al servidor a buscar respuestas recursivamente de otros servidores DNS cuando un cliente emita una consulta. 

Esto significa que realiza el trabajo de consultar cada servidor DNS relacionado por turno hasta encontrar la respuesta completa.

Los archivos de configuración de Bind se guardan de forma predeterminada en un directorio en /etc/bind. Muévete a ese directorio ahora:

<img width="831" height="403" alt="image" src="https://github.com/user-attachments/assets/e175b638-fbcc-450f-9b92-9d72413c12d9" />

Se llama al archivo de configuración principal named.conf (named y bind son dos nombres para la misma aplicación). Este archivo simplemente obtiene el named.conf.options archivo, el named.conf.local archivo, y el named.conf.default-zones archivo.

Para un servidor DNS de almacenamiento en caché, solo modificaremos el named.conf.options archivo. 

Ponemos esto en /etc/bind/named.conf.options:

options {
        directory "/var/cache/bind";

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};

<img width="977" height="648" alt="image" src="https://github.com/user-attachments/assets/e2f7acf6-270b-4023-9ea4-b0bd91004e1f" />

Para configurar el almacenamiento en caché, el primer paso es configurar una lista de control de acceso o ACL.

Como servidor DNS que se utilizará para resolver consultas recursivas, no queremos que usuarios malintencionados abusen del servidor DNS. Un ataque llamado a Ataque de amplificación de DNS es especialmente problemático

Para evitar la posibilidad de que su servidor sea utilizado con fines maliciosos, configuraremos una lista de direcciones IP o rangos de red en los que confiamos.

Por encima del options bloque, crearemos un nuevo bloque llamado acl. Crea una etiqueta para el grupo ACL que estás configurando. En esta guía llamaremos al grupo buenos clientes.

<img width="787" height="185" alt="image" src="https://github.com/user-attachments/assets/f09d7c60-dc45-4625-9e5b-7933e990173e" />

Enumeramos las direcciones IP o redes a las que se les debe permitir utilizar este servidor DNS. Dado que tanto nuestro servidor como nuestro cliente operan dentro de la misma subred /24 en nuestro ejemplo, restringiremos el ejemplo a esta red. 

Debemos ajustar esto para incluir a tus propios clientes y no a terceros. También agregaremos localhost y localnets que intentará hacer esto automáticamente:

<img width="745" height="143" alt="image" src="https://github.com/user-attachments/assets/43f3d7d5-3347-4d22-ba91-ba1720a7f27a" />

Ahora podemos configurar las capacidades del acl en el options.

<img width="550" height="137" alt="image" src="https://github.com/user-attachments/assets/cf829a48-8cff-4d3b-b7f0-77b83e7c7fb8" />


Activamos explícitamente la recursión y luego configuramos el allow-query parámetro para utilizar nuestra especificación ACL.Si está presente y la recursión está activada, allow-recursion dictará la lista de clientes que pueden utilizar servicios recursivos.

Sin embargo, si allow-recursion no está configurado, entonces Bind vuelve a caer sobre el allow-query-cache lista, luego la allow-query lista, y finalmente un valor predeterminado de localnets y localhost sólo. Dado que estamos configurando un servidor solo de almacenamiento en caché (no tiene zonas autorizadas propias y no reenvía solicitudes), allow-query La lista siempre se aplicará solo a la recursión. Lo usamos porque es la forma más general de especificar la ACL.

Esto es todo lo que se requiere para un servidor DNS de almacenamiento en caché.
Configurar como servidor DNS de reenvío

Si un servidor DNS de reenvío se adapta mejor a su infraestructura, podemos configurarlo fácilmente.

Comenzaremos con la configuración que dejamos en la configuración del servidor de almacenamiento en caché. El named.conf.options El archivo debería verse así:

En /etc/bind/named.conf.options

acl goodclients {
    	192.0.2.0/24;
        localhost;
        localnets;
};

options {
        directory "/var/cache/bind";

        recursion yes;
        allow-query { goodclients; };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};

<img width="889" height="508" alt="image" src="https://github.com/user-attachments/assets/29453595-d3dd-4f59-89a3-e6251f43e898" />

Usaremos la misma lista ACL para restringir nuestro servidor DNS a una lista específica de clientes. Sin embargo, necesitamos cambiar la configuración para que el servidor ya no intente realizar consultas recursivas por sí mismo.

El servidor de reenvío sigue proporcionando servicios recursivos respondiendo consultas sobre zonas para las que no tiene autoridad. En su lugar, necesitamos configurar una lista de servidores de almacenamiento en caché a los que reenviar nuestras solicitudes

Primero, creamos un bloque dentro llamado forwarders que contiene las direcciones IP de los servidores de nombres recursivos a los que queremos reenviar solicitudes. En nuestra guía utilizaremos los servidores DNS públicos de Google (8.8.8.8 y 8.8.4.4):

<img width="766" height="390" alt="image" src="https://github.com/user-attachments/assets/99898c98-8c5c-44a5-adfd-40c485d8fbf4" />

Después debemos configurar el forward directiva a “solamente” ya que este servidor reenviará todas las solicitudes y no debe intentar resolverlas por sí solo.

El archivo quedará así:

<img width="829" height="636" alt="image" src="https://github.com/user-attachments/assets/057b6870-0036-4996-bd14-97b53177ac30" />

Un último cambio que deberíamos hacer es el dnssec parámetros. Con la configuración actual, dependiendo de la configuración de los servidores DNS reenviados, es posible que vea algunos errores.

Para evitar esto, cambie el dnssec-validation configurando en “sí” y habilitando explícitamente dnssec:

<img width="793" height="498" alt="image" src="https://github.com/user-attachments/assets/bc9bb445-abe1-4524-a23e-6b3e19e39d6f" />

Ahora debería tener instalado un servidor DNS de reenvío. 

Pruebe su configuración y reinicie Bind

Debemos utilizar las herramientas incluidas de Bind para verificar la sintaxis de nuestros archivos de configuración.

Podemos hacer esto fácilmente escribiendo:

sudo named-checkconf

<img width="802" height="82" alt="image" src="https://github.com/user-attachments/assets/56ebf46f-735d-4925-ae23-3d2a05450e81" />

Cuando haya verificado que sus archivos de configuración no tienen ningún error de sintaxis, reinicie el demonio Bind para implementar sus cambios:

<img width="805" height="120" alt="image" src="https://github.com/user-attachments/assets/0e179481-87ee-4848-8b4b-9a9d899b6d82" />

La primera vez que hacemos la consulta a google.com el tiempo de respuesta por parte de google es 319 msec.

<img width="1199" height="775" alt="image" src="https://github.com/user-attachments/assets/776881a5-8469-4c18-888d-9bb7b4ce9c44" />

La segunda consulta que se realiza tarda 1msec ya que esta consulta ya se ha guardado en caché.

<img width="1205" height="777" alt="image" src="https://github.com/user-attachments/assets/a276abe9-3a3d-4abb-81b1-7e75f43284a2" />

## Comprobación desde otro Ubuntu Desktop

En la primera consulta el query time fue de 23msec.

<img width="1027" height="667" alt="image" src="https://github.com/user-attachments/assets/babe5aeb-0d02-4ab5-a8d0-7f5d29d48997" />

Y en la segunda consulta vemos como el query time es de 2msec.

<img width="998" height="669" alt="image" src="https://github.com/user-attachments/assets/cc43e61c-d6d8-4360-8012-d0564e193a4f" />





