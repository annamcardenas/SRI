# Práctica Final SRI

## 1. Creación de la Máquina Virtual

Creamos una máquina virtual con Ubuntu Server.

Configuramos la red para poner una IP estática, de la siguiente manera:

- **Interfaz:** ens18 (eth)
- **Método IPv4:** Manual
- **Subnet:** 192.168.193.0/24
- **Address:** 192.168.193.43
- **Gateway:** 192.168.193.101
- **Name servers:** 127.0.0.1, 8.8.8.8

<img width="916" height="240" alt="image" src="https://github.com/user-attachments/assets/1dd947ca-6423-4f37-8da2-b78d8a906be1" />

<img width="931" height="522" alt="image" src="https://github.com/user-attachments/assets/c92674b3-45a9-49f5-82f6-6b7fae32f85b" />

---

## 2. Configuración del Perfil

Configuramos nuestro perfil con los siguientes datos:

- **Your name:** admin
- **Server name:** server-sri
- **Username:** ana
- **Password:** (establecida)

<img width="958" height="293" alt="image" src="https://github.com/user-attachments/assets/c42a2e6e-29d8-4d74-ba99-c8705c14f811" />

### Instalación de SSH Key

Se instaló el servidor OpenSSH habilitando la opción **Install OpenSSH server**. Se permitió la autenticación por contraseña sobre SSH.

<img width="819" height="269" alt="image" src="https://github.com/user-attachments/assets/483284b0-a4ca-4c55-86ec-6075a35c88ae" />

<img width="1117" height="716" alt="image" src="https://github.com/user-attachments/assets/8e5bbf4b-f36e-473d-b3b2-28480262da6c" />

---

## 3. Actualización e Instalación de Paquetes

Actualizamos el sistema:

```bash
sudo apt update && sudo apt upgrade
```

<img width="633" height="176" alt="image" src="https://github.com/user-attachments/assets/a07de9b6-490b-4d2d-94a0-15a5f38d9c09" />


Instalamos todos los paquetes necesarios:

```bash
sudo apt install apache2 php libapache2-mod-php php-mysql mariadb-server phpmyadmin vsftpd bind9 bind9utils dnsutils python3 libapache2-mod-wsgi-py3 -y
```

<img width="1624" height="114" alt="image" src="https://github.com/user-attachments/assets/4f49e058-0828-4007-85a8-6f185c6e4e02" />

### Configuración de phpMyAdmin

- Se eligió **apache2** como servidor web.
- Se configuró la base de datos de phpMyAdmin con `dbconfig-common`.
- Se asignó y confirmó una contraseña de aplicación MySQL para phpMyAdmin.

<img width="851" height="324" alt="image" src="https://github.com/user-attachments/assets/ef20dd26-a3a5-42a0-9e8a-01fd0b4fe4b1" />

<img width="1105" height="270" alt="image" src="https://github.com/user-attachments/assets/bdd94bb5-7052-4596-b7c2-b173779b5101" />

<img width="1106" height="245" alt="image" src="https://github.com/user-attachments/assets/6930bf39-6828-48e5-b379-f205eccc9bf9" />

<img width="508" height="312" alt="image" src="https://github.com/user-attachments/assets/901b178d-aa44-4d61-b40a-bcd7da3bf114" />

<img width="905" height="571" alt="image" src="https://github.com/user-attachments/assets/12c7c054-d98d-4207-98e9-892ed117eb7b" />

---

## 4. Verificación de Servicios

### Apache

```bash
systemctl status apache2
```
Estado: **active (running)**

<img width="1131" height="320" alt="image" src="https://github.com/user-attachments/assets/8b93ebd8-c4d6-424a-9bec-69f75b3b7975" />

### MariaDB

```bash
systemctl status mariadb
```
Estado: **active (running)**

<img width="1134" height="366" alt="image" src="https://github.com/user-attachments/assets/b0f71ffb-207b-4ace-b87e-f94a25afcc08" />

### Servidor FTP (vsftpd)

```bash
systemctl status vsftpd
```
Estado: **active (running)**

<img width="673" height="208" alt="image" src="https://github.com/user-attachments/assets/0c273a0f-b383-4bd5-b94c-504c774769b0" />

### Servidor DNS (bind9)

```bash
systemctl status named
```
Estado: **active (running)**

<img width="1129" height="343" alt="image" src="https://github.com/user-attachments/assets/7d7714d5-58a0-47a2-9e1a-f321645e5dd5" />

---

## 5. Configuraciones

### 5.1 Asegurar el FTP con certificado TLS

