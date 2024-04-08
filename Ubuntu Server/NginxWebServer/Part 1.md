
## Part 1 - NGINX Web Server ( Installation & Configuration )
https://www.youtube.com/watch?v=RzrIqzC9bQc


# NGINX Web Server
( Installation & Configuration )
[link](https://www.youtube.com/watch?v=RzrIqzC9bQc&list=PL7QFPg5Pk4_WEJ7jQ8Mfky7Gslg0oL7sR).

## Objetivo
Instalar y Configurar Nginx en ubuntu 22.04 live server.

## Instalar Virtual box con Ubuntu Server
Instalar *ubuntu-22.04.4-live-server-amd64.iso*.
Ya que se instalo, configurar la VM en modo:
`network bridge`

## Instalar Nginx
`sudo apt install nginx`

## Probar 
Probar que este sirviendo la pagina de prueba en puerto 80.
`curl localhost`
Nos regresa el html de:
`/var/www/html/index.nginx-debian.html`
Con eso comprobamos que nginx esta trabajando, para poder ver esta pagina de prueba desde en navegador, desde el host abrimos un navegador y entramos el ip de nuestra VM.
`http://192.168.1.67`
Vemos la pagina de entrada de Nginx!

## Sudo
Cambiarse a sudo user:
`sudo su`

## Archivos de Configuración Nginx
`cd /etc/nginx`

default config:
`cat nginx.conf`

por default la pagina de prueba:
`cd /etc/nginx/sites-enabled`
Listamos:
`ls -lt`
*lrwxrwxrwx 1 root root 34 Apr  7 14:17 default -> /etc/nginx/sites-available/default*

Vamos a eliminar este simlink.
`unlink default`

Ahora vamos a crear nuestro propio archivo de configuracion en:
`cd /etc/nginx/conf.d`

Por ejemplo crear una pagina de prueba que se llame test.local:
`micro test.local.conf`

Insertar Código:
```
server{
        listen 80 default_server;
        index index.html index.html index.php;
        server_name test.local;
        root /var/www/test.local;
}
```

## Crear archivo index.html
`/var/www/test.local/index.html`
con un sencillo mensaje:
`<h1>Esta en la pagina de /var/www/test.local/index.html</h1>`

## Reload Nginx
`systemctl restart nginx`

## Test Nginx
`nginx -t`

## Visitar Pagina
Visitar pagina desde host, abrir navegador y visitar:
`http://192.168.0.6`

Si tenemos configurado en el host el archivo `/etc/hosts` podemos visitar:
`http://nginx.local`

## Clonar de GitHub
*https://github.com/techbeast-org/nginx-basics*
En VM ir a:
`/home/enrique`
Clonar de github la pagina de prueba:
`git clone https://github.com/techbeast-org/nginx-basics`

Ahora mover los archivos de nginx-basics a la carpeta */var/www/test.local/*.
`mv -v /home/enrique/nginx-basics/* /var/www/test.local/`
también los puedo mover con mc.

Reload:
`http://nginx.local`
Listo, ya vemos la pagina.

ahora para poder ver:
`http://nginx.local/foss`

*Configurar*
`/etc/nginx/conf.d/test.local.conf`
Código:
```
server{

        listen 80 default_server;
        index index.html index.html index.php;
        server_name test.local;
        root /var/www/test.local;

        location / {
                try_files $uri $uri/ $uri.html =400;
        }

        location /foss {
                try_files $uri /foss.html;
        }
}
```

*Probar Configuración*
`nginx -t`

*Restart Nginx*
`systemctl restart nginx`

## Probar 400 error - Pagina no existe
*Ahora Probar Pagina*
`http://nginx.local/foss`
Listo, ya vemos la pagina.
![b1da06fb6a39fa5e0da648857f6f1129.png](:/25f068ee6c7646828e4c510ed6e33e0a)
<br>

*AL tratar una pagina que no existe*
`http://nginx.local/mipagina`
![f4319d0994846115ac06f8d1ce95d540.png](:/ece8586fdba54f21981f37aca5d00eff)
<br>

*Para una pagina de error custom*
`En /etc/nginx/conf.d/test.local.conf`
Agregamos:
```
server{
        ...
        error_page 400 404 /400.html;
        location = /400.html {
                internal;
        }
}
```

*Probar Configuración*
`nginx -t`
*Restart Nginx*
`systemctl restart nginx`

*Probar en el navegador*
`http://nginx.local/mipagina`
![cb7e5121f4a15ed10c0bb38b1af0fc37.png](:/d65a2390542c46bfb2eb2d85c775e9c0)
<br>

## Probar 500 error - Internal Server Error

*Para simular un error interno de server agregar*
`En /etc/nginx/conf.d/test.local.conf`
Agregamos:
```
server{
        ...
        location /500error {
                fastcgi_pass unix:/this/is/error;
        }
}
```

*Todo el código quedaría así*
`En /etc/nginx/conf.d/test.local.conf`
Agregamos:
```
server{

        listen 80 default_server;
        index index.html index.html index.php;
        server_name test.local;
        root /var/www/test.local;

        location / {
                try_files $uri $uri/ $uri.html =400;
        }

        location /foss {
                try_files $uri /foss.html;
        }

        location /500error {
                fastcgi_pass unix:/this/is/error;
        }

        error_page 400 404 /400.html;
        location = /400.html {
                internal;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                internal;
        }
}
```


*Probar Configuración*
`nginx -t`
*Restart Nginx*
`systemctl restart nginx`

*Probar en el navegador*
`http://nginx.local/500error`
![1bb6ad26c2cd89dc347649d9d9c13477.png](:/1b67dd90ee6d483c9ea1d0b0dd7829ef)
<br>


* * *
Listo!




