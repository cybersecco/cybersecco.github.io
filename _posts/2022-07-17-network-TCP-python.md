---
layout: single
title: Aprende a crear una aplicación cliente-servidor con TCP y UDP en Python
excerpt: "Tutorial sobre la creación de un cliente-servidor básico utilizando el protocolo de red TCP y UDP en Python! En este tutorial, aprenderás cómo utilizar las funcionalidades de sockets de Python para crear una aplicación cliente-servidor. Aprenderás sobre la diferencia entre el protocolo TCP y UDP y cuándo utilizar uno u otro. Además, profundizarás en los conceptos de conexión, comunicación y transmisión de datos en tiempo real a través de redes. "
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

El codigo para crear un cliente UDP es similar al del TCP solo que algunos cambios, solo dos.

### codigo cliente udp basico
```python
# Cambiamos el parametro de SOCK_STREAM por SOCK_DGRAM
client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM) 

# Para enviar los datos usamos la funcion sendto() pasandole los parametros de host y port
client.sendto(b'test',(target_host, target_port))

# Recibimos los datos con recvfrom()
data, address = client.recvfrom(4096)

#Imprimimos el mnsaje y cerramos el cliente
print(data.decode())
client.close()

```
Estos fragmentos de codigo son lo suficientemente rapidos y faciles para tareas diarias en el mundo de los hackers.

<p align="center" >
<img src='/assets/images/TCP-network/server-tcp.jpeg' width="500" />
</p>

# Servidor TCP

Recordemos que un servidor es una aplicacion que ofrece un servicio a usuarios de internet y TCP es un protocolo orientado a conexion. Entonces un usuario invoca la parte cliente de la aplicacion, la cual esta construye una solicitud para ese servicio y se la envia al servidor de la aplicacion que usa TCP/IP.

Asi si creamos nuestro propio servidor TCP podemos hacer uso de las posibles reverse shells que creemos o simplemente un shell de comandos o inclusive al crear un proxy.

```python

# Como siempre hacemos uso del modulo socket 
import socket

# Importamos el modulo threading para hacer posible la programacion con hilos
import threading

# Definimos las variables 
IP = '0.0.0.0'
PORT = 9998

# creamos la funcion principal donde creamos el objeto de socket llamado en este caso server.
def main():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    #Le pasamos las variables con la funcion bind()
    server.bind((IP, PORT))
    
    # Y ponemos el servidor en escucha con una acumulacion maxima de conexiones establecida en 5
    server.listen(5)

    print(f'[*] Listening on {IP}:{PORT}')

    # Colocamos el servidor en un bucle principal, claro dentro dela funcion principal donde espera una conexion entrante.
    while True:

        # Cuando el cliente se conecta, se recibe socket del cliente en la variable de cliente y los detalles de la conexion remota
        client, address = server.accept()
        print(f'[*] Accepted connection from {address[0]}:{address[1]}')
        client_handler = threading.Thread(target=handle_client, args=(client,))
        client_handler.start()

    # Creamos una funcion dentro de la funcion principal donde la pasamos como
    # argumento el socket_client
    def handle_client(client_socket):
        
        # Aqui le indicamos manejar la conexion de cliente, momento en el
        # que el programa esta listo
        with client_socket as sock:
            request = sock.recv(1024)
            print(f'[*] Received: {request.decode("utf-8")}')
            sock.send(b'ACK')


if __name__ == '__main__':
    main()
```
<br>

# Pruebas

---

Pasemos ahora a hacer algunas pruebas de conexion y demas para verificar si todo anda bien, como hacemos estas pruebas?. Ya tenemos un cliente TCP, ahora tenemos el servidor TCP, vamos a configurar el codigo para crear la conexion entre ambos, esto modificando las variables de entrada las cuales son la IP y el PORT

![](/assets/images/TCP-network/codeTcp.png)

![](/assets/images/TCP-network/consoleTcp.png)

Despues de cambiado el puerto y la ip en el cliente TCP del lado de la victima y de haber sido enviado el mensaje, pasamos a correr el servidor desde la maquina del atacante y a ponerlo en escucha a ver que recibimos.

![](/assets/images/TCP-network/consolanicol.png)

Y ahi vemos el mensaje "Esto es una prueba" que enviamos desde el cliente TCP, lo hemos recibo en el servidor TCP y ademas recibimos una confirmacion que fue "ACK" y esto quiere acknowledgement, este mensaje de comunicacion entre computadoras y hace referencia a la confirmacion del mensaje