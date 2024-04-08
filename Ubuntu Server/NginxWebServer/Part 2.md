# Part 2 - Security
( Part 2 - Security )
[link](https://www.youtube.com/watch?v=1BlUS5TzSzk&list=PL7QFPg5Pk4_WEJ7jQ8Mfky7Gslg0oL7sR&index=2).

## Objetivo
Configurar el certificado `ssl` para nuestra pagina.

## Seguridad en Nginx
- Restringir acceso a una pagina por medio de bloqueo de *IP*.
- Proteger una pagina con *user* y *password*.
- Certificado *SSL* para encriptar el trafico de nuestro sitio.
	Aquí vamos a usar un certificado *self-sign certificate*.
	
## Bloqueo por IP

Editar archivo de configuración.
`/etc/nginx/conf.d/test.local.conf`

Agregar la siguiente ruta:
```
location /secure {
			try_files $uri /secure.html;
			deny all;
	}
```
Por ahorita vamos a negar todo trafico a esta pagina.

*Probar Configuración*
`nginx -t`
*Restart Nginx*
`systemctl restart nginx`

*Visitar*
`http://nginx.local/secure`
![98d42e8b701af6827f52d25163b82ec6.png](:/e9a59c1ebc3942cfb7af8d93ab26f05d)
Vemos que el acceso esta prohibido.
<br>

*Cual es mi IP*
Podemos preguntarle a google cual es [my *ip*](https://www.whatismyip.com), en mi caso ahorita es:
`177.236.61.245`
Este es dinámico y puede cambiar en unas horas o días!

Ahora agregar este *ip* a la configuración en */etc/nginx/conf.d/test.local.conf*.
```
location /secure {
		try_files $uri /secure.html;
		allow 177.236.61.245;
		deny all;
}
```

No me funciona a mi porque mi *VM* esta dentro de mi local network.
Para poder acceder a la pagina y [darle acceso](https://stackoverflow.com/questions/51801772/allowing-only-local-network-access-in-nginx) solo a las maquinas que están dentro de mi red local tenemos que:
```
location /secure {
		try_files $uri /secure.html;
		allow 192.168.0.0/24;
		deny all;
}
```

Si quiero solo restringir el acceso solo a una maquina de mi red, entonces:
```
location /secure {
		try_files $uri /secure.html;
		# allow 192.168.0.0/24;
		allow 192.168.0.14;
		deny all;
}
```
Listo, solo la puedo accesar desde esta maquina!

## Bloqueo con User y Password
Para poder utilizar bloqueo de pagina por *user y password* necesitamos instalar unas utilerias:
`sudo apt-get install -y apache2-utils`

Para crear un password para *admin* y guardarlo:
`htpasswd -c /etc/nginx/passwords admin`

asignar otro usuario *test* le voy a dar la misma password la de siempre:
`htpasswd /etc/nginx/passwords test`

Para poder proteger la pagina y tener solo acceso con estos dos users que acabamos de dar de alta, modificar nuestro archivo de configuración.
```
location /secure {
		try_files $uri /secure.html;
		# Auntenticacion
		auth_basic "Se requiere usuario y contraseña ...";
		auth_basic_user_file /etc/nginx/passwords;
		# allow 192.168.0.0/24;
		allow 192.168.0.14;
		deny all;
}
```

*Probar Configuración*
`nginx -t`
*Restart Nginx*
`systemctl restart nginx`

*Al visitar la pagina*
![3fd54a5f28a26f446a048032f20e6021.png](:/0f9268cf050747f88fe0dde37fb3fb58)
<br>
Solo podemos ingresar a ella con user y password, los que dimos de alta.

## Certificado *SSL*
Para poder generar un archivo ssl necesitamos instalar:
`sudo apt-get install -y openssl`

Para generar un certificado, colocarnos en `/etc/nginx` 
Dar el siguiente comando:
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/nginx.key -out /etc/nginx/nginx.cert`

Se crearon los archivos en `/etc/nginx` :
- nginx.key
- nginx.cert

Ahora en el archivo de configuración:
```
server{ 
        listen 80;
        return 301 https://$server_addr$request_uri;    
}

server{

        listen 443 ssl default_server;
        ssl_certificate /etc/nginx/nginx.cert;
        ssl_certificate_key /etc/nginx/nginx.key;
        index index.html index.html index.php;
        server_name test.local;
        root /var/www/test.local;

        location / {
                try_files $uri $uri/ $uri.html =400;
        }

        location /foss {
                try_files $uri /foss.html;
        }

        location /secure {
                try_files $uri /secure.html;
                # Auntenticacion
                # auth_basic "Se requiere usuario y contraseña ...";
                # auth_basic_user_file /etc/nginx/passwords;
                # allow 192.168.0.0/24;
                allow 192.168.0.14;
                deny all;
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

*Agregar regla al ufw*
`ufw allow 443`

*Probar Configuración*
`nginx -t`
*Restart Nginx*
`systemctl restart nginx`

*Visitar*
`https://nginx.local/`
![42bb6d8e855c45b90291821283554321.png](:/a14ad0f096154d6abbc4a63abf9f55d7)
Acept risk and continue.


## IP especifico
Para usar un ip especifico por donde va a estar sirviendo esta pagina:
```
server{
      listen 80 default_server;
      server_name 192.168.0.6;
      return 301 https://$server_name$request_uri;    
}
```

También puede ser así:
```
server{
      listen 80 default_server;
      server_name "nginx.local";
      return 301 https://$server_name$request_uri;    
}
```


* * *
Listo!



