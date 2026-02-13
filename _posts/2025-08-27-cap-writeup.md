---
layout: post
title: "Cap Writeup"
date: 2025-08-27
categories: [HackTheBox, Linux]
image: "/assets/images/cap.png"
description: "Resolviendo la máquina Cap, Easy..."
---

Primero escaneé la IP, en mi caso 10.10.10.245, luego de esto ví que estaban los puertos 21, 22, 80, 8000 abiertos,![](/assets/images/writeups/Pasted image 20250823165147.png)

- Vemos que el puerto 21 ftp está abierto, intentamos entrar con el login anonymous
 
 ![](/assets/images/writeups/Pasted image 20250823165907.png)
Como podemos observar, no está activo el login a ftp mediante anonymous, entonces intentamos entrar a la página http://10.10.10.245/ y nos sale lo siguiente: ![[Pasted image 20250823170238.png]]

Entramos al apartado Security Snapshot, ahí en el enlace podemos ver que entra a /data/6, dónde el 6 es un id, si retrocedemos, y cambiamos el id a 0, y descargamos el archivo, al ver que es un archivo .pcap lo analizamos con wireshark, vemos que nos ofrece información interesante:
![](/assets/images/writeups/Pasted image 20250823172129.png)

y vemos que en el protocolo FTP nos está arrojando el user y la contraseña, intentamos logear mediante ftp con este usuario y esta contraseña:
![](/assets/images/writeups/Pasted image 20250823172309.png)
Fué un éxito, logramos entrar a la máquina del usuario nathan mediante el protocolo ftp, dónde en este se encuentra el archivo user.txt y está la flag de usuario.

Luego de esto me dí cuenta de que por ssh era mejor, entonces entré por ssh a la máquina con las mismas credenciales

para obtener privilegios root, me fuí al path /usr/bin/python3.8 y mediante el interprete de comandos de python agregamos estas líneas de comandos:

>[!Code] 
import os
os.setuid(0)
os.system("/bin/bash")

Con esto obtenemos los privilegios root, nos vamos al directorio user y ahí se encuentra la root flag

![](/assets/images/writeups/Pasted image 20250823175546.png)

Y eso es todo por la máquina CAP de Hack The Box.
