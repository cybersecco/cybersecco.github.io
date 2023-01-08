---
layout: single
title: "TryHackMe ChillHack Writeup"
excerpt: "En este writeup nos enfrentamos a un sitio web el cual tiene ejecucion remota de comandos y aprovechandonos de eso, obtenemos acceso al servidor donde esta alojado el sitio web."
date: 2023-01-08
classes: wide
header:
 teaser: /assets/images/chillHack/chillHack.png
 teaser_home_page: true
categories:
  - CTF
tags:
  - security
  - hacking web
  - command injection
  - sql injection
---



## Introduccion

Este writeup detalla el proceso que se llevo a cabo para hackear la maquina objetivo llamada "ChillHack" de TryHackMe. Tiene un sitio web el cual tiene diferentes vulnerabilidades que se pueden explotar para comprometer el sistema. Utilizando diferentes herramientas y tecnicas de hacking incluyendo `nmap, gobuster, whatweb, netcat`.

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

