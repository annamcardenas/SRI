# Actividad 4 - Almacenamiento y redes en Docker

## Ejemplo 1: Volumen Docker (persistencia de datos)

**Objetivo:** Crear un volumen que sobreviva al contenedor.

# Crear un volumen
```
docker volume create mi-volumen
```

<img width="665" height="67" alt="image" src="https://github.com/user-attachments/assets/ab40c6ab-4ea2-4648-931a-e8d779bfff90" />

# Ver volúmenes existentes
```
docker volume ls
```
<img width="590" height="107" alt="image" src="https://github.com/user-attachments/assets/f78950c8-646a-49fb-8873-28589a86f4cc" />

# Ejecutar contenedor con el volumen montado
```
docker run -d --name contenedor1 -v mi-volumen:/datos alpine tail -f /dev/null
```

<img width="968" height="181" alt="image" src="https://github.com/user-attachments/assets/97c4380a-1242-422f-8ef6-f03f20064875" />

# Escribir datos dentro del contenedor
```
docker exec contenedor1 sh -c "echo 'Hola persistente' > /datos/prueba.txt"
```

<img width="1062" height="53" alt="image" src="https://github.com/user-attachments/assets/54e2ea18-6b5a-41e1-b3fd-435fa08e2014" />

# Eliminar el contenedor
```
docker rm -f contenedor1
```

<img width="612" height="67" alt="image" src="https://github.com/user-attachments/assets/e9f70eae-5f43-489d-b0f4-a49ddd6caf0a" />

# Crear un nuevo contenedor con el mismo volumen
```
docker run -d --name contenedor2 -v mi-volumen:/datos alpine tail -f /dev/null
```

<img width="1081" height="71" alt="image" src="https://github.com/user-attachments/assets/729af899-0378-426a-9df3-50465fead3a4" />

# Verificar que los datos siguen ahí
```
docker exec contenedor2 cat /datos/prueba.txt
```

<img width="789" height="69" alt="image" src="https://github.com/user-attachments/assets/fca051d5-b41c-47f4-9d7d-68a1b6732060" />

# Conclusión: El volumen mantiene los datos aunque el contenedor original se elimine.

## Ejemplo 2: Bind mount (montar directorio del host)
# Objetivo: Compartir una carpeta del sistema host con el contenedor.

# Crear un directorio en el host
```
mkdir -p ~/docker/html
```

<img width="659" height="46" alt="image" src="https://github.com/user-attachments/assets/d31274d1-e2c8-48b1-b67a-db30d462ffe8" />

# Crear un archivo HTML de prueba
```
echo "<h1>Bind mount funciona</h1>" > ~/docker/html/index.html
```

<img width="964" height="47" alt="image" src="https://github.com/user-attachments/assets/1f10edd7-85ce-4e59-88f4-07ae937eaa63" />

# Ejecutar nginx con bind mount
```
docker run -d --name mi-web -p 8080:80 -v ~/docker/html:/usr/local/apache2/htdocs/ httpd
```

<img width="1101" height="83" alt="image" src="https://github.com/user-attachments/assets/3c3437ef-4012-4f43-8241-033bb4061839" />

# Probar acceso
```
curl localhost:8080
```

<img width="563" height="70" alt="image" src="https://github.com/user-attachments/assets/2a63f4ff-3449-42a2-88b6-ce486d43025a" />

# Conclusión: Los cambios en el host se reflejan instantáneamente en el contenedor.

## Ejemplo 3: Red personalizada (bridge)
# Objetivo: Crear una red aislada para comunicar contenedores por nombre.

# Crear una red personalizada
```
docker network create mi-red
```

<img width="671" height="65" alt="image" src="https://github.com/user-attachments/assets/d7197afd-613e-4aef-a080-8027914d53e2" />

# Crear dos contenedores en la misma red
```
docker run -d --name contenedor-a --network mi-red alpine tail -f /dev/null
docker run -d --name contenedor-b --network mi-red alpine tail -f /dev/null
```

<img width="1066" height="104" alt="image" src="https://github.com/user-attachments/assets/18a601b6-0b31-4a45-8cf5-3c028ad13129" />

# Verificar comunicación por nombre
```
docker exec contenedor-a ping -c 2 contenedor-b
```

<img width="839" height="177" alt="image" src="https://github.com/user-attachments/assets/9773d86e-5a62-42ad-837b-5f3caf4f5311" />

# Inspeccionar la red
```
docker network inspect mi-red
```

<img width="835" height="580" alt="image" src="https://github.com/user-attachments/assets/394adf2b-793d-4812-9306-1abd0577143c" />

# Limpiar
```
docker rm -f contenedor-a contenedor-b
docker network rm mi-red
```

<img width="747" height="127" alt="image" src="https://github.com/user-attachments/assets/3ee376ca-b863-41d9-b3ac-fbd7ad78c2b5" />

# Conclusión: Los contenedores se comunican usando sus nombres como si fueran DNS.