Convertimos FTP (inseguro) en FTPS, que cifra contraseñas y archivos. Creamos un certificado autofirmado válido por un año:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/vsftpd.key \
  -out /etc/ssl/certs/vsftpd.crt \
  -subj "/C=ES/ST=Andalusia/L=Huelva/O=Hosting/CN=192.168.193.43"
```

<img width="1155" height="201" alt="image" src="https://github.com/user-attachments/assets/f084985c-07fe-4b3d-883d-7340025d3f0e" />

Configuramos vsftpd desde su archivo de configuración (`/etc/vsftpd.conf`):

<img width="786" height="385" alt="image" src="https://github.com/user-attachments/assets/c1ec942b-a89c-4dd1-a4ba-13094f7a6824" />

- Descomentamos `write_enable=YES` para permitir que los usuarios suban archivos.

<img width="416" height="61" alt="image" src="https://github.com/user-attachments/assets/17d997f9-5b9e-49bc-945f-8b01e1f4ce9a" />

- Reemplazamos la configuración del certificado RSA por defecto con el certificado TLS:

<img width="577" height="192" alt="image" src="https://github.com/user-attachments/assets/edded95b-2028-44b3-8b3a-8dc88c7cbc76" />


```
rsa_cert_file=/etc/ssl/certs/vsftpd.crt
rsa_private_key_file=/etc/ssl/private/vsftpd.key
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
ssl_ciphers=HIGH
```

Comprobación del TLS:

```bash
openssl s_client -connect 127.0.0.1:21 -starttls ftp
```

Resultado: certificado autofirmado válido, CN=192.168.193.43, válido desde Feb 14 2026 hasta Feb 14 2027.

<img width="657" height="400" alt="image" src="https://github.com/user-attachments/assets/5e642fe9-6abb-40fb-bc42-2ee871429010" />

---

### 5.2 Configuración de la Zona DNS

Editamos `/etc/bind/named.conf.local` y sustituimos el contenido por:

```
zone "dominiosri.local" {
    type master;
    file "/etc/bind/db.dominiosri.local";
};

zone "193.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.193";
};
```

<img width="685" height="184" alt="image" src="https://github.com/user-attachments/assets/f029e70b-7dba-4cf8-b53b-35f2be8cf3d8" />

<img width="557" height="300" alt="image" src="https://github.com/user-attachments/assets/0c8d0753-8ab7-447d-9276-507338ae8dba" />


#### Zona directa (`/etc/bind/db.dominiosri.local`)

```
$TTL    604800
@       IN      SOA     ns.dominiosri.local. root.dominiosri.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.dominiosri.local.
ns      IN      A       192.168.193.43
```

<img width="712" height="223" alt="image" src="https://github.com/user-attachments/assets/4e8374d8-9706-4786-b21d-e2f20a24d068" />

#### Zona inversa (`/etc/bind/db.193`)

```
$TTL    604800
@       IN      SOA     ns.dominiosri.local. root.dominiosri.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.dominiosri.local.
110     IN      PTR     ns.dominio.local.
```

<img width="716" height="191" alt="image" src="https://github.com/user-attachments/assets/a7f17d04-bac6-4841-840c-b96a544d063f" />

Verificación de zonas:

```bash
sudo named-checkzone dominiosri.local /etc/bind/db.dominiosri.local
# zone dominiosri.local/IN: loaded serial 2 → OK

sudo named-checkzone 193.168.192.in-addr.arpa /etc/bind/db.193
# zone 193.168.192.in-addr.arpa/IN: loaded serial 2 → OK
```

<img width="655" height="65" alt="image" src="https://github.com/user-attachments/assets/3b492d85-de28-4490-86f1-8556b19edff6" />

<img width="672" height="67" alt="image" src="https://github.com/user-attachments/assets/63c61b2f-efc2-4a8f-9658-24dc76d85ac9" />

Reinicio y comprobación:

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

<img width="902" height="364" alt="image" src="https://github.com/user-attachments/assets/9f69b5cb-03c3-4037-900c-644c0533fdac" />

Comprobación con dig:

```bash
dig @127.0.0.1 ns.dominiosri.local +short
# 192.168.193.43
```

<img width="607" height="68" alt="image" src="https://github.com/user-attachments/assets/6f02e15c-4352-4688-9e52-783a33d85ee4" />

---

### 5.3 Habilitación de Python (WSGI)

```bash
sudo a2enmod wsgi
# Module wsgi already enabled

