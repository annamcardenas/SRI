# Crea un script para cada uno de los siguientes problemas:

## 1. Crea un script que añada un puerto de escucha en el fichero de configuración de Apache. El puerto se recibirá como parámetro en la llamada y se comprobará que no esté ya presente en el fichero de configuración.

#!bin/bash

ARCHIVO=”/etc/apache2/ports.conf”

if [ $# -ne 1]; then
	echo “Error: Debes especificar un puerto”
	echo “Uso: $0 [puerto]”
	exit 1
fi


PUERTO=$1

if grep -q “Listen $PUERTO” “$ARCHIVO”; then
	echo “El puerto $PUERTO ya existe en la configuración.”
	echo “Líneas encontradas: ”
	grep “Listen $PUERTO” “$ARCHIVO”
	exit 1
else
	echo “Agregando puerto $PUERTO al archivo de configuración…”
	echo “Listen $PUERTO” » “$ARCHIVO”
fi


if grep -q “Listen $PUERTO” “$ARCHIVO”; then
	echo “Puerto $PUERTO agregado con éxito.”
	echo “Recuerda reiniciar Apache: sudo systemctl restart apache2”
else
	echo “Error: No se pudo agregar el puerto”
	exit 1
fi

## 2. Crea un script que añada un nombre de dominio y una ip al fichero hosts. Debemos comprobar que no existe dicho dominio en el fichero hosts

#!bin/bash


ARCHIVO=”/etc/hosts”

if [ $# -ne 2]; then
	echo “Error: Debes especificar una IP y un dominio.”
	exit 1
fi

IP=$1
DOMINIO=$2


if grep -q “$DOMINIO” “$ARCHIVO”; then
	echo “Error: El dominio $DOMINIO ya existe en la configuración.”
	grep “$DOMINIO” “$ARCHIVO”
	exit 1
else
	echo “Agregando dominio e IP al archivo de configuración…”
	echo “$IP $DOMINIO” » “$ARCHIVO”
fi

## 3. Crea un script que nos permita crear una página web con un título, una cabecera y un mensaje

#!bin/bash

if [ $# -ne 4]; then
	echo “Error: Parámetros incorrectos.”
	exit 1
fi

cat « HTML > $1
<html>
<head>
	$2
</head>
<body>
	<h1> $3 </h1>
	<p> $4 </p>
</body>
</html>

HTML

if [ $# -ne 1]; then
	echo “Error: Debes especificar un puerto”
	echo “Uso: $0 [puerto]”
	exit 1
fi
