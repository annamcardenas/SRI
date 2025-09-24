# Actividad 0.2 - UDP and TCP: Comparison of Transport Protocols

## Diferencias entre udp y tcp? (min 2:46 y 4:15)

**TCP** es como una llamada telefónica formal: establece conexión, verifica que todo llega en orden y retransmite si se pierde algo. Es confiable pero con más overhead.

**UDP** es como lanzar una postal: sin confirmación de llegada, más rápido pero menos fiable. Ideal cuando la velocidad es prioritaria.

TCP = Conexión segura | UDP = Datagramas rápidos

## ¿Qué aplicaciones usan tcp?  http, smtp, pop, imap, ssh

Las que necesitan fiabilidad absoluta, como:

**HTTP/HTTPS** - Navegación web

**SMTP/POP/IMAP** - Emails

**SSH** - Conexiones seguras

**FTP** - Transferencia de archivos

**Bases de datos, transacciones bancarias..**

## ¿Qué aplicaciones usan udp?

Las que priorizan velocidad sobre perfección, como:

**DNS** - Resolución de nombres

**Videollamadas** (Zoom, Skype)

**Streaming** (YouTube, Netflix)

**Juegos online**

**VoIP** - Llamadas por internet

## ¿Qué capa almacena el puerto?

Los puertos viven en la capa de transporte (TCP/UDP). Son como números de departamento en un edificio, dirigiendo el tráfico a la aplicación correcta.

## ¿Qué capa almacena la dirección IP?

Las direcciones IP operan en la capa de red. Son como la dirección del edificio completo en la ciudad digital.

## ¿Qué es three-way handshake?

Es el ritual de saludo de TCP para establecer conexión:

**SYN** - Cliente: "¿Puedo conectar?"

**SYN-ACK** - Servidor: "¡Sí, conectemos!"

**ACK** - Cliente: "¡Perfecto, empezamos!"