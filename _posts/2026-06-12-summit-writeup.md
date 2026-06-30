---
layout: post 
title: "THM: Summit" 
date: 2026-06-12 
categories: [Blue Team, SOC, Linux]
image: /assets/images/summit-1.png
description: "Resolviendo la Room Summit de TryHackMe..."
---
## Objetivo

En este laboratorio, PicoSecure plantea una simulación de amenazas enfocado en las capacidades de detección de malware. Para ello se trabaja en un entorno purple team, donde un pentester externo intenta ejecutar muestras maliciosas 
mientras se configuran mecanismos de detección y prevención.

El objetivo es incrementar el costo operativo del atacante siguiendo la Pirámide del dolor (Pyramid of Pain), así priorizando indicadores cada vez más difíciles de evadir.

-------------

Una vez iniciada la máquina, nos dirigimos al enlace proporcionado  en la room de tryhackme: https://10-64-166-193.reverse-proxy.cell-prod-us-east-1a.vm.tryhackme.com/

Una vez dentro, se puede observar  la página de PicoSecure:

![](/assets/images/writeupsPasted image 20260506125445.png)

---------------
## Primera flag

La primera pregunta del laboratorio dice: `What is the first flag you receive after successfully detecting **sample1.exe**?`

Para resolver esta primera pregunta, se analiza el archivo `sample1.exe` dentro de la plataforma, en el apartado Malware scan. Después de haber escaneado satisfactoriamente el archivo, se obtienen múltiples hashes (MD5, SHA1 y SHA256). Se utiliza el hash SHA256 para identificar la muestra, lo que permite su correcta detección.

Tras este proceso, el sistema envía un correo con la primera flag, solo tenemos que entrar en el apartado `inbox` para poder ver la misma: 

![](/assets/images/writeups/Pasted image 20260506130203.png)

---------------
## Segunda flag

La segunda pregunta es: `What is the second flag you receive after successfully detecting **sample2.exe**?`

Para ello analizamos el segundo archivo llamado `sample2.exe` tal y como indica la pregunta. Luego de haberlo analizado, el mismo nos da más información que el anterior, así dándonos lo uqe parece ser Network Activity y Conexiones en  general, en la cuál vemos un HTTP requests del proceso sample2.exe con PID 1927 por el método GET a la IP `154.35.10.113:4444` y la URL `http://154.35.10.113:4444/uvLk8YI32`

Para esta tarea lo que haremos es bloquear cualquier salida que el destino sea la IP `154.35.10.113`. Tal que quede de la siguiente forma:

![](/assets/images/writeups/Pasted image 20260506131346.png)

al revisar en `inbox` podemos obtener la flag de esta segunda pregunta

![](/assets/images/writeups/Pasted image 20260506131449.png)

-----------
## Tercera flag

La tercera pregunta: `What is the third flag you receive after successfully detecting sample3.exe?`

Procedemos a analizar nuevamente el malware llamado `sample3.exe` que una vez analizado nos da mucha más información que la anterior, pero existe un archivo con un nombre muy peculiar llamado `backdoor.exe` el cuál se atribuye al Domain `emudyn.bresonicz.info` que al  crear una regla DNS para que no se pueda entrar, podemos conseguir la flag:

![](/assets/images/writeups/Pasted image 20260506131744.png)


Luego de enviado el correo, podemos ver la flag:
![](/assets/images/writeups/Pasted image 20260506132826.png)

------------
## Cuarta flag

Ahora procedemos con la pregunta 4: `What is the fourth flag you receive after successfully detecting sample4.exe?`

El mismo método primero. Analizar el archivo llamado `sample4.exe` Y una vez analizado, podemos ver que existen varios Registros de actividad y modficaciones de eventos (logs) en el mismo, aunque también existe un nuevo dominio con otro nombre, esta vez lo solucionaremos por reglas del registro de eventos, Por ejemplo, Vimos un evento llamado `DisableRealtimeMonitoring` que se ve bastante sospechoso:

![](/assets/images/writeups/Pasted image 20260506133315.png)

