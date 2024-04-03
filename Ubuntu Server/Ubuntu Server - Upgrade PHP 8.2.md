# Ubuntu Server - Upgrade PHP 8.2

## Upgrade to PHP 8.2
[link](https://codewithsusan.com/notes/upgrade-php-nginx).

In this guide I’ll walk through the process of upgrading PHP on an Ubuntu server running Nginx. For my example, I’ll do a PHP 8.0 → 8.2 upgrade, but you should be able to apply the same general principles to other version jumps.

In the notes that follow, when you see reference to php8.x swap out x with the relevant version number for the upgrade you’re doing.

## Output phpinfo
Before you begin the upgrade, create a PHP file on your server that you can access in the browser that invokes PHP’s *info.php* function:
En mi caso por ahora tengo en mi server php8.1.

*Crear el archivo:*
`/var/www/demo/info.php`

Código:
```
<?php
phpinfo();
```

Para poder servir esta pagina *php* tenemos que preparar a *nginx*.
`En /etc/nginx/sites-available`
Crear el archivo:
`demo`

con el siguiente código:
*server_name* 
should point to the domain (or subdomain) you’re using for this application.

*root*
should point to the path of your application directory.

*The reference*
to php8.1-fpm.sock should match whatever version of PHP your server is running

```
server {

    # Port 80 is used for incoming http requests
    listen 8081;

    # The URL we want this server config to apply to
    server_name optiplex755;

    # The document root of our site - i.e. where its files are located
    root /var/www/demo;

    # Which files to look for by default when accessing directories of our site
    index index.php index.html;

    # All requests that end with .php will be handled by the server’s PHP install
    # If you’re not using PHP you can omit this block
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
Lo mando al puerto *8081* porque en el *80* ya tenemos la pagina de *default* de *nginx*.

*/etc/nginx/sites-enabled* 
Next, to enable this config we need to symbolically link the file to the /etc/nginx/sites-enabled directory.
To do this, run the following command, replacing demo with the name of the file you created:
`sudo ln -s /etc/nginx/sites-available/demo /etc/nginx/sites-enabled`

*With that complete*
Run the command sudo nginx -t to check your configs making sure there are no issues:
`sudo nginx -t`

*Expected output*
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If all looks good, restart Nginx to make the changes take effect:
`systemctl restart nginx`

*Visitar Pagina*
`http://optiplex755:8081`

*Captura de Pantalla*
![3473b9ff76ee38d59dbbbc45af401ff5.png](:/a46c3b4a256142f2ac9f92215cbe63ee)

## Record list of current PHP related packages
In addition to updating PHP, you will need to update any PHP related packages your server is using. To help with this, run the following command which will create a text file called php-packages.txt that will contain a list of all your currently installed PHP packages.
`sudo dpkg -l | awk '/^ii/{print $2}' | grep php | tee php-packages.txt`

*Mi salida*
```
enrique@optiplex755:~$ sudo dpkg -l | awk '/^ii/{print $2}' | grep php | tee php-packages.txt
[sudo] password for enrique: 
php-common
php-fpm
php-mysql
php8.1
php8.1-cli
php8.1-common
php8.1-curl
php8.1-fpm
php8.1-igbinary
php8.1-imap
php8.1-mbstring
php8.1-mysql
php8.1-opcache
php8.1-readline
php8.1-redis
php8.1-snmp
php8.1-xml
php8.1-zip
```

## Add repository
Next, we’ll use the Linux package manager apt to target a repository called ondrej/php PPA which contains all the PHP software we’ll need:
`sudo add-apt-repository ppa:ondrej/php`
Actualizar:
`sudo apt update`

## Install PHP and packages
Now we’re ready to install PHP and its related packages referencing the text file you created in the above step. Here’s a template for a command you can use to do this:
`sudo apt install php8.x {list packages here...}`
Note that most packages follow the pattern of php8.x-packagename, e.g. php8.2-mysql.
As an example, based on my above packages list and an update to PHP 8.2 my install command would look like this:

*Para mi Server:*
`sudo apt install php-common php-fpm php-mysql php8.2 php8.2-cli php8.2-common php8.2-curl php8.2-fpm php8.2-igbinary php8.2-imap php8.2-mbstring php8.2-mysql php8.2-opcache php8.2-readline php8.2-redis php8.2-snmp php8.2-xml php8.2-zip`

*Version*
`php -v`
	PHP 8.3.4 (cli) (built: Mar 16 2024 08:40:08) (NTS)
	Copyright (c) The PHP Group
	Zend Engine v4.3.4, Copyright (c) Zend Technologies
		with Zend OPcache v8.3.4, Copyright (c), by Zend Technologies

No se porque me actualizo a la *8.3*

## Update site’s PHP handler
Next you need to update your Nginx site configs (located in /etc/nginx/sites-enabled) to use your updated PHP handler. To do this, edit all of your site configs so that fastcgi_pass points to the appropriate php-fpm version. For example:
`En /etc/nginx/sites-available/demo`
Código:
```
server {

    # Port 80 is used for incoming http requests
    listen 8081;

    # The URL we want this server config to apply to
    server_name optiplex755;

    # The document root of our site - i.e. where its files are located
    root /var/www/demo;

    # Which files to look for by default when accessing directories of our site
    index index.php index.html;

    # All requests that end with .php will be handled by the server’s PHP install
    # If you’re not using PHP you can omit this block
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Then restart Nginx:
`sudo service nginx restart`

## Confirm it worked
Refresh your phpinfo page and confirm you’re seeing the updated PHP version number:

*Captura de Pantalla*
![3bd908f8bbac42ee18a8bf439ddf150c.png](:/3a0010e07a5d4f019f9514752e346439)


## Sync changes to php.ini
Your final step is to make sure any configurations you might have made to PHP in your previous install are reflected in your upgraded install. To do this you need to first locate the path to your new/current php.ini file from the Loaded Configuration File value in your phpinfo output:

*Example path:*
`/etc/php/8.3/fpm/php.ini`

From this path we can infer the path to your previous php.ini file by changing the version number:
`/etc/php/8.1/fpm/php.ini`

Next, you can plug these paths into a helper script I’ve created that will tell you the setting differences between these two files. To do this. create a PHP file on your server that you can access in the browser called ini-diff.php that contains [this code](https://gist.github.com/susanBuck/17d9c88ab845bbb6f088e48d1cd56435).
Within the code, update the first two variables to reflect your current and previous php.ini paths. E.g.:
```
# UPDATE THESE TWO PATHS:
$current = '/etc/php/8.2/fpm/php.ini';
$previous = '/etc/php/8.0/fpm/php.ini';
```

When you run this file, it will outline any setting differences in your previous and current ini file. E.g.:

Use this information to update your current php.ini file to reflect any settings you wish to persist in your upgraded PHP.
When you’re done, make the changes take effect by reloading your PHP handler:
`systemctl reload php8.2-fpm`
You can reload your phpinfo page and search for one of the settings you changed to confirm the changes took affect.

## Clean up
When you’re done you can (but don’t have to) purge your old unused PHP packages using the following command (replace 8.x as appropriate)
`sudo apt purge php8.x*`

* * *
Listo!

