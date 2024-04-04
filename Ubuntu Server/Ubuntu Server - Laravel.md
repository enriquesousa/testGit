# Laravel
Ubuntu Server - Laravel
[link](https://codewithsusan.com/notes/deploy-laravel-on-nginx).

## Objetivo
Instalar Aplicación de Laravel en Ubuntu Server.
*Mi server:* optiplex755

Get the code base
Now that our server is prepped to run Laravel, we can get a copy of our application on the server by cloning it from Github. In my example, I will clone it to my web directory at:
`/var/www/`

## Laravel PHP extension requirements
 ✅ Ctype PHP Extension
✅ cURL PHP Extension
❌ DOM PHP Extension
✅ Fileinfo PHP Extension
✅ Filter PHP Extension
✅ Hash PHP Extension
✅ Mbstring PHP Extension
✅ OpenSSL PHP Extension
✅ PCRE PHP Extension
✅ PDO PHP Extension
✅ Session PHP Extension
✅ Tokenizer PHP Extension
❌ XML PHP Extension

`sudo apt install zip unzip` 

## Clonar Código Demo2 de Mi GitHub
Es un Laravel 11 Boiler Plate sin Base de datos.
```
cd /var/www/
sudo git clone git@github.com:enriquesousa/demo2.git
```

No lo pude clonar directo en /var/www/.
Lo que hice fue clonar en /home/enrique
`cd /home/enrique/`
clonar:
`git clone git@github.com:enriquesousa/demo2.git`
Lo clono bien!
Ahora moverlo a /var/www/
`sudo mv /home/enrique/demo2/ /var/www/`
Ya lo movio!
Pero tiene el user enrique:enrique

Once this is done, I can move into the resulting directory:
`cd /var/www/demo2`

## Get dependencies
Next, we need to pull in the application’s Composer dependencies (i.e. our vendor/ directory).
If working on a production server, use the following command so any development-specific dependencies are excluded and the version number of your dependencies match whatever was used in development and written to composer.lock
`composer install --optimize-autoloader --no-dev`

If working on a development server, use the following command so all dependencies are included and you get the latest versions within your version constraints listed in your composer.json file:
`composer update`

Yo lo voy hacer como producción:
`composer install --optimize-autoloader --no-dev`

Copiar el .env.example a .env
`cp .env.example .env`

Generar Key:
`php artisan key:generate`

Archivo .env
```
APP_NAME=Demo2
APP_ENV=production
APP_KEY=base64:zTnLjJtXnOmvEKUP8a/z+YbHyKY+We4h2z21qILGjmA=
APP_DEBUG=true
APP_TIMEZONE=UTC
APP_URL=http://optiplex755.local:8085

APP_LOCALE=en
APP_FALLBACK_LOCALE=en
APP_FAKER_LOCALE=en_US

APP_MAINTENANCE_DRIVER=file
APP_MAINTENANCE_STORE=database

BCRYPT_ROUNDS=12

LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=sqlite
...
```

## Set permissions
There are two directories within a Laravel application that need to be writable by the server: storage and bootstrap/cache. Within these directories, the server will write application-specific files such as cache info, session data, error logs, etc.
To allow this to happen, you need to update the permissions of storage and bootstrap/cache so they are owned by the system user your web server is running as.
Run the following command to see which user your Nginx web server runs as:
`ps aux | grep "nginx: worker process" | awk '{print $1}' | grep -v root`
*www-data
www-data*

On my server, the above command outputs the user www-data so I will update storage and bootstrap/cache to be owned by www-data with the following two commands:
`chown -R www-data storage`
Y
`chown -R www-data bootstrap/cache`

## Configure site
At this point, everything is set up within our application, we just need to configure our server to run it.

Within `/etc/nginx/sites-available/` create a new file with the following content (ref: Nginx deployment). You can call the file whatever you want; I’ll name mine demo after the name of the app.

Within the content update the following:

`server_name` should point to the domain (or subdomain) you’re using for this application
`root` should point to the path of your Laravel application’s public directory
The reference to *php8.3-fpm.sock* should match whatever version of PHP your server is running.

Crear archivo:
`/etc/nginx/sites-available/demo2`
Y copiar este código:
```
server {
    listen 8085;
    listen [::]:8085;
    server_name optiplex755;
    root /var/www/demo2/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Next, to enable this config we need to symbolically link the file to the /etc/nginx/sites-enabled directory.
To do this, run the following command, replacing demo with the name of the file you created:
`sudo ln -s /etc/nginx/sites-available/demo2 /etc/nginx/sites-enabled`

With that complete, run the command sudo nginx -t to check your configs making sure there are no issues:
`sudo nginx -t`

Expected output:
*nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful*

If all looks good, restart Nginx to make the changes take effect:
`systemctl restart nginx`



## Problemas
No podía escribir a base de datos.
[link](https://stackoverflow.com/questions/47835714/laravel-sqlite-sqlstatehy000general-error-8-attempt-to-write-a-readonly-d)

Estos pasos me funcionaron:
`sudo chown -R :www-data /var/www/demo2`
Y
`sudo chmod -R 775 /var/www/demo2/storage`

`sudo chmod -R 775 database`
`sudo chown -R $(whoami) database`

Ya pude visitar:
`http://optiplex755.local:8085`

## Captura de Pantalla
![5aee7d1e714890ed5132465484eb1acc.png](:/b5952fc1026343ba8faf54858f88a02b)


* * *
Listo!