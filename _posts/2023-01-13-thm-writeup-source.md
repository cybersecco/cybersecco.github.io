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
  - Webmin
  - Metasploit
---

![](/assets/images/THM/source/source.png)

## Introducción

En este Write-Up, exploraremos la máquina "Source" en la plataforma TryHackMe. Durante la investigación, descubrimos una aplicación web vulnerable llamada WebMin, que se utilizó para obtener acceso a la máquina. Utilizamos un exploit de Metasploit para explotar la vulnerabilidad y ganar acceso a los sistemas y datos de la máquina.

## Enumeración

Como siempre de primeras lanzo un PING a la máquina objetivo, con el fin de identificar si está encendida o no la máquina y saber su sistema operativo, porque si es Linux debe tener un TTL (Time to live) aproximado o igual a 64 y si es un sistema Windows debe tener un TTL aproximado o igual a 128.

```
❯ ping -c 1 10.10.211.234
PING 10.10.211.234 (10.10.211.234) 56(84) bytes of data.
64 bytes from 10.10.211.234: icmp_seq=1 ttl=63 time=445 ms

--- 10.10.211.234 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 445.194/445.194/445.194/0.000 ms
```

### Escaneo de puertos TCP

Haciendo uso de la herramienta nmap, voy a realizar un escaneo a todo el rango de puertos con las respectivas versiones y servicios que están corriendo bajo esa IP.

El comando que use para este escaneo es el siguiente:

`nmap -p- --open -sS --min-rate 5000 -n -Pn -v -oN targeted 10.10.249.51`

El resultado del escaneo nos dice

```
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: nmap/targeted
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # Nmap 7.93 scan initiated Wed Feb  1 17:03:35 2023 as: nmap -sCV -p22,10000 -oN targeted 10.10.135.94
   2   │ Nmap scan report for 10.10.135.94
   3   │ Host is up (0.22s latency).
   4   │ 
   5   │ PORT      STATE SERVICE VERSION
   6   │ 22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
   7   │ | ssh-hostkey: 
   8   │ |   2048 b74cd0bde27b1b15722764562915ea23 (RSA)
   9   │ |   256 b78523114f44fa22008e40775ecf287c (ECDSA)
  10   │ |_  256 a9fe4b82bf893459365becdac2d395ce (ED25519)
  11   │ 10000/tcp open  http    MiniServ 1.890 (Webmin httpd)
  12   │ |_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
  13   │ |_http-trane-info: Problem with XML parsing of /evox/about
  14   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  15   │ 
  16   │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  17   │ # Nmap done at Wed Feb  1 17:04:26 2023 -- 1 IP address (1 host up) scanned in 51.36 seconds
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Hay 2 puertos abiertos, el cual de primeras pase a revisar el puerto `10000` como es un servicio HTTP, use la herramienta WhatWeb para ver las tecnologías que está usando el servicio HTTP.

### WhatWeb

```
❯ whatweb https://10.10.211.234:10000/
https://10.10.211.234:10000/ [200 OK] Cookies[redirect,testing], Country[RESERVED][ZZ], HTML5, HTTPServer[MiniServ/1.890], IP[10.10.211.234], PasswordField[pass], Script, Title[Login to Webmin], UncommonHeaders[auth-type,content-security-policy], X-Frame-Options[SAMEORIGIN]
```

### HTTP

Pase a abrir el sitio en la web por el puerto `10000` y de primeras salio lo siguiente:

![](/assets/images/THM/source/2023-02-08_00-19.png)

Entonces pase a abrir el servicio con HTTPS.

![](/assets/images/THM/source/2023-02-08_00-21.png)

Y vemos un panel de inicio de sesion de WebMin.

![](/assets/images/THM/source/2023-02-08_00-22.png)

En este caso, como en el reporte del escaneo, vi la versión del WebMin corriendo entonces lo que hice fue buscar una vulnerabilidad para esa versión de WebMin y encontre un texto que habla acerca la vuln, en su sitio oficial

![](/assets/images/THM/source/2023-02-08_01-18.png)

Revise ese archivo `password_change.cgi` a ver que y me salio lo siguiente.

![](/assets/images/THM/source/2023-02-08_01-19.png)

Esta versión de WebMin en ejecución tiene una puerta trasera, el exploit está en Metasploit y listo para ser explotado. Pero lo yo lo intenté de forma manual, analizando la peticion por BurpSuite.

### BurpSuite

Abri el Burp


