---
layout: single
title: "Retro - Try Hack Me"
excerpt: "En esta maquina vamos a explorar un servidor Windows utilizando técnicas de fuerza bruta para encontrar una ruta que nos lleve a un blog de WordPress. Una vez allí, revisaremos el sitio para encontrar el usuario y la contraseña de acceso al administrador del blog. Utilizando esta información, nos conectaremos al sitio y nos montaremos una reverse shell para obtener acceso al sistema. Finalmente, utilizaremos JuicyPotato para escalar privilegios y obtener un control total del sistema."
date: 2023-01-09
classes: wide
header:
 teaser: /assets/images/THM/retro/retro.jpg
 teaser_home_page: true
 icon: /assets/images/thm.webp
categories:
  - THM
  - Windows
tags:
  - hacking web
  - wordpress
  - reverse shell php
---

![](/assets/images/THM/retro/retro.jpg)

## Introducción

En esta maquina vamos a explorar un servidor Windows utilizando técnicas de fuerza bruta para encontrar una ruta que nos lleve a un blog de WordPress. Una vez allí, revisaremos el sitio para encontrar el usuario y la contraseña de acceso al administrador del blog. Utilizando esta información, nos conectaremos al sitio y nos montaremos una reverse shell para obtener acceso al sistema. Finalmente, utilizaremos JuicyPotato para escalar privilegios y obtener un control total del sistema.

## Enumeración

Para empezar con la enumeración, siempre lanzo un ping a la maquina objetivo para saber su estado, encendida o no y ademas ver su TTL para saber con que sistema operativo me encuentro

Comando para lanzar el ping 

`ping -c 1 <IP Victima>`

Pero leyendo la informacion de la maquina, nos dice que esta maquina no responde al comando ping, por lo que podria hacer uso de alguna herramienta que me permita ejecutar un ping, entonces haciendo uso de una herramienta que esta en github y se llama [Tcp Ping](https://github.com/cloverstd/tcping), con ella intentare lanzar un ping a ver que nos dice

![](/assets/images/THM/retro/1.png)

El resultado es que si esta activa, asi que pase a usar nmap y hacer un escaneo potente con el siguiente comando.

`nmap -p- --open -sS --min-rate 5000 -n -Pn -v -sCV -oN targeted <IP-Objetivo>`

Este comando es un escaneo intensivo que busca todos los puertos abiertos en la dirección IP objetivo. Algunos de los parámetros utilizados incluyen:

* -p- : Escanea todos los puertos
* --open : Solo muestra los puertos abiertos
* -sS : Realiza un escaneo TCP SYN
* -n : Desactiva la resolución de nombres
* -Pn : Supone que todos los hosts están activos
* -v : Muestra información detallada
* -sCV : Realiza un escaneo de detección de versión
* -oN : Guarda el resultado en un archivo de texto

![](/assets/images/THM/retro/2.png)

Despues de la enumeración realizada vi que hay dos puertos abiertos en esta maquina objetivo y son

* Puerto 80 
* Puerto 3389

Cuando trabajo con máquinas Windows, suelo agregar una línea en el archivo /etc/hosts que asocia el nombre de la máquina con su dirección IP. Esto puede ser útil para facilitar la identificación y acceso a la máquina en cuestión y mejorar la eficiencia de los procesos de enumeración y explotación.

```
# Agregar una entrada al archivo /etc/hosts
sudo nano /etc/hosts

# Añadir una línea con la dirección IP y el nombre de la máquina
<IP-Objetivo> <Nombre-de-la-Máquina>
```

Y ingreso al dominio desde la web.

![](/assets/images/THM/retro/3.png)

Me encontre con IIS "Internet Information Services", un servidor web de Windows por defecto, así que use fuerza bruta de rutas de directorios con la herramienta [Go Buster](https://github.com/OJ/gobuster)

Comando de Go Buster

`gobuster dir -u http://retro.thm -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 --no-error`

El comando `gobuster dir` es una herramienta de descubrimiento de directorios que se utiliza para enumerar y descubrir directorios y archivos en un servidor web. La opción `-u` especifica la URL base a la que se realizará la petición. En este caso, se está utilizando http://retro.thm.

La opción `-w` especifica un archivo de lista de palabras que se usará como diccionario para la búsqueda de directorios. En este caso, se está utilizando un archivo llamado `directory-list-2.3-medium.txt` ubicado en `/usr/share/SecLists/Discovery/Web-Content/`.

La opción `-t` especifica el número de hilos que se utilizarán para realizar las peticiones. En este caso, se está utilizando un valor de `100` hilos.

La opción `--no-error` especifica que no se deben mostrar mensajes de error.

![](/assets/images/THM/retro/4.png)

El resultado del escaneo es una ruta de acceso encontrada en el servidor del puerto 80.

Ingresé 

![](/assets/images/THM/retro/5.png)

Hice uso de la herramienta What Web que es la que nos brinda detalles relevantes sobre un sitio web específico.

![](/assets/images/THM/retro/6.png)

Pude ver que el servidor esta corriendo Wordpress en la version 5.2.1

Pase a buscar exploits para ese WordPress en searchsploit

![](/assets/images/THM/retro/7.png)

Por lo que intenté otro escaneo de gobuster pero incluyendo la ruta /retro y que busque archivos por extension con la opcion `-x`

![](/assets/images/THM/retro/8.png)

Vemos la ruta del panel de inicio de inicio de sesion de wordpress y efectivamente es el panel.

![](/assets/images/THM/retro/9.png)

Buscando en la pagina algun posible usuario, encontré uno que resalto y era Wade y siguiendo buscando, encontre 1 comentario a uno de los post y el comentario al final decia parzival

![](/assets/images/THM/retro/10.png)

![](/assets/images/THM/retro/11.png)

Podemos ver que el usuario existe por este simple error de la version que tiene el wordpress, entonces probé con la palabra `parzival` que anteriormente vi en el comentario al post de Wade

Y efectivo `parzival` es la contraseña del usuario Wade en Wordpress.

![](/assets/images/THM/retro/12.png)

Recordando que el puerto 3389 estaba abierto pase a probar en el servicio RDP las credenciales de WordPress.

Con el siguiente comando puedo acceder a ese servicio con una interfaz grafica

`xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:retro.thm /u:wade /p:'parzival'`

![](/assets/images/THM/retro/13.png)

Aqui ya pude obtener la primer flag que es `user.txt`

## Escalada de Privilegios






