---
layout: single
title: "ChocolateFactory - Try Hack Me"
excerpt: "En este desafío, tendrás la oportunidad de poner en práctica tus habilidades de hacking mientras exploras y explotas un sitio web. Comenzarás realizando tareas de reconocimiento y enumeración, seguido de la utilización de técnicas de esteganografía para obtener información oculta en una imagen. Además, tendrás que crackear hashes y obtener shells para continuar con la escalada de privilegios, abusando de un binario específico. ¡Prepárate para un desafío emocionante y una gran experiencia de aprendizaje!"
date: 2023-01-11
classes: wide
header:
 teaser: /assets/images/chocolateFactory/e2eed78e92b4890174e0a2510b6e7a7c.jpeg
 teaser_home_page: true
 icon: /assets/images/thm.webp
categories:
  - CTF
  - Free
tags:
  - hacking web
  - security
  - esteganografía
---

![](/assets/images/chocolateFactory/e2eed78e92b4890174e0a2510b6e7a7c.jpeg)

## Introduccion

En este desafío, tendrás la oportunidad de poner en práctica tus habilidades de hacking mientras exploras y explotas un sitio web. Comenzarás realizando tareas de reconocimiento y enumeración, seguido de la utilización de técnicas de esteganografía para obtener información oculta en una imagen. Además, tendrás que crackear hashes y obtener shells para continuar con la escalada de privilegios, abusando de un binario específico. ¡Prepárate para un desafío emocionante y una gran experiencia de aprendizaje!

## Escaneo

Inicialmente siempre empiezo lanzando un ping a la maquina objetivo con el fin de descartar cierta información, como lo es, primero si esta encendida la maquina y luego reconocer el sistema que esta corriendo, es decir si es Windows o Linux.

Si es Linux debe tener un TTL (Time to live) aproximado o igual a 64 y si es un sistema Windows debe tener un TTL aproximado o igual a 128.

```

```