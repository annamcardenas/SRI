# Actividad 6 - Creación de imágenes Docker

## Ejemplo 1: Imagen con página estática (HTML)

### Paso a paso

```bash
# 1. Crear directorio
mkdir ~/mi-web && cd ~/mi-web
```

<img width="508" height="61" alt="image" src="https://github.com/user-attachments/assets/13b55b27-67e5-4109-917e-639179024b31" />

# 2. Crear archivo HTML
```
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Mi página con Docker</title>
</head>
<body>
    <h1>Hola desde mi imagen Docker</h1>
    <p>Esta página se creó con un Dockerfile</p>
</body>
</html>
EOF
```

<img width="586" height="254" alt="image" src="https://github.com/user-attachments/assets/467ede09-72bc-4a3a-9368-4c026e76d348" />

# 3. Crear Dockerfile
```
cat > Dockerfile << 'EOF'
FROM httpd:alpine
COPY index.html /usr/local/apache2/htdocs/
EXPOSE 80
EOF
```

<img width="606" height="116" alt="image" src="https://github.com/user-attachments/assets/e9f8a10c-f5dc-431f-b1a9-c5781158630a" />

# 4. Construir la imagen
```
docker build -t mi-web .
```

<img width="1105" height="613" alt="image" src="https://github.com/user-attachments/assets/bd6948f8-e388-4ff2-b2ed-7d6ac4ef31ae" />

# 5. Ejecutar el contenedor
```
docker run -d -p 8085:80 --name mi-web-contenedor mi-web
```

<img width="795" height="66" alt="image" src="https://github.com/user-attachments/assets/d466d585-e757-47da-9df4-554f886033bc" />

# 6. Probar
```
curl localhost:8085
```
<img width="485" height="236" alt="image" src="https://github.com/user-attachments/assets/fa8b3e3c-bb12-45d4-a073-1c7d0f71e74e" />
