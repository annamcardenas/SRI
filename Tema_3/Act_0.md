# Actividad 0 - Docker

## Conceptos básicos sobre Docker
Docker es una plataforma open source que estandariza la entrega de software mediante contenedores. 
Aísla la aplicación del entorno de ejecución, eliminando la dependencia del sistema operativo subyacente. 
Esto resuelve las inconsistencias entre entornos de desarrollo, prueba y producción.

## Imágenes vs. Contenedores
- **Imagen Docker**: Capa estática e inmutable. Es un artefacto que incluye el sistema de archivos, las variables de entorno y el comando de inicio. Se distribuye a través de registros como Docker Hub.
- **Contenedor Docker**: Proceso en ejecución sobre una imagen. Al arrancar, añade una capa de escritura sobre la imagen base. Es desechable por diseño, aunque su estado puede persistirse mediante volúmenes.

## Volúmenes de almacenamiento
Los contenedores son transitorios. Al eliminarlos, cualquier dato escrito en su sistema de archivos interno se pierde. Los volúmenes permiten desacoplar los datos del ciclo de vida del contenedor. 
Se montan desde el host o desde otros contenedores, soportan copias de seguridad y pueden ser gestionados independientemente.
