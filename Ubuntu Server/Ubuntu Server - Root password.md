# Ubuntu Server - Root password

## Objetivo
Crear un password para "root".
[link](https://logmeonce.com/resources/how-to-set-root-password-in-ubuntu/).

## Como crear contraseña
Como crear contraseña para el user root.

Entrar al server:
`ssh enrique@192.168.0.9`

Open Terminal – First, open the Terminal application via the Ubuntu Dash. Type:
`sudo passwd root`

Enter the Desired Password – You will be prompted to enter your desired root password, which will be saved to the system. Re-enter it to confirm the password.

Test the Root Password – After the password is entered, you can test it out by typing in “sudo su”, and then entering the password.

Salir del user enrique con:
`exit`

Y volver a entrar al server, pero ahora con root:
`ssh root@192.168.0.9`
dar la clave de acceso: *passworddeserver*

Esto lo hice para poder hacer login con root y poder usar remote ssh con vs code.
Y así poder editar cualquier archivo.

## How to set up passwordless SSH access for root user
[link](https://askubuntu.com/questions/115151/how-to-set-up-passwordless-ssh-access-for-root-user).

*On the client (where you ssh FROM)*
First make a ssh key with no password. I highly suggest you give it a name rather then using the default
`ssh-keygen -f foo`
Yo ya crie estas llaves, ver *Ubuntu Server - SSH Keys*.

*On the server (where you ssh TO)*
edit /etc/ssh/sshd_config
`sudo nano /etc/ssh/sshd_config`
Make sure you allow root to log in with the following syntax
```
PasswordAuthentication yes
PermitRootLogin yes
```
Restart the server
`sudo service ssh restart`

Set a root password, use a strong one
`sudo passwd`
Ya lo hice desde el user enrique al inicio de este tema.

*On the client :*
From the client, Transfer the key to the server
ssh-copy-id -i ~/.ssh/foo root@server
change "foo" the the name of your key and enter your server root password when asked.

Yo lo hice con:
`ssh-copy-id -i ~/.ssh/enrique-latitude enrique@192.168.0.9`
Me va a pedir el password.

*Test the key*
`ssh root@192.168.0.9`

Assuming it works, unset a root password and disable password login.
Ya podemos des habilitar el login con password en:
`sudo nano /etc/ssh/sshd_config`

*On the server :*
`sudo passwd -l root`

Edit /etc/ssh/sshd_config
`sudo nano /etc/ssh/sshd_config`

Change the following :
```
PasswordAuthentication no
PermitRootLogin without-password
```

Restart the server
`sudo service ssh restart`

* * *
Listo!