Al ver esta creación de Registro, podemos realizar varias cosas:

- Abrir el Sigma Rule Builder de PicoSecure
- ir a Sysmon Event Logs
- Luego dirigirnos a la parte de abajo dónde está ubicada la parte `Registry Modifications`
- Ahí añadir todos los datos que nos piden
![](/assets/images/writeups/Pasted image 20260506133654.png)

Al tratarse  de un intento de Evasión de Microsoft Defender, el `ATT&CK ID` lo ponemos como `Defense Evasion (TA0005)` El cuál nos dimos cuenta que es una evasión del defender debido al registro que se estaba modificando y al mismo tiempo el nombre que le pusieron al registro. Y con ello obtenemos la flag de esta pregunta:

![](/assets/images/writeups/Pasted image 20260506133828.png)

-------------
## Quinta flag

Ahora procedemos con la pregunta: `What is the fifth flag you receive after successfully detecting sample5.exe?`

Al analizar los logs y los HTTP requests de este `sample5.exe` se puede observar como combinan los dos, tenemos un beacon.bat que se ve altamente sospechoso, y la ip `51.102.10.19` conectada al puerto `1382` se ve altamente sospechosa también.

![](/assets/images/writeups/Pasted image 20260506134316.png)

![](/assets/images/writeups/Pasted image 20260506134729.png)

En la captura de los logs podemos ver una similitud en cada log, los bytes son exactamente iguales y siempre salen desde la misma dirección IP hacia la IP que estamos sospechando, es decir: `10.10.15.12` -> `51.102.10.19` Y además aunque no se aprecia en la captura, la conexión se está haciendo cada 30 minutos, que en segundos serían unos 1800 segundos.

Como el atacante se puede conectar desde cualquier otra IP, debemos de asegurar que no se vuelva a conectar para crear un C2, por ello fuimos a `Sigma Rule Builder` nuevamente, en Sysmon Event Logs, y luego en Network Connections pusimos las siguientes características que ya nos encontramos:

![](/assets/images/writeups/Pasted image 20260506135548.png)

Luego de esto recibimos el correo con la flag.

![](/assets/images/writeups/Pasted image 20260506135628.png)

-----------
## Sexta flag

Y para la última pregunta del lab: `What is the final flag you receive from Sphinx?`

Nuevamente escaneamos el sample, en este caso llamado `sample6.exe` En este nos encontramos con unos procesos hechos en CMD y un archivo llamado extiltr8.log: 

![](/assets/images/writeups/Pasted image 20260506135852.png)

Sphinx nos envió un archivo con los logs de commandos llamado `commands.log` que tenía los siguiente comandos ejecutados: 

```powershell
dir c:\ >> %temp%\exfiltr8.log
dir "c:\Documents and Settings" >> %temp%\exfiltr8.log
dir "c:\Program Files\" >> %temp%\exfiltr8.log
dir d:\ >> %temp%\exfiltr8.log
net localgroup administrator >> %temp%\exfiltr8.log
ver >> %temp%\exfiltr8.log
systeminfo >> %temp%\exfiltr8.log
ipconfig /all >> %temp%\exfiltr8.log
netstat -ano >> %temp%\exfiltr8.log
net start >> %temp%\exfiltr8.log
```

Y creamos una regla en el mismo `Sigma Rule Builder`:
![](/assets/images/writeups/Pasted image 20260506140113.png)

Y ya obtuvimos la última flag del laboratorio: 

![](/assets/images/writeups/Pasted image 20260506140150.png)

Este laboratorio permitió practicar la detección de malware utilizando diferentes enfoques, desde indicadores simples como hashes hasta reglas Sigma para eventos de Sysmon. También reforzó conceptos relacionados con la Pirámide del Dolor, demostrando cómo aumentar el costo operativo de un atacante mediante controles cada vez más difíciles de evadir.

Aunque se trata de un entorno simulado, reproduce situaciones comunes que puede encontrar un analista SOC durante tareas de monitoreo y respuesta ante incidentes.
