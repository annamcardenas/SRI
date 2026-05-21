# Actividad 3 - Pull, ejecución y borrado de contenedores

## Descargar imágenes

```bash
docker pull ubuntu
docker pull hello-world
docker pull nginx
```

<img width="678" height="471" alt="image" src="https://github.com/user-attachments/assets/d2872742-bdde-452d-8733-d6412e58e326" />

## Listar imágenes descargadas

```bash
docker images
```

<img width="1091" height="171" alt="image" src="https://github.com/user-attachments/assets/aa112edb-3d7d-43b6-a9fa-7f3708acabb6" />

## Ejecutar contenedores con nombre

```bash
docker run --name myhello1 hello-world
docker run --name myhello2 hello-world
docker run --name myhello3 hello-world
```

<img width="751" height="426" alt="image" src="https://github.com/user-attachments/assets/cd1b0ac6-c199-41d9-8c09-0611df25ebd4" />

<img width="739" height="433" alt="image" src="https://github.com/user-attachments/assets/195aa6cc-f992-491b-b0bc-54b1658c0d55" />

<img width="750" height="453" alt="image" src="https://github.com/user-attachments/assets/4af9d20d-af89-46cf-b32a-7836e171b141" />

## Ver contenedores activos

```bash
docker ps
```

<img width="1104" height="194" alt="image" src="https://github.com/user-attachments/assets/5890ec94-d342-4f2f-938c-3f7371de29d2" />

Nota: Los contenedores hello-world se detienen automáticamente al terminar, por eso no aparecen en docker ps.


## Ver todos los contenedores (incluidos detenidos)

```bash
docker ps -a
```

<img width="1101" height="312" alt="image" src="https://github.com/user-attachments/assets/b2300676-403c-4883-9d85-6ddff44a8a95" />

## Detener contenedores

```bash
docker stop myhello1
docker stop myhello2
```

<img width="572" height="103" alt="image" src="https://github.com/user-attachments/assets/7027dc37-ffc3-490d-9010-f41970314b7d" />

## Borrar un contenedor

```bash
docker rm myhello1
```

<img width="578" height="65" alt="image" src="https://github.com/user-attachments/assets/ce746a3c-2fa0-44ea-b1ac-d92788d117ff" />

## Ver contenedores activos nuevamente

```bash
docker ps
```

<img width="1093" height="182" alt="image" src="https://github.com/user-attachments/assets/99b2d84b-7888-4435-8953-d979e5def65e" />

## Borrar todos los contenedores

```bash
docker rm $(docker ps -a -q)
```

<img width="1105" height="354" alt="image" src="https://github.com/user-attachments/assets/e7862a11-73c0-4911-beb2-ded949acc992" />

## Verificar que no quedan contenedores

```bash
docker ps -a
```

<img width="706" height="67" alt="image" src="https://github.com/user-attachments/assets/b8ebf2fd-b388-4d61-864c-1dab7ea5a684" />
