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
