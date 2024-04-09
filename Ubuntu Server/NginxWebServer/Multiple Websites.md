# Multiple Websites

## Objetivo
How to host multiple websites on Nginx.
[link](https://www.youtube.com/watch?v=GCznXfbfMq0).

## Paso 1 - Crear Sitios
Crear sitios web en:
`/var/www`
Para este ejemplo:
*Test 1*
`/var/www/test1.local/`
Y copiar dentro de la carpeta el sitio web.
`/var/www/test1.local/index.html`
index.html
```
<!DOCTYPE html>
<html>
<style>
div{
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
}
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;

}
</style>
<body style="background-color:white">
<div>
<h1 style="font-family:verdana" style="color:black">Bienvenido test1.local</h1>
<a href="/foss" h2 style="font-family:verdana" style="color:black">click for More >
<h3 style="font-family:verdana">This is a simple Welcome page, powered by </h3>
</div>

<img src=https://www.nginx.com/wp-content/uploads/2018/08/NGINX-logo-rgb-large.png>


</body>
</html>
```
<br>
*Test 2*
`/var/www/test1.local/`
Y copiar dentro de la carpeta el sitio web.
`/var/www/test1.local/index.html`
index.html
<br>

## Crear Sites Available
Crear sitios disponibles en;
`/etc/nginx/sites-available`
Con los siguientes archivos de configuracion!

*Para el sitio 1:*
`/etc/nginx/sites-available/test1.local`
*test1.local*
```
server {
    listen 80;
    server_name test1.local;

    location / {
        root /var/www/test1.local;
        index index.html;
    }
}
```

*Para el sitio 2:*
`/etc/nginx/sites-available/test2.local`
*test2.local*
```
server {
    listen 80;
    server_name test2.local;

    location / {
        root /var/www/test2.local;
        index index.html;
    }
}
```

## Crear Simbolic links
Crear simbolic links.
`cd /etc/nginx/sites-available/`

*Para Sitio 1*
`sudo ln -s /etc/nginx/sites-available/test1.local /etc/nginx/sites-enable/`

*Para Sitio 2*
`sudo ln -s /etc/nginx/sites-available/test2.local /etc/nginx/sites-enable/`

*Probar Configuración*
`nginx -t`
*Restart Nginx*
`systemctl restart nginx`

## Crear DNS 
Crear los DNS en la maquina local.
`sudo micro /etc/hosts`
Entrar:
```
127.0.0.1 localhost
127.0.0.1 mxLatitude

192.168.0.9 optiplex755 optiplex755.local

192.168.0.6 nginx nginx.local
192.168.0.6 test1 test1.local
192.168.0.6 test2 test2.local


# The following lines are desirable for IPv6 capable hosts
::1		localhost ip6-localhost ip6-loopback
fe00::0		ip6-localnet
ff00::0		ip6-mcastprefix
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters
```

## Visitar
Para visitar la pagina, abrir navegador y visitar:

*Para sitio 1*
`http://test1.local`
![c8d1faa5d1cb4f53ce99334553b0116a.png](:/486dee010a754eff81ab88f4ee794538)

*Para sitio 2*
`http://test2.local`
![f27cd1c443e2c488cfd9af577e9e2f84.png](:/648f2d18d0294e34a08ab86574532feb)


* * *
Listo!


