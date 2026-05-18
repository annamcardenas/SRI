# Guía Completa para la Actividad 8: Subdominios en BIND9

**Dominio principal:** `marisma.intranet` | **Subdominio:** `informatica.marisma.intranet`

---

## Índice

1. [Introducción y objetivos](#1-introducción-y-objetivos)
2. [Requisitos previos](#2-requisitos-previos)
3. [Método 1: Subdominio virtual (obligatorio)](#3-método-1-subdominio-virtual-obligatorio)
4. [Método 2: Script en Bash para automatizar subdominios (opcional)](#4-método-2-script-en-bash-para-automatizar-subdominios-opcional)
5. [Método 3: Uso de $INCLUDE (opcional)](#5-método-3-uso-de-include-opcional)
6. [Verificaciones finales](#6-verificaciones-finales)

---

## 1. Introducción y objetivos

### ¿Qué es un subdominio?

Un subdominio es una división jerárquica de un dominio principal. Por ejemplo, `informatica.marisma.intranet` es un subdominio de `marisma.intranet`. Los subdominios permiten organizar mejor los recursos de una organización (departamentos, sedes, servicios).

### ¿Qué vamos a configurar?

| Elemento | Valor |
|----------|-------|
| Dominio principal | `marisma.intranet` |
| Subdominio | `informatica.marisma.intranet` |
| Hosts en el subdominio | `www`, `ftp`, `smtp` |
| IPs asignadas | `192.168.191.60`, `192.168.191.61`, `192.168.191.62` |

---

## 2. Requisitos previos

Antes de empezar, verificamos que el servidor DNS de la Actividad 6 sigue funcionando correctamente.

### 2.1 Verificar el estado de BIND9

```bash
sudo systemctl status bind9
```

Salida esperada: `active (running)`

<img width="763" height="430" alt="image" src="https://github.com/user-attachments/assets/107dfff0-6af9-4af5-b14d-fef65138e0e5" />


### 2.2 Verificar que el cliente DNS sigue apuntando a nuestro servidor

```bash
cat /etc/resolv.conf
```

Salida esperada:

```
nameserver 192.168.191.76
search marisma.intranet
```

<img width="310" height="76" alt="image" src="https://github.com/user-attachments/assets/846c0185-5574-4cf3-825d-7579f84a8bf9" />


### 2.3 Verificar que el dominio principal resuelve correctamente

```bash
dig www.marisma.intranet
```

Salida esperada: Sección `ANSWER SECTION` con `192.168.191.30`

<img width="614" height="410" alt="image" src="https://github.com/user-attachments/assets/c186bed2-9855-439a-aa7d-6f49c2f8d969" />


### 2.4 Hacer copia de seguridad de la configuración actual

```bash
# Copia de seguridad del archivo de zona directa
sudo cp /var/lib/bind/master/directa.marisma.db /var/lib/bind/master/directa.marisma.db.backup.act6

# Copia de seguridad de la configuración de zonas
sudo cp /etc/bind/named.conf.local /etc/bind/named.conf.local.backup.act6

# Verificar que las copias existen
ls -la /var/lib/bind/master/*.backup*
ls -la /etc/bind/*.backup*
```

> Ante cualquier error, podemos restaurar la configuración anterior fácilmente.

<img width="869" height="139" alt="image" src="https://github.com/user-attachments/assets/2ddc923a-50a6-4a0d-8a9f-21d267e53c14" />

---

## 3. Método 1: Subdominio virtual (obligatorio)

Este es el método más sencillo. Consiste en añadir los registros del subdominio directamente en el archivo de zona del dominio principal.

### 3.1 Editar el archivo de zona directa

```bash
sudo nano /var/lib/bind/master/directa.marisma.db
```

### 3.2 Añadir los registros del subdominio

Localiza la sección de registros A (después de los registros de departamentos) y añade:

```
; ============================================================
; SUBDOMINIO informatica.marisma.intranet
; ============================================================
; Hosts del departamento de informática
www.informatica   IN  A   192.168.191.60
ftp.informatica   IN  A   192.168.191.61
smtp.informatica  IN  A   192.168.191.62
```

<img width="602" height="351" alt="image" src="https://github.com/user-attachments/assets/aa228933-ec76-4386-9a62-335a69c593f0" />

**Explicación importante:**

- Al escribir `www.informatica`, BIND automáticamente añade el dominio base `marisma.intranet.` al final, formando el FQDN completo `www.informatica.marisma.intranet.`
- El punto al final de `marisma.intranet.` indica que es un nombre completamente calificado (FQDN)
- Cada host del subdominio tiene su propia IP, lo que permite separar servicios

### 3.3 Incrementar el número de serial

Localizamos la línea SOA que contiene el serial y lo cambiamos:

**Antes:**

```
2026040201  ; Serial (YYYYMMDDNN)
```

**Después:**

```
2026040202  ; Serial (YYYYMMDDNN) - Incrementado por la adición del subdominio
```

> Los servidores DNS secundarios (si los hubiera) y las cachés DNS utilizan el serial para saber si la zona ha cambiado. Si no incrementamos el serial, los cambios no se propagarán correctamente.

### 3.4 Verificar la sintaxis del archivo modificado

```bash
sudo named-checkzone marisma.intranet /var/lib/bind/master/directa.marisma.db
```

Salida esperada:

```
zone marisma.intranet/IN: loaded serial 2026040202
OK
```

<img width="781" height="78" alt="image" src="https://github.com/user-attachments/assets/bb617d28-b549-4327-be57-ba83d4c1442f" />


### 3.5 Reiniciar BIND9

```bash
sudo systemctl restart bind9
```

### 3.6 Verificar que el servicio arrancó correctamente

```bash
sudo systemctl status bind9
```

Salida esperada: `active (running)`

<img width="746" height="427" alt="image" src="https://github.com/user-attachments/assets/20f8df20-20be-4408-98fa-5e5ad9ed157e" />

### 3.7 Verificar los logs por si hay errores

```bash
sudo tail -f /var/log/named/custom.log
```

Presionamos `Ctrl+C` para salir.

---

<img width="849" height="393" alt="image" src="https://github.com/user-attachments/assets/6acb0c4f-7912-4d62-8ac5-db4cafe8fc6f" />


## 4. Método 2: Script en Bash para automatizar subdominios

### 4.1 Crear el script

```bash
sudo nano /usr/local/bin/crear_subdominio.sh
```

### 4.2 Contenido del script

```bash
#!/bin/bash

# ============================================================
# SCRIPT PARA CREAR SUBDOMINIOS EN BIND9
# Autor: Ana
# Dominio base: marisma.intranet
# ============================================================

# Colores para output más legible
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Archivo de zona principal
ZONE_FILE="/var/lib/bind/master/directa.marisma.db"

# Función para mostrar mensajes
mostrar_mensaje() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

mostrar_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

mostrar_advertencia() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

# Solicitar información al usuario
echo "=========================================="
echo "   CREADOR DE SUBDOMINIOS - BIND9"
echo "=========================================="
echo ""

read -p "Introduce el nombre del subdominio (ej: ventas, informatica, rrhh): " SUBDOMINIO

# Validar que el nombre no esté vacío
if [ -z "$SUBDOMINIO" ]; then
    mostrar_error "El nombre del subdominio no puede estar vacío"
    exit 1
fi

read -p "Introduce la IP base para este subdominio (ej: 192.168.191.70): " IP_BASE

# Validar formato de IP básico
if ! echo "$IP_BASE" | grep -qE '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$'; then
    mostrar_error "Formato de IP no válido"
    exit 1
fi

# Calcular IPs para los servicios
IP_WWW="$IP_BASE"
IP_FTP=$(echo "$IP_BASE" | awk -F. '{print $1"."$2"."$3"."($4+1)}')
IP_SMTP=$(echo "$IP_BASE" | awk -F. '{print $1"."$2"."$3"."($4+2)}')

mostrar_mensaje "Creando subdominio: $SUBDOMINIO.marisma.intranet"
mostrar_mensaje "IPs asignadas:"
echo "  - www.$SUBDOMINIO → $IP_WWW"
echo "  - ftp.$SUBDOMINIO → $IP_FTP"
echo "  - smtp.$SUBDOMINIO → $IP_SMTP"

# Preguntar si desea continuar
read -p "¿Continuar? (s/n): " CONFIRMAR
if [ "$CONFIRMAR" != "s" ]; then
    mostrar_mensaje "Operación cancelada"
    exit 0
fi

# Crear las líneas a añadir
NUEVAS_LINEAS=""
NUEVAS_LINEAS="$NUEVAS_LINEAS\n; ============================================================"
NUEVAS_LINEAS="$NUEVAS_LINEAS\n; SUBDOMINIO $SUBDOMINIO.marisma.intranet"
NUEVAS_LINEAS="$NUEVAS_LINEAS\n; ============================================================"
NUEVAS_LINEAS="$NUEVAS_LINEAS\nwww.$SUBDOMINIO   IN  A   $IP_WWW"
NUEVAS_LINEAS="$NUEVAS_LINEAS\nftp.$SUBDOMINIO   IN  A   $IP_FTP"
NUEVAS_LINEAS="$NUEVAS_LINEAS\nsmtp.$SUBDOMINIO  IN  A   $IP_SMTP"

# Añadir las líneas al archivo de zona
echo -e "$NUEVAS_LINEAS" | sudo tee -a "$ZONE_FILE" > /dev/null

mostrar_mensaje "Registros añadidos a $ZONE_FILE"

# Incrementar automáticamente el serial (opción avanzada)
mostrar_advertencia "Recuerda incrementar manualmente el serial en el archivo SOA"
mostrar_advertencia "Serial actual: busca la línea que contiene '; Serial' y aumenta el número"

# Verificar la sintaxis
mostrar_mensaje "Verificando sintaxis..."
if sudo named-checkzone marisma.intranet "$ZONE_FILE" > /dev/null 2>&1; then
    mostrar_mensaje "Sintaxis correcta ✅"
else
    mostrar_error "Error de sintaxis. Revisa el archivo"
    sudo named-checkzone marisma.intranet "$ZONE_FILE"
    exit 1
fi

# Reiniciar BIND9
mostrar_mensaje "Reiniciando BIND9..."
sudo systemctl restart bind9

# Verificar estado
if systemctl is-active --quiet bind9; then
    mostrar_mensaje "BIND9 reiniciado correctamente ✅"
else
    mostrar_error "BIND9 no pudo reiniciarse"
    sudo systemctl status bind9
    exit 1
fi

echo ""
echo "=========================================="
mostrar_mensaje "SUBDOMINIO CREADO CON ÉXITO"
echo "=========================================="
echo "Comprueba el funcionamiento con:"
echo "  dig www.$SUBDOMINIO.marisma.intranet"
echo "  dig ftp.$SUBDOMINIO.marisma.intranet"
echo "  dig smtp.$SUBDOMINIO.marisma.intranet"
```

<img width="730" height="421" alt="image" src="https://github.com/user-attachments/assets/75cb38e9-4c20-4ae1-8bbe-8d98b1f80012" />


### 4.3 Dar permisos de ejecución al script

```bash
sudo chmod +x /usr/local/bin/crear_subdominio.sh
```

### 4.4 Ejecutar el script para crear el subdominio informatica

```bash
sudo /usr/local/bin/crear_subdominio.sh
```

Datos a introducir:

```
Nombre del subdominio: informatica
IP base: 192.168.191.60
```

<img width="738" height="249" alt="image" src="https://github.com/user-attachments/assets/e02d67db-8ffd-48c9-8e90-48cf73869443" />

<img width="734" height="412" alt="image" src="https://github.com/user-attachments/assets/232b4591-d4f7-4479-939b-34b053c0c853" />


### 4.5 Verificar el resultado del script

```bash
# Verificar que los registros se añadieron correctamente
tail -20 /var/lib/bind/master/directa.marisma.db
```

<img width="625" height="383" alt="image" src="https://github.com/user-attachments/assets/4047cd03-7570-4211-81ea-8fcd2e207dbb" />

```
# Comprobar resolución
dig www.informatica.marisma.intranet
```

<img width="731" height="412" alt="image" src="https://github.com/user-attachments/assets/e595063e-971d-42d7-9d04-01f631344aee" />

---

## 5. Método 3: Uso de $INCLUDE (opcional)

Este método separa la configuración del subdominio en un archivo independiente, haciendo la zona principal más organizada y fácil de mantener.

### 5.1 Crear archivo independiente para el subdominio

```bash
sudo nano /var/lib/bind/master/db.informatica
```

**Contenido del archivo:**

```
; ============================================================
; ARCHIVO DE SUBDOMINIO: informatica.marisma.intranet
; ============================================================
; Este archivo es incluido desde directa.marisma.db
; ============================================================

; Registros A del subdominio informatica
www.informatica   IN  A   192.168.191.60
ftp.informatica   IN  A   192.168.191.61
smtp.informatica  IN  A   192.168.191.62
```

<img width="585" height="228" alt="image" src="https://github.com/user-attachments/assets/2491b2bb-0c1e-4832-97ae-6a496e72472b" />


### 5.2 Modificar el archivo de zona principal para incluir el subdominio

```bash
sudo nano /var/lib/bind/master/directa.marisma.db
```

Al final del archivo, añade:

```
; ============================================================
; INCLUSIÓN DE SUBDOMINIOS
; ============================================================
$INCLUDE /var/lib/bind/master/db.informatica
```

<img width="716" height="422" alt="image" src="https://github.com/user-attachments/assets/bc4dcc27-18e4-4c3b-add7-5bf75d65e032" />


**Explicación de `$INCLUDE`:**

- La directiva `$INCLUDE` le dice a BIND que procese el contenido del archivo especificado como si estuviera escrito directamente en ese lugar
- Es útil para organizar zonas grandes o para compartir configuración entre múltiples zonas
- En nuestro caso, separa la configuración del subdominio del dominio principal

### 5.3 Verificar la sintaxis

```bash
sudo named-checkzone marisma.intranet /var/lib/bind/master/directa.marisma.db
```

<img width="811" height="74" alt="image" src="https://github.com/user-attachments/assets/a632af55-e58a-461b-84ee-abf3b91c6100" />

### 5.4 Reiniciar BIND9

```bash
sudo systemctl restart bind9
```

---

## 6. Verificaciones finales

### 6.1 Consultar el host `www` del subdominio

```bash
dig www.informatica.marisma.intranet
```

Salida esperada:

```
;; ANSWER SECTION:
www.informatica.marisma.intranet. 86400 IN A 192.168.191.60
```

<img width="735" height="416" alt="image" src="https://github.com/user-attachments/assets/ec53a821-4827-4e24-a6f2-4c1b8d40654e" />


### 6.2 Consultar el host `ftp` del subdominio

```bash
dig ftp.informatica.marisma.intranet
```

Salida esperada:

```
;; ANSWER SECTION:
ftp.informatica.marisma.intranet. 86400 IN A 192.168.191.61
```

<img width="757" height="408" alt="image" src="https://github.com/user-attachments/assets/869c653a-0743-4b18-81d3-2c791aeafc0b" />


### 6.3 Consultar el host `smtp` del subdominio

```bash
dig smtp.informatica.marisma.intranet
```

Salida esperada:

```
;; ANSWER SECTION:
smtp.informatica.marisma.intranet. 86400 IN A 192.168.191.62
```

<img width="743" height="410" alt="image" src="https://github.com/user-attachments/assets/3a664957-4ccf-46c4-9866-4430403bf8e2" />


### 6.4 Verificar que el dominio principal sigue funcionando

```bash
dig www.marisma.intranet
dig ftp1.marisma.intranet
dig mail1.marisma.intranet
```

<img width="633" height="418" alt="image" src="https://github.com/user-attachments/assets/dfcf2048-7dcc-459c-bf37-5cd53cdc5d2f" />

<img width="647" height="411" alt="image" src="https://github.com/user-attachments/assets/b4fe5a8e-bf6c-4839-8e55-3586d634ee56" />

<img width="653" height="411" alt="image" src="https://github.com/user-attachments/assets/12bc120f-8f83-4baf-abcb-50a129761797" />


Todas deben resolver a sus IPs originales (`.30`, `.10`, `.20` respectivamente).

### 6.5 Consulta con nslookup (alternativa)

```bash
nslookup www.informatica.marisma.intranet
```

Salida esperada:

```
Name:   www.informatica.marisma.intranet
Address: 192.168.191.60
```

<img width="508" height="192" alt="image" src="https://github.com/user-attachments/assets/86645e59-c997-4589-8a7b-4e686da50bdf" />
