---
layout: single
title: Cliente-servidor basico del protocolo de red TCP y UDP
excerpt: "En ocasiones nos encontramos en maquinas que no cuentan con herramientas para ejecutar ataques de red, por lo que es mejor desarrollar nuestras propias herramientas y llevarlas en nuestras cabeza en todo momento."
date: 2022-07-17
classes: wide 
header:
 teaser: /assets/images/TCP-network/tcp0.png
 teaser_home_page: true
categories:
- programacion
tags:
- seguridad ofensiva
- python
- conceptos
- herramientas
- protocolo de red
---
<p align="center" >
<img src='/assets/images/TCP-network/tcp0.png' width='300' height='300'/>
</p>

Se dice que el escenario mas interesante para un hacker, es la red. Con un simple acceso a la red, ya podemos molestarle la vida a alguien mas, pero no solo eso, tambien podemos probar cosas, como inyeccion de paquetes, explotar los hosts, buscar hosts, recolectar datos para crear informacion.

Pero bueno que es eso de TCP, cliente-servidor? Usando el lenguaje python y su variedad de librerias o modulos, en especial usaremos el modulo socket porque con este solo modulo ya tenemos lo necesario para crear clientes y servidores del protocolo TCP, no solo eso, tambien del protocolo UDP.

<p align="center" >
<img src='/assets/images/TCP-network/TCP-connection.png'/>
</p>

# TCP

TCP significa Protocolo de Transmision (TCP). Un ejemplo del dia a dia es una comunicacion por via telefonica, en la que el emisor y el receptor establecen una comunicacion entre ellos, asi similar es como funciona el protocolo TCP. Pero en este caso la comunicacion es realizada en base a paquetes o segmentos.

## Cliente TCP

Normalmente hacemos uso de un cliente TCP ya sea para probar servicios, enviar datos, fuzzear entre otras tareas. Pero existen ocasiones que en el entorno donde nos encontramos no cuenta con ninguna herramienta de red o compiladores, aqui es donde los verdaderos ninjas usan su cabeza para pensar y tener la capacidad de construir un propio cliente TCP ya que es bastante util.

### codigo cliente tcp basico
```python

# Primero y lo mas importante, importamos el modulo socket
import socket

# Definimos las variables
HOST = "www.google.com" # ejemplo 0.0.0.0, www.test.com
PORT = 80 # ejemplo 80:http, 22:ssh, 21:ftp

# Aqui creamos un objeto de socket
# AF_INET: parametro que indica que usaremos direcciones o nombres de hosts IPv4 estandar
# SOCK_STREAM: indica que sera un cliente TCP
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 

# Realizamos la conexion del cliente, pasandole las variables que definimos anteriormente
client.connect((HOST,PORT))

# Enviamos los datos o mejor dicho los bytes
client.send(b'GET / HTTP/1.1\r\nHost:google.com\r\n\r\n')

# Recibimos los datos correspondientes
response = client.recv(4096)

# imprimir lo que se recibio
print(response.decode('utf-8'))

# cerramos el socket
client.close()
```
Esta forma es la mas basica de escribir un cliente tcp, a simple vista sabemos que la conexion tendra exito, que el servidor siempre espera algun dato y que tambien nos devolvera datos, la finalidad de estas herramientas es que sean rapidas y sucias para los trabajos que requieran reconocimiento o explotacion

<p align="center" >
<img src='/assets/images/TCP-network/udp-client.png' width="500" />
</p>


# UDP

UDP significa Protocolo de Datagramas de Usario, es un protocolo del nivel de capa de transporte, basado en la transmision sin conexion de datagramas, es decir, la alternativa a TCP

## Cliente UDP

El codigo para crear un cliente UDP es similar al del TCP solo que algunos cambios, solo dos cambios

### codigo cliente udp basico
```python


```
