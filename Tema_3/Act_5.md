<img width="1102" height="217" alt="image" src="https://github.com/user-attachments/assets/3baa182b-1fe6-4bdc-97a2-7c31b754a1d5" /># Actividad 5 - Docker Compose

## Instalación de Docker Compose

En versiones recientes de Docker, Compose ya viene incluido. Verificar:

```bash
docker compose version
```

<img width="597" height="71" alt="image" src="https://github.com/user-attachments/assets/5c664e1e-5609-47b3-b148-e6b55307d66b" />

---

## Ejemplo 1: Guestbook (aplicación con Redis)

**Objetivo:** Desplegar una aplicación Guestbook que usa Redis como almacenamiento.

### Paso a paso

```bash
# 1. Crear directorio y entrar
mkdir ~/guestbook && cd ~/guestbook
```

<img width="708" height="43" alt="image" src="https://github.com/user-attachments/assets/c2b17971-e5e6-48da-86cb-ce56c3de419b" />

# 2. Crear el archivo docker-compose.yml
```
cat > docker-compose.yml << 'EOF'
services:
  app:
    image: iesgn/guestbook
    ports:
      - "8080:5000"
    depends_on:
      - db
  db:
    image: redis
EOF
```

<img width="746" height="314" alt="image" src="https://github.com/user-attachments/assets/1c5afedb-b078-4c1f-a3cf-7b65ab408a1e" />

# 3. Verificar que el archivo se creó correctamente
```
cat docker-compose.yml
```

<img width="571" height="215" alt="image" src="https://github.com/user-attachments/assets/7b72528f-46d2-4643-a147-17bfaae61791" />


# 4. Levantar los servicios en segundo plano
```
docker compose up -d
```

<img width="1092" height="138" alt="image" src="https://github.com/user-attachments/assets/b8be338a-dc35-49f0-a5d2-a011b10b0911" />

# 5. Verificar que los contenedores están corriendo
```
docker compose ps
```

<img width="971" height="82" alt="image" src="https://github.com/user-attachments/assets/5335cba9-a2ee-4b40-b0ac-6abe1e0c9560" />

# 6. Ver logs (opcional)
```
docker compose logs
```

<img width="1097" height="206" alt="image" src="https://github.com/user-attachments/assets/a2209279-ea9b-4325-905e-7529e76bdf92" />

# 7. Probar la aplicación
```
curl localhost:3000
```

<img width="1101" height="442" alt="image" src="https://github.com/user-attachments/assets/6624f9bf-b83b-4a13-9a51-9deced68c445" />

### Limpiar

```bash
# Detener y eliminar contenedores
docker compose down
```

<img width="1103" height="136" alt="image" src="https://github.com/user-attachments/assets/8e8db969-f0b6-4604-81d3-2a388dd421a2" />

# Volver al directorio anterior
```
cd ..
```
<img width="381" height="44" alt="image" src="https://github.com/user-attachments/assets/25e5db97-d52d-4ffe-a4ea-f3eeddbbca98" />

---

## Ejemplo 2: Wordpress + MariaDB

**Objetivo:** Desplegar Wordpress con base de datos MariaDB usando volúmenes.

### Paso a paso

```bash
# 1. Crear directorio y entrar
mkdir ~/wordpress && cd ~/wordpress
```

<img width="537" height="40" alt="image" src="https://github.com/user-attachments/assets/2202b144-2de9-4647-a032-360c9f36da74" />

# 2. Crear el archivo docker-compose.yml
```
cat > docker-compose.yml << 'EOF'
services:
  db:
    image: mariadb:10.6
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    restart: always
  wordpress:
    image: wordpress:latest
    ports:
      - "8083:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db
    restart: always

volumes:
  db_data:
  wp_data:
EOF
```

<img width="687" height="587" alt="image" src="https://github.com/user-attachments/assets/d99bfe1e-55cd-496e-a2d0-974020bb22a4" />

# 3. Verificar el archivo
```
cat docker-compose.yml
```

<img width="528" height="558" alt="image" src="https://github.com/user-attachments/assets/aba09159-71ce-4789-9ade-0c5638a8931b" />

# 4. Levantar los servicios
```
docker compose up -d
```

<img width="1113" height="200" alt="image" src="https://github.com/user-attachments/assets/53886348-dce4-4a96-b94d-6e24a1e53f0b" />

# 5. Verificar estado de los contenedores
```
docker compose ps
```

<img width="1114" height="123" alt="image" src="https://github.com/user-attachments/assets/1b366c7c-f100-483d-b19b-7fe41dc43c06" />

# 6. Ver volúmenes creados
```
docker volume ls
```

<img width="685" height="137" alt="image" src="https://github.com/user-attachments/assets/480970d1-1d4f-4b6f-a5a1-04e6ef2a5b1a" />

### Acceder

Abrir navegador en `http://localhost:8080` y seguir la instalación de Wordpress.

### Limpiar

```bash
# Detener, eliminar contenedores y volúmenes (-v)
docker compose down -v

cd ..
```

<img width="1113" height="178" alt="image" src="https://github.com/user-attachments/assets/7ebc6b18-a241-4097-b957-98b1db5c6618" />

---

## Ejemplo 3: Temperaturas (aplicación con API)

**Objetivo:** Desplegar una aplicación simple que consulta temperaturas.

### Paso a paso

```bash
# 1. Crear directorio y entrar
mkdir ~/temperaturas && cd ~/temperaturas
```

<img width="613" height="48" alt="image" src="https://github.com/user-attachments/assets/ac0da798-c306-4dbc-93a9-05433fb3946b" />

# 2. Crear el archivo docker-compose.yml
```
cat > docker-compose.yml << 'EOF'
services:
  backend:
    image: iesgn/temperaturas_backend
  frontend:
    image: iesgn/temperaturas_frontend
    ports:
      - "8084:3000"
    depends_on:
      - backend
EOF
```

<img width="651" height="229" alt="image" src="https://github.com/user-attachments/assets/c38f3294-8600-4f79-a8bb-4a1820352ce6" />

# 3. Verificar el archivo
```
cat docker-compose.yml
```

<img width="567" height="227" alt="image" src="https://github.com/user-attachments/assets/0fef6b63-0147-4e35-b8b5-a5b98667fa4a" />

# 4. Levantar los servicios
```
docker compose up -d
```

<img width="1114" height="164" alt="image" src="https://github.com/user-attachments/assets/4295da74-62ab-4145-ba71-f09a0217a0cc" />

# 5. Verificar estado
```
docker compose ps
```

<img width="1103" height="157" alt="image" src="https://github.com/user-attachments/assets/2443b316-b69c-4b4d-9c80-ffafb2f68d9b" />

# 6. Probar la aplicación
```
curl http://localhost:3000
```

<img width="897" height="577" alt="image" src="https://github.com/user-attachments/assets/5d8189b4-8b12-4cf5-8c59-f95ec5f2abeb" />

### Limpiar

```bash
docker compose down

cd ..
```

<img width="1120" height="140" alt="image" src="https://github.com/user-attachments/assets/1c831d75-502f-4e72-8dda-2b9465d17303" />

---

## Estructura de un docker-compose.yml

```yaml
version: '3.8'
services:
  nombre_servicio:
    image: imagen:tag
    ports:
      - "host:contenedor"
    volumes:
      - volumen:/ruta
    environment:
      - VARIABLE=valor
    depends_on:
      - otro_servicio

volumes:
  nombre_volumen:
```
