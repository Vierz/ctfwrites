# CTF: RickdiculouslyEasy



### Vulnhub: https://www.vulnhub.com/entry/rickdiculouslyeasy-1,207/
**Total de puntos:** 130

----------------

Descargue la vm y luego de correrla en VirtualBox, lo primero que hice fue hacer un escaneo de todos los puertos y las versiones de sus servicios:
```console
root@localhost$: nmap -sV 192.168.56.106 -p- 
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

En esa pagina hay un login pero no encontre la manera de romperlo asi que segui al proximo puerto, el 80.

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



