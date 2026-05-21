# Actividad 2 - Introducción a los contenedores Docker

## 1. Hello World y primeros comandos

```bash
docker run hello-world
```

<img width="705" height="204" alt="image" src="https://github.com/user-attachments/assets/4826a14b-4ecd-42ec-b0db-4e835db84623" />

## 2. Ver contenedores (activos y detenidos):

```bash
docker ps -a
```

<img width="1068" height="84" alt="image" src="https://github.com/user-attachments/assets/e9f721d1-d4e0-401c-83a1-87c3dd8465df" />

## 3. Eliminar un contenedor:

```bash
docker rm <ID o nombre>
```

<img width="576" height="62" alt="image" src="https://github.com/user-attachments/assets/f411c510-890b-490c-9e95-3a6e7cab8837" />

## 4. Ver imágenes descargadas:

```bash
docker images
```
<img width="1107" height="104" alt="image" src="https://github.com/user-attachments/assets/a491b20a-bc55-4dd0-8005-d3ea300d8c09" />


## 5. Contenedor interactivo
Ejecutar una terminal interactiva dentro de Ubuntu:

```bash
docker run -it ubuntu bash
```

<img width="765" height="177" alt="image" src="https://github.com/user-attachments/assets/5ad98bce-ed21-4c99-b59f-6826c159191d" />

Dentro del contenedor se pueden ejecutar comandos (ls, apt update, etc.). Al salir (exit), el contenedor se detiene.

<img width="994" height="446" alt="image" src="https://github.com/user-attachments/assets/7f0030e2-8ed3-459d-83b7-787cafed05a6" />

## 6. Reconectar a un contenedor detenido:

```bash
docker start -ai <ID o nombre>
```
<img width="932" height="136" alt="image" src="https://github.com/user-attachments/assets/b9d98e43-0b64-4391-adb0-2623c974b99a" />

## 7. Ejecutar un comando específico sin entrar:

```bash
docker exec <ID o nombre> ls -la
```
<img width="730" height="538" alt="image" src="https://github.com/user-attachments/assets/954ea00b-23f6-49c2-8f22-baf4b7fbc44a" />

## 8. Ver metadatos del contenedor:

```bash
docker inspect <ID o nombre>
```
<img width="1095" height="442" alt="image" src="https://github.com/user-attachments/assets/14a44b9c-1cc8-48fa-acb5-484b92d1df42" />

## 9. Contenedor demonio (background)

```bash
docker run -d --name mi_demonio ubuntu bash -c "while true; do echo 'Hola Docker'; sleep 5; done"
```

<img width="1099" height="84" alt="image" src="https://github.com/user-attachments/assets/71121cc7-785a-40db-90f5-b317c9ab0c9b" />

Ver logs:

```bash
docker logs mi_demonio
```
<img width="580" height="231" alt="image" src="https://github.com/user-attachments/assets/50f09c99-e6a4-4de9-a282-91472d322316" />

Detener y eliminar:

```bash
docker stop mi_demonio
docker rm mi_demonio
```

<img width="595" height="123" alt="image" src="https://github.com/user-attachments/assets/bb6efe61-e236-4c45-8e18-3d12a269e0d4" />

## 10. Servidor web con Apache

```bash
docker run -d -p 8080:80 --name mi_web httpd
```

<img width="790" height="291" alt="image" src="https://github.com/user-attachments/assets/a304b802-c8ba-4f04-ab43-079457e95f76" />

Acceder desde el navegador o con curl:

```bash
curl localhost:8080
```

<img width="829" height="216" alt="image" src="https://github.com/user-attachments/assets/f0b640e1-107b-4e7c-8506-56a8467a6ce3" />

Modificar la página de inicio:

```bash
docker exec -it mi_web bash
cd /usr/local/apache2/htdocs/
echo "<h1>Mi prop prop web con Docker</h1>" > index.html
exit
```

<img width="934" height="312" alt="image" src="https://github.com/user-attachments/assets/7ecd5424-7f99-48cd-89ed-dbd1e0ab94f1" />

## 11. Variables de entorno
Ejemplo con MariaDB:

```
bash
docker run -d --name mi_mariadb -e MYSQL_ROOT_PASSWORD=mi_password -e MYSQL_DATABASE=mi_db mariadb
```

<img width="1094" height="268" alt="image" src="https://github.com/user-attachments/assets/c7a024fb-92c2-4c11-a468-bf4053f3503e" />

Verificar variables dentro del contenedor:

```bash
docker exec -it mi_mariadb env
```

<img width="695" height="217" alt="image" src="https://github.com/user-attachments/assets/2a2ee813-5426-4133-9d27-2eedfba0295f" />

Acceder a la base de datos:

```bash
docker exec -it mi_mariadb mysql -u root -p
```
<img width="782" height="445" alt="image" src="https://github.com/user-attachments/assets/a8a65eab-17fb-44cc-9010-6481e75abea5" />
