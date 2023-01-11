---
layout: single
title: "Health - Hack The Box"
excerpt: "Health es una máquina Linux de calificación media donde se utilizó un ataque SSRF en una máquina Linux con un nivel medio de seguridad para obtener acceso al sistema, mediante el uso de técnicas de inyección de URL y escalada de privilegios. Se obtuvo el hash de la contraseña y nombre de usuario, luego se descifró y se utilizó para iniciar sesión en el sistema. Finalmente, se escaló a acceso de root utilizando el control completo sobre la base de datos y se obtuvo la clave SSH."
date: 2023-01-11
classes: wide
header:
 teaser: /assets/images/health/health.png
 teaser_home_page: true
 icon: /assets/images/hackthebox.webp
categories:
  - HTB
  - Free 
tags:
  - web enumeration
  - abusing webhook setup
  - sql injection
  - ssrf
---

![](/assets/images/health/fullHealth.png)

## Introducción

Health es una máquina Linux de calificación media donde se utilizó un ataque SSRF en una máquina Linux con un nivel medio de seguridad para obtener acceso al sistema, mediante el uso de técnicas de inyección de URL y escalada de privilegios. Se obtuvo el hash de la contraseña y nombre de usuario, luego se descifró y se utilizó para iniciar sesión en el sistema. Finalmente, se escaló a acceso de root utilizando el control completo sobre la base de datos y se obtuvo la clave SSH.

## Escaneo