sudo systemctl restart apache2
# Syntax OK
```

<img width="399" height="55" alt="image" src="https://github.com/user-attachments/assets/ae74da5d-f9db-48ec-8930-d8f1af3c6cd6" />

<img width="1182" height="90" alt="image" src="https://github.com/user-attachments/assets/d186bb09-e166-4ebf-a9b4-f0d4a8dcd7d6" />

---

## 6. Script de Automatización de Hosting

Se creó el script `crear_hosting.sh` con `sudo nano crear_hosting.sh`:

<img width="422" height="82" alt="image" src="https://github.com/user-attachments/assets/51a5d237-fc2f-46ce-ae14-3ad71f8690f5" />

```bash
#!/bin/bash
# Script simplificado para creación de hosting
################################################################################

# Verificar parámetros
if [ $# -ne 2 ]; then
    echo "Uso: sudo ./crear_hosting.sh USUARIO CONTRASEÑA"
    echo "Ejemplo: sudo ./crear_hosting.sh juan miapass123"
    exit 1
fi

CLIENTE=$1
CLAVE=$2
DOMINIO="${CLIENTE}.dominiosri.local"
IP="192.168.193.110"
ULTIMO_OCTETO="110"

echo ">>> Configurando hosting para: $CLIENTE"

# 1. Crear usuario y carpeta web
################################################################################
echo "[1/4] Creando usuario y carpeta web..."
useradd -m -s /bin/bash $CLIENTE
echo "$CLIENTE:$CLAVE" | chpasswd

CARPETA_WEB="/home/$CLIENTE/public_html"
mkdir -p $CARPETA_WEB

# Página de prueba
echo "" > $CARPETA_WEB/index.php

# Asignar permisos
chown -R $CLIENTE:www-data /home/$CLIENTE
chmod -R 755 /home/$CLIENTE

# 2. Crear base de datos
################################################################################
echo "[2/4] Creando base de datos..."
BD="${CLIENTE}_db"
mysql -u root -e "CREATE DATABASE $BD;"
mysql -u root -e "CREATE USER '$CLIENTE'@'localhost' IDENTIFIED BY '$CLAVE';"
mysql -u root -e "GRANT ALL ON $BD.* TO '$CLIENTE'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"

# 3. Configurar sitio web Apache
################################################################################
echo "[3/4] Configurando Apache..."
CONF="/etc/apache2/sites-available/$DOMINIO.conf"

cat > $CONF <<EOF
<VirtualHost *:80>
    ServerName $DOMINIO
    DocumentRoot $CARPETA_WEB

    <Directory $CARPETA_WEB>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/${CLIENTE}_error.log
    CustomLog \${APACHE_LOG_DIR}/${CLIENTE}_access.log combined
</VirtualHost>
EOF

a2ensite $DOMINIO.conf
systemctl reload apache2

# 4. Configurar DNS
################################################################################
echo "[4/4] Configurando DNS..."
echo "$CLIENTE IN A $IP" >> /etc/bind/db.dominiosri.local
echo "$ULTIMO_OCTETO IN PTR $DOMINIO." >> /etc/bind/db.193
systemctl restart bind9

# Resumen final
################################################################################
echo ""
echo "¡Todo listo!"
echo "====================================="
echo "Web:          http://$DOMINIO"
echo "FTP/SSH:      $CLIENTE / $CLAVE"
echo "Base de datos: $BD"
echo "====================================="
```

<img width="733" height="418" alt="image" src="https://github.com/user-attachments/assets/efcab8e8-daf6-47bd-9563-c8a8ae7d31f9" />


Se le concedieron permisos de ejecución:

```bash
chmod +x crear_hosting.sh
```

<img width="539" height="142" alt="image" src="https://github.com/user-attachments/assets/1b72060d-5b67-4679-ac50-b9f7b55b8109" />


Se creó un usuario de prueba:

```bash
sudo ./crear_hosting.sh Ana 1234
```

<img width="483" height="293" alt="image" src="https://github.com/user-attachments/assets/61386bb7-b0c1-4804-b053-74de242d5a1d" />

Salida:

```
>>> Configurando hosting para: Ana
[1/4] Creando usuario y carpeta web...
[2/4] Creando base de datos...
[3/4] Configurando Apache...
Enabling site Ana.dominiosri.local.
[4/4] Configurando DNS...
¡Todo listo!
=====================================
Web:          http://Ana.dominiosri.local
FTP/SSH:      Ana / 1234
Base de datos: Ana_db
=====================================
```

---

## 7. Comprobación con curl

Para comprobar el hosting se usa:

```bash
curl http://Ana.dominiosri.local
```

<img width="730" height="420" alt="image" src="https://github.com/user-attachments/assets/07e44461-f3d7-4a42-a047-b3091175279a" />


Si aparece "Could not resolve host", el sistema no usa el DNS local. Se soluciona configurando `/etc/resolv.conf`:

```
nameserver 127.0.0.1
nameserver 8.8.8.8
options edns0 trust-ad
search .
```

Otras comprobaciones:

```bash
# Ver solo las cabeceras HTTP
curl -I http://Ana.dominiosri.local

# Ver más detalles (incluyendo resolución DNS)
curl -v http://Ana.dominiosri.local
```

<img width="1104" height="622" alt="image" src="https://github.com/user-attachments/assets/af8abe9d-fc4b-42f0-86ff-da1f1ad7cbeb" />

<img width="1111" height="609" alt="image" src="https://github.com/user-attachments/assets/27445956-5c6e-49e9-bbb7-00188867824d" />


Resultado de `curl -I`:

```
HTTP/1.1 200 OK
Date: Wed, 01 Apr 2026 17:48:14 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Sat, 14 Feb 2026 12:49:30 GMT
Content-Type: text/html
```

<img width="524" height="176" alt="image" src="https://github.com/user-attachments/assets/8bdf3363-79b5-44fa-9399-39a20b8be800" />

---

## 8. Docker

### 8.1 Instalación

```bash
sudo apt install docker.io docker-compose-v2 -y
```

<img width="645" height="222" alt="image" src="https://github.com/user-attachments/assets/a6297b8e-8768-4014-820e-d3abff0558e1" />


Comprobación:

```bash
docker --version
# Docker version 28.2.2, build 28.2.2-0ubuntu1~24.04.1

docker compose version
# Docker Compose version 2.37.1+ds1-0ubuntu2~24.04.1
```

<img width="490" height="91" alt="image" src="https://github.com/user-attachments/assets/493dcf7e-71dd-45af-be5f-48951b6f76c3" />


### 8.2 Estructura de directorios

```bash
cd ~
mkdir -p practica_docker/html
mkdir -p practica_docker/dns
cd practica_docker
```

<img width="446" height="98" alt="image" src="https://github.com/user-attachments/assets/5a0b094e-d2ec-480c-b8f4-c65bccc5ea13" />

### 8.3 Archivo `docker-compose.yml`

```yaml
version: '3.8'

services:
  # 1. Contenedor DNS
  servidor_dns:
    image: ubuntu/bind9:latest
    container_name: dns_docker
    ports:
      - "5353:53/udp"
      - "5353:53/tcp"
    volumes:
      - ./dns:/etc/bind
    networks:
      - red_practica
    restart: unless-stopped

  # 2. Contenedor Web
  servidor_web:
    image: nginx:latest
    container_name: web_docker
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    networks:
      - red_practica
    restart: unless-stopped

# Configuración de la red virtual de Docker
networks:
  red_practica:
    driver: bridge
```

<img width="535" height="366" alt="image" src="https://github.com/user-attachments/assets/176d0f80-27ac-42bf-8b3d-8a3521d93784" />


### 8.4 Página web de prueba

```bash
echo "<h1>Práctica Docker - Servidor Web Alternativo</h1><p>Funcionando en puerto 8080</p>" > html/index.html
```

<img width="1137" height="441" alt="image" src="https://github.com/user-attachments/assets/1b11dd95-f47e-445c-8144-6227def51ebe" />

<img width="1130" height="311" alt="image" src="https://github.com/user-attachments/assets/5e058c2b-785e-4cb2-8286-cbd8936cdbc3" />


### 8.5 Configuración DNS en Bind9

`dns/named.conf`:

```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    listen-on { any; };
    listen-on-v6 { none; };
};

zone "docker.local" {
    type master;
    file "/etc/bind/db.docker.local";
};
```

`dns/db.docker.local`:

```
$TTL 604800
@       IN      SOA     ns.docker.local. admin.docker.local. (
                        2026040101      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@       IN      NS      ns.docker.local.
ns      IN      A       172.18.0.2
test    IN      A       172.18.0.2
```

### 8.6 Solución al error de permisos de Docker

Al ejecutar `docker compose up -d` por primera vez, se produjo un error de permisos. Solución:

```bash
# Añadir usuario al grupo docker
sudo usermod -aG docker $USER

# Verificar que se añadió
groups $USER

# Cerrar sesión completamente y volver a entrar
exit
```

### 8.7 Levantar los contenedores

```bash
cd ~/practica_docker
docker compose up -d
```

<img width="848" height="548" alt="image" src="https://github.com/user-attachments/assets/c4ea4dbd-fde3-4051-9758-86cb197ec7aa" />


---

## 9. Comprobaciones Docker

### 9.1 Estado de los contenedores

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

Resultado esperado:

```
NAMES        IMAGE                   STATUS          PORTS
web_docker   nginx:latest            Up X minutes    0.0.0.0:8080->80/tcp
dns_docker   ubuntu/bind9:latest     Up X minutes    0.0.0.0:5353->53/tcp
```

<img width="1136" height="101" alt="image" src="https://github.com/user-attachments/assets/737f89be-25d0-4c58-a9b8-04737938d9a3" />


### 9.2 Verificación del servidor web

```bash
curl http://localhost:8080
# <h1>Práctica Final - Docker Funcionando</h1>
```

<img width="772" height="57" alt="image" src="https://github.com/user-attachments/assets/4b1d918f-4d65-421e-8ed9-4df46799498d" />

### 9.3 Verificación del servidor DNS

```bash
dig @localhost -p 5353 test.docker.local
# test.docker.local. 604800 IN A 172.18.0.2
```

### 9.4 Comunicación entre contenedores

```bash
# Entrar al contenedor web
docker exec -it web_docker sh

# Dentro del contenedor, instalar dnsutils
apt update && apt install dnsutils -y

# Probar resolución DNS usando el contenedor dns_docker
dig @dns_docker test.docker.local +short
# 172.18.0.2

exit
```

<img width="662" height="80" alt="image" src="https://github.com/user-attachments/assets/03335467-c4b2-4f1e-9398-5493f17d82de" />

### 9.5 Verificación de volúmenes montados

```bash
# Ver archivos web desde el host
cat ~/practica_docker/html/index.html

# Modificar la página web desde el host
echo "<h1>Práctica Final - Docker Funcionando</h1>" > ~/practica_docker/html/index.html

# Verificar que el cambio se refleja en el contenedor
curl http://localhost:8080
```

<img width="784" height="75" alt="image" src="https://github.com/user-attachments/assets/08aa5fcc-41ee-4f3d-a533-21d2f946ab78" />

<img width="775" height="88" alt="image" src="https://github.com/user-attachments/assets/d194aa59-c5a7-4391-8d13-8dcaa6795e78" />

<img width="775" height="88" alt="image" src="https://github.com/user-attachments/assets/ce44f1a2-c77c-40a1-afb4-e0cf5a8607b7" />

<img width="678" height="255" alt="image" src="https://github.com/user-attachments/assets/785ff598-8ae2-4bbd-bec0-3a45049ff9bb" />

<img width="735" height="226" alt="image" src="https://github.com/user-attachments/assets/89a59b7b-b418-4115-9893-9c4b52213a45" />

---

## 10. Script de Despliegue Automatizado Docker

Se creó `desplegar_docker.sh`:

```bash
#!/bin/bash
# Script de despliegue automático de entorno Docker
# Práctica: Servidor Web y DNS en contenedores

echo "==========================================="
echo "  Iniciando Entorno Docker (Web + DNS)"
echo "==========================================="
echo ""

# Verificar que docker compose está disponible
if ! command -v docker &> /dev/null; then
    echo "Docker no está instalado"
    exit 1
fi

# Levantar los contenedores
echo "Levantando contenedores..."
docker compose up -d

# Esperar a que inicien
sleep 3

echo ""
echo "Estado de los contenedores:"
echo "-------------------------------------------"
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

echo ""
echo "Prueba de la web:"
curl -s http://localhost:8080 | head -1

echo ""
echo "Prueba del DNS:"
dig @localhost -p 5353 test.docker.local +short

echo ""
echo "==========================================="
echo "  Entorno desplegado correctamente"
echo "==========================================="
echo ""
echo "Para probar desde otro equipo:"
echo "  Web: http://192.168.193.110:8080"
echo "  DNS: dig @192.168.193.110 -p 5353 test.docker.local"
```

```bash
chmod +x desplegar_docker.sh
./desplegar_docker.sh
```

<img width="1283" height="539" alt="image" src="https://github.com/user-attachments/assets/90c7a192-a52a-4c3c-95c4-e85481ba8942" />

### Prueba DNS

<img width="780" height="448" alt="image" src="https://github.com/user-attachments/assets/57bfccef-3488-4d88-ad3b-ef67b26788cf" />

### Comunicación entre contenedores

<img width="891" height="40" alt="image" src="https://github.com/user-attachments/assets/d90e60cc-8619-4c2b-a4ed-992405d5f052" />


