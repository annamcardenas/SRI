# GUÍA COMPLETA PARA CONFIGURAR UN SERVIDOR DNS CON BIND9

**Dominio:** `marisma.intranet` | **Servidor:** `192.168.191.76`

---

## ÍNDICE

1. [Introducción](#1-introducción)
2. [Paso 1: Instalación de BIND9](#4-paso-1-instalación-de-bind9)
3. [Paso 2: Configuración del hostname](#5-paso-2-configuración-del-hostname)
4. [Paso 3: Configuración de opciones globales](#6-paso-3-configuración-de-opciones-globales)
5. [Paso 4: Definición de las zonas](#7-paso-4-definición-de-las-zonas)
6. [Paso 5: Creación del archivo de zona directa](#8-paso-5-creación-del-archivo-de-zona-directa)
7. [Paso 6: Creación del archivo de zona inversa](#9-paso-6-creación-del-archivo-de-zona-inversa)
8. [Paso 7: Verificación de la configuración](#10-paso-7-verificación-de-la-configuración)
9. [Paso 8: Reinicio del servicio BIND9](#11-paso-8-reinicio-del-servicio-bind9)
10. [Paso 9: Configuración del cliente DNS](#12-paso-9-configuración-del-cliente-dns)
11. [Paso 10: Comprobaciones finales](#13-paso-10-comprobaciones-finales)

---

## 1. INTRODUCCIÓN

### ¿Qué es un servidor DNS autoritativo?

Un servidor DNS autoritativo (authoritative-only) es aquel que contiene la información "oficial" y definitiva sobre uno o varios dominios. A diferencia de un servidor recursivo (que consulta a otros servidores para resolver nombres), el servidor autoritativo solo responde preguntas sobre los dominios que él mismo gestiona.

### Ventajas de este enfoque:

- **Mayor seguridad** (no se usa para recursión abierta, evitando ataques de amplificación DNS)
- **Mejor rendimiento** (menos carga de trabajo al no hacer caché de dominios externos)
- **Control total** sobre los registros del dominio

### ¿Qué vamos a configurar?

- **Dominio:** `marisma.intranet`
- **Red:** `192.168.191.0/24`
- **Servidor DNS Master:** IP `192.168.191.76` (tu máquina actual)

# Configuración de servidor DNS con BIND9

---

## 2. PASO 1: INSTALACIÓN DE BIND9

Ahora que el sistema está preparado, instalamos el servidor DNS.

```bash
# Actualizar la lista de paquetes
sudo apt update

# Instalar BIND9 (el servidor DNS)
sudo apt install bind9 -y

# Verificar que el servicio se instaló correctamente
systemctl status bind9
```

<img width="771" height="374" alt="image" src="https://github.com/user-attachments/assets/d2e4d46f-76b5-452c-8b90-495bc89d446d" />

En este momento BIND9 intentará arrancar pero fallará porque el puerto 53 está libre (ya deshabilitamos systemd-resolved). Más adelante lo configuraremos correctamente.

**Salida esperada:**

```
named.service - BIND Domain Name Server
Loaded: loaded (/lib/systemd/system/named.service; enabled)
Active: active (running)
```

<img width="745" height="407" alt="image" src="https://github.com/user-attachments/assets/018a38eb-4e61-4a35-86bc-9a6598fc688f" />

---

## 3. PASO 2: CONFIGURACIÓN DEL HOSTNAME

Es importante que el servidor tenga un nombre de host correctamente configurado.

```bash
# Establecer el nombre del equipo
sudo hostnamectl set-hostname ns1.marisma.intranet

# Editar el archivo /etc/hosts para resolución local
sudo nano /etc/hosts
```

**Contenido de `/etc/hosts`:**

```
127.0.0.1       localhost
192.168.191.76  ns1.marisma.intranet ns1

# Las siguientes líneas son para IPv6 (opcional)
::1     localhost ip6-localhost ip6-loopback
```

<img width="552" height="181" alt="image" src="https://github.com/user-attachments/assets/59b244ff-d9fc-4340-a331-aa7fa222b594" />

**Verificar configuración:**

```bash
hostname
hostname -f  # Debe mostrar ns1.marisma.intranet
```

<img width="256" height="88" alt="image" src="https://github.com/user-attachments/assets/6a1a2ea2-a545-4b27-a0be-cdc6eb55b0b5" />

---

## 4. PASO 3: CONFIGURACIÓN DE OPCIONES GLOBALES

Editamos el archivo principal de opciones de BIND.

```bash
sudo nano /etc/bind/named.conf.options
```

**Contenido completo del archivo:**

```
options {
    # Directorio donde BIND guarda archivos temporales y caché
    directory "/var/cache/bind";

    # DESHABILITAR RECURSIÓN - Clave para servidor autoritativo
    # Esto evita que nuestro servidor actúe como resolvedor recursivo
    # Mejora la seguridad y el rendimiento
    recursion no;

    # Denegar transferencias de zona por defecto
    # Las permitiremos solo para zonas específicas si es necesario
    allow-transfer { none; };

    # Escuchar SOLO en nuestra interfaz de red
    # No escuchamos en localhost para evitar conflictos
    listen-on { 192.168.191.76; };
    listen-on-v6 { none; };

    # Quiénes pueden consultar nuestro servidor
    allow-query { any; };

    # Deshabilitar caché de consultas (no necesitamos caché al ser autoritativo)
    allow-query-cache { none; };

    # Ocultar versión de BIND por seguridad
    version "Servidor DNS Autorizado - marisma.intranet";

    # Deshabilitar DNSSEC para simplificar la práctica
    dnssec-validation no;

    # No aceptar consultas recursivas de nadie
    allow-recursion { none; };
};

# Configuración de logging personalizada para facilitar depuración
logging {
    channel custom_log {
        file "/var/log/named/custom.log" versions 3 size 5m;
        severity info;
        print-time yes;
        print-severity yes;
        print-category yes;
    };
    category default { custom_log; };
    category queries { custom_log; };
};
```

<img width="595" height="414" alt="image" src="https://github.com/user-attachments/assets/29c2095f-1321-4ff8-b44d-c8067340b83c" />

**Crear directorio de logs:**

```bash
sudo mkdir -p /var/log/named
sudo chown bind:bind /var/log/named
```

<img width="498" height="74" alt="image" src="https://github.com/user-attachments/assets/42f0d3f4-926e-42d4-9865-b9fbdd34307e" />

---

## 5. PASO 4: DEFINICIÓN DE LAS ZONAS

Editamos el archivo donde declaramos las zonas que gestionará nuestro servidor.

```bash
sudo nano /etc/bind/named.conf.local
```

**Contenido completo:**

```
// ============================================================
// ZONA DIRECTA: marisma.intranet
// ============================================================
zone "marisma.intranet" {
    type master;                                          // Somos el servidor primario
    file "/var/lib/bind/master/directa.marisma.db";      // Archivo de zona
    notify yes;                                           // Notificar cambios a secundarios (si los hubiera)
};

// ============================================================
// ZONA INVERSA: Red 192.168.191.0/24
// ============================================================
zone "191.168.192.in-addr.arpa" {
    type master;                                          // Somos el servidor primario
    file "/var/lib/bind/master/inversa.191.db";           // Archivo de zona
};
```

<img width="747" height="298" alt="image" src="https://github.com/user-attachments/assets/df7a863b-386a-46e1-8fd8-6ff1caa227f9" />


**Explicación del nombre de la zona inversa:**

- Tu red es `192.168.191.0/24`
- Para la zona inversa, se invierten los octetos: `191.168.192.in-addr.arpa`
- El 192 se queda igual, el 168 igual, y el 191 es el que cambia (es el tercer octeto de tu IP)

**Crear el directorio para los archivos de zona:**

```bash
# Crear directorio master dentro de /var/lib/bind (ubicación estándar de Debian/Ubuntu)
sudo mkdir -p /var/lib/bind/master

# Asegurar permisos correctos
sudo chown -R bind:bind /var/lib/bind/master
```

<img width="522" height="67" alt="image" src="https://github.com/user-attachments/assets/5c73110d-64dd-424a-915b-a0b86bcb8c3b" />

---

## 6. PASO 5: CREACIÓN DEL ARCHIVO DE ZONA DIRECTA

Este archivo contiene los registros que traducen nombres de dominio a direcciones IP.

```bash
sudo nano /var/lib/bind/master/directa.marisma.db
```

**Contenido completo del archivo:**

```
; ============================================================
; ZONA DNS DIRECTA PARA marisma.intranet
; Servidor: ns1.marisma.intranet (192.168.191.76)
; Fecha creación: Abril 2026
; ============================================================
$ORIGIN marisma.intranet.
$TTL 86400
@   IN  SOA ns1.marisma.intranet. hostmaster.marisma.intranet. (
            2026040201  ; Serial (YYYYMMDDNN) - ¡INCREMENTAR AL MODIFICAR!
            10800       ; Refresh (3 horas) - Cada cuánto revisan los secundarios
            3600        ; Retry (1 hora) - Tiempo entre reintentos fallidos
            604800      ; Expire (7 días) - Tiempo hasta expirar zona
            86400 )     ; Minimum TTL (1 día) - Tiempo de caché negativo

; ============================================================
; REGISTROS NS (Name Servers)
; ============================================================
@   IN  NS  ns1.marisma.intranet.
@   IN  NS  ns2.marisma.intranet.  ; Secundario (previsión de futuro)

; ============================================================
; REGISTROS A (Name -> IP Address)
; ============================================================
; Servidores DNS
ns1             IN  A   192.168.191.76
ns2             IN  A   192.168.191.100  ; Secundario (previsión)

; Servidor FTP
ftp1            IN  A   192.168.191.10

; Servidores de correo
mail1           IN  A   192.168.191.20
mail2           IN  A   192.168.191.21

; Servidores web
www             IN  A   192.168.191.30
departamentos   IN  A   192.168.191.31

; ============================================================
; REGISTROS MX (Mail Exchange) - Prioridad menor = más prioritaria
; ============================================================
@   IN  MX  10  mail1.marisma.intranet.
@   IN  MX  20  mail2.marisma.intranet.

; ============================================================
; REGISTROS CNAME (Alias) - Valor añadido
; ============================================================
ftp     IN  CNAME   ftp1.marisma.intranet.
webmail IN  CNAME   www.marisma.intranet.
```

**Explicación de cada componente:**

| Directiva/Registro | Significado |
|--------------------|-------------|
| `$TTL 86400` | Tiempo de vida de información en caché (1 día = 86400 segs) |
| `$ORIGIN` | Define el dominio base para todos los nombres no cualificados |
| `SOA` (Start of Authority) | Registro obligatorio que identifica al servidor principal |
| `hostmaster` | Email del administrador (el `@` se sustituye por punto) |
| `Serial` | Número de versión de la zona. Incrementar cada modificación |
| `NS` (Name Server) | Indica qué servidores son autoritativos para la zona |
| `A` (Address) | Traduce un nombre de host a dirección IPv4 |
| `MX` (Mail Exchange) | Indica servidores de correo y sus prioridades |
| `CNAME` (Canonical Name) | Crea un alias para otro nombre |


<img width="749" height="436" alt="image" src="https://github.com/user-attachments/assets/7c0ffad3-b416-44e0-8c68-56a76027b83f" />

---

## 7. PASO 6: CREACIÓN DEL ARCHIVO DE ZONA INVERSA

La zona inversa permite traducir direcciones IP a nombres de dominio (resolución inversa).

```bash
sudo nano /var/lib/bind/master/inversa.191.db
```

**Contenido completo del archivo:**

```
; ============================================================
; ZONA DNS INVERSA PARA RED 192.168.191.0/24
; Resolución IP -> Nombre
; ============================================================
$ORIGIN 191.168.192.in-addr.arpa.
$TTL 86400
@   IN  SOA ns1.marisma.intranet. hostmaster.marisma.intranet. (
            2026040201  ; Serial (debe coincidir con zona directa)
            10800       ; Refresh
            3600        ; Retry
            604800      ; Expire
            86400 )     ; Minimum TTL

; ============================================================
; REGISTROS NS (Name Servers)
; ============================================================
@   IN  NS  ns1.marisma.intranet.
@   IN  NS  ns2.marisma.intranet.

; ============================================================
; REGISTROS PTR (Pointer) - IP -> Nombre
; ============================================================
; Servidores DNS
76  IN  PTR ns1.marisma.intranet.
100 IN  PTR ns2.marisma.intranet.

; Servidor FTP
10  IN  PTR ftp1.marisma.intranet.

; Servidores de correo
20  IN  PTR mail1.marisma.intranet.
21  IN  PTR mail2.marisma.intranet.

; Servidores web
30  IN  PTR www.marisma.intranet.
31  IN  PTR departamentos.marisma.intranet.
```

<img width="729" height="414" alt="image" src="https://github.com/user-attachments/assets/fe6045b1-6c07-4c9c-a4f6-b9b340f41748" />


**Explicación de los registros PTR:**

- `PTR` (Pointer): El registro que asocia una IP con un nombre de dominio
- Formato: Solo se escribe el último octeto de la IP (el resto lo da `$ORIGIN`)
  - `76` → Se convierte en `76.191.168.192.in-addr.arpa`
  - `10` → Se convierte en `10.191.168.192.in-addr.arpa`
- Correspondencia: Cada IP debe tener UN solo registro PTR

---

## 8. PASO 7: VERIFICACIÓN DE LA CONFIGURACIÓN

Antes de reiniciar el servicio, debemos comprobar que no hay errores de sintaxis.

**Verificar archivo de configuración global:**

```bash
sudo named-checkconf
```

Si no muestra nada, significa que la sintaxis es correcta.

**Verificar zona directa:**

```bash
sudo named-checkzone marisma.intranet /var/lib/bind/master/directa.marisma.db
```

Salida esperada:

```
zone marisma.intranet/IN: loaded serial 2026040201
OK
```

<img width="781" height="94" alt="image" src="https://github.com/user-attachments/assets/22cfd621-1e5b-4876-b0da-163d754d6b7f" />


**Verificar zona inversa:**

```bash
sudo named-checkzone 191.168.192.in-addr.arpa /var/lib/bind/master/inversa.191.db
```

Salida esperada:

```
zone 191.168.192.in-addr.arpa/IN: loaded serial 2026040201
OK
```

<img width="783" height="69" alt="image" src="https://github.com/user-attachments/assets/25dbcd1a-9a5f-4aad-929b-fc971148ee03" />


---

## 9. PASO 8: REINICIO DEL SERVICIO BIND9

Una vez verificada toda la configuración, reiniciamos BIND9.

```bash
# Reiniciar el servicio
sudo systemctl restart bind9

# Verificar que el servicio está activo
sudo systemctl status bind9
```

**Salida esperada:**

```
named.service - BIND Domain Name Server
Loaded: loaded (/lib/systemd/system/named.service; enabled)
Active: active (running) since …
```

<img width="749" height="438" alt="image" src="https://github.com/user-attachments/assets/dfbc7e2b-a873-4890-8595-c2f21f765b9d" />


**Verificar logs por si hay errores:**

```bash
# Ver logs específicos de BIND
sudo tail -f /var/log/named/custom.log

# Alternativa: ver logs del sistema
sudo tail -f /var/log/syslog | grep named
```

<img width="819" height="294" alt="image" src="https://github.com/user-attachments/assets/f9ccb2fa-50d1-47fe-9eed-52cbc30d5bc8" />


Presiona `Ctrl+C` para salir de la visualización de logs.

**Verificar que BIND9 está escuchando en el puerto 53:**

```bash
sudo ss -tulpn | grep :53
```

Salida esperada:

```
udp  LISTEN 0  10  192.168.191.76:53  0.0.0.0:*  users:(("named",pid=XXXX,fd=XX))
tcp  LISTEN 0  10  192.168.191.76:53  0.0.0.0:*  users:(("named",pid=XXXX,fd=XX))
```

<img width="939" height="316" alt="image" src="https://github.com/user-attachments/assets/8772a3ee-2543-4c0e-a38a-21b79a05bdce" />


---

## 10. PASO 9: CONFIGURACIÓN DEL CLIENTE DNS

Ahora configuramos el sistema para que use nuestro servidor DNS (el mismo BIND9 que acabamos de instalar).

### 9.1 Cambiar el archivo `/etc/resolv.conf`

```bash
sudo nano /etc/resolv.conf
```

**Contenido:**

```
nameserver 192.168.191.76
search marisma.intranet
```

<img width="526" height="81" alt="image" src="https://github.com/user-attachments/assets/4155837d-9e4c-42c8-9f65-5a800fba9b3b" />


### 9.2 Configurar `/etc/nsswitch.conf`

Para que `ping` y el navegador funcionen correctamente:

```bash
sudo nano /etc/nsswitch.conf
```

Buscar la línea que empieza por `hosts:` y modificarla:

**Antes:**

```
hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4
```

**Después:**

```
hosts: files dns
```

> Esto elimina el uso de `mdns4` (multicast DNS) que puede interferir con la resolución de nombres locales. Ahora el sistema consultará primero `/etc/hosts` (`files`) y luego el servidor DNS (`dns`).

<img width="726" height="425" alt="image" src="https://github.com/user-attachments/assets/ff08decf-91cd-427b-91fd-c9f9fb9affc8" />


### 9.3 Verificar la configuración del cliente

```bash
# Ver qué servidor DNS está usando el sistema
cat /etc/resolv.conf

# Probar resolución local (debe usar nuestro servidor)
dig www.marisma.intranet
```

<img width="301" height="73" alt="image" src="https://github.com/user-attachments/assets/5acef623-64b2-4073-8c78-3bbe0211ca77" />


<img width="626" height="409" alt="image" src="https://github.com/user-attachments/assets/3aa3346b-a5a8-4931-98c7-a8757762cb9d" />


---

## 11. PASO 10: COMPROBACIONES FINALES

### Comprobación 1: Consulta SOA (Start of Authority)

```bash
dig SOA marisma.intranet
```

Buscar en la respuesta la sección `ANSWER SECTION` con el registro SOA:

```
;; ANSWER SECTION:
marisma.intranet. 86400 IN SOA ns1.marisma.intranet. hostmaster.marisma.intranet. 2026040201 10800 3600 604800 86400
```

<img width="894" height="424" alt="image" src="https://github.com/user-attachments/assets/7c4623f2-ffc1-4f99-b781-280eeff6db97" />


### Comprobación 2: Consulta NS (Name Servers)

```bash
dig NS marisma.intranet
```

Resultado esperado: `ns1.marisma.intranet` y `ns2.marisma.intranet`

```
;; ANSWER SECTION:
marisma.intranet. 86400 IN NS ns1.marisma.intranet.
marisma.intranet. 86400 IN NS ns2.marisma.intranet.
```

<img width="632" height="495" alt="image" src="https://github.com/user-attachments/assets/87acb511-8cb9-4d16-aa38-34411e97ec8d" />


### Comprobación 3: Consulta MX (Mail Exchange)

```bash
dig MX marisma.intranet
```

Resultado esperado:

```
;; ANSWER SECTION:
marisma.intranet. 86400 IN MX 10 mail1.marisma.intranet.
marisma.intranet. 86400 IN MX 20 mail2.marisma.intranet.
```

<img width="680" height="492" alt="image" src="https://github.com/user-attachments/assets/e1adf972-1186-4013-bf71-d239477a38ec" />


### Comprobación 4: Consultas A (direcciones IP)

```bash
# Consultar servidor DNS
dig A ns1.marisma.intranet
```

<img width="653" height="415" alt="image" src="https://github.com/user-attachments/assets/f913da2d-94da-488a-9e81-e95676d8c9e4" />

```
# Consultar servidor web
dig A www.marisma.intranet
```

<img width="653" height="421" alt="image" src="https://github.com/user-attachments/assets/e9c52059-6e56-43c6-bdd1-5ff08827b056" />

```
# Consultar servidor FTP
dig A ftp1.marisma.intranet
```

<img width="654" height="413" alt="image" src="https://github.com/user-attachments/assets/2b48806a-56b5-40b2-968e-f8a329f82415" />

```
# Consultar servidor de correo
dig A mail1.marisma.intranet
```

<img width="676" height="408" alt="image" src="https://github.com/user-attachments/assets/a16f7aba-dbbf-4a76-8755-04127a9bf481" />

```
# Consultar departamentos
dig A departamentos.marisma.intranet
```

<img width="725" height="414" alt="image" src="https://github.com/user-attachments/assets/700b4065-789a-40f1-be65-91e255fdca87" />


Resultado esperado para `www`:

```
;; ANSWER SECTION:
www.marisma.intranet. 86400 IN A 192.168.191.30
```

### Comprobación 5: Resolución inversa (PTR)

```bash
# Consultar IP del servidor DNS
dig -x 192.168.191.76
```

<img width="623" height="405" alt="image" src="https://github.com/user-attachments/assets/c449db84-ed7d-46ea-b3e7-5f09b7cb849a" />

```
# Consultar IP del servidor web
dig -x 192.168.191.30
```

<img width="648" height="408" alt="image" src="https://github.com/user-attachments/assets/03c864b7-f195-4a81-9b1e-e0cba3712f8f" />

```
# Consultar IP del servidor FTP
dig -x 192.168.191.10
```

<img width="645" height="412" alt="image" src="https://github.com/user-attachments/assets/14416fcc-9b52-4d15-87aa-a3be4feaa43f" />


Resultado esperado para `192.168.191.30`:

```
;; ANSWER SECTION:
30.191.168.192.in-addr.arpa. 86400 IN PTR www.marisma.intranet.
```

### Comprobación 6: Prueba con ping

```bash
ping -c 3 www.marisma.intranet
ping -c 3 ftp1.marisma.intranet
ping -c 3 ns1.marisma.intranet
```

Resultado esperado:

```
PING www.marisma.intranet (192.168.191.30) 56(84) bytes of data.
64 bytes from 192.168.191.30: icmp_seq=1 ttl=64 time=0.023 ms
```

> **Nota:** El ping puede fallar si no hay ningún servidor escuchando en esas IPs. Esto no es un problema del DNS. El enunciado solo pide comprobar que el DNS resuelve correctamente. Si `dig` muestra la IP correcta, el DNS funciona.

<img width="784" height="480" alt="image" src="https://github.com/user-attachments/assets/c02f3a95-8cb2-470e-be21-b743d7a472e4" />


### Comprobación 7: Consulta con nslookup (alternativa)

```bash
nslookup marisma.intranet
nslookup www.marisma.intranet
nslookup 192.168.191.30
```
<img width="464" height="162" alt="image" src="https://github.com/user-attachments/assets/becdea94-3037-4f4f-aca9-b45b39ba0bd6" />

<img width="476" height="176" alt="image" src="https://github.com/user-attachments/assets/8aeb8ea1-da99-43d5-8e24-d0d222329f53" />

<img width="555" height="85" alt="image" src="https://github.com/user-attachments/assets/fc4eea7b-3e85-4d0a-9072-9dfcd5d3e409" />

> **Nota:** El comando nslookup marisma.intranet. Devuelve *** Can't find marisma.intranet: No answer. Esto es normal porque nslookup por defecto busca registros de tipo A, y no hay un registro A directo para marisma.intranet (solo hay NS, MX y SOA). Probamos:

```bash
nslookup -type=SOA marisma.intranet
```

<img width="467" height="259" alt="image" src="https://github.com/user-attachments/assets/75deb9a3-4f16-43bf-a930-1c16637b2156" />

```bash
nslookup -type=NS marisma.intranet
```

<img width="529" height="156" alt="image" src="https://github.com/user-attachments/assets/ed8c26a5-e0f5-4c4e-a89e-c82e9563ce4f" />


