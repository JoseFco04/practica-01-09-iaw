# Practica-01-09-IAW Jose Francisco León López
##  Para esta práctica vamos a necesitar dos instancias de Amazon: Una va a ser la máquina del frontend y otra la del backend.
## Su configuración en amazon se deberían de ver así:
### La configuración de la máquina del frontend es esta
![1](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/f57015c3-913f-43bf-b48f-5d46b79bca13)
### Y su grupo de seguridad es este:
![Captura de pantalla 2023-12-08 212026](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/e6ae71af-6a57-4993-a7ac-ebed39b15230)

### La configuarción de la máquina del backend es esta :
![1](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/22a730aa-27aa-43a5-ba6d-3ac92c5d45d1)

### Y su grupo de seguridad es este:
![2](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/4decf889-2ad6-42ca-bcde-c9c5bc3f1f16)

### El objetivo de la práctica es automatizar la instalación y configuración de una página web a través de dos máquinas virtuales de ubuntu. Una de ellas será la que controlará la parte del frontend y otra la del backend.

### Para esta práctica debemos separar los scripts de instalación en dos un install_lamp y un deploy para el frontend y otros distintos para el del backend.

### Aparte de estos scripts tenemos que tener el script del lets encrypt que ya hemos usado en prácticas anteriores, un .env con las variables que necesitamos para instalar la aplicación web que debería verse parecido a este:
~~~
# Configuramos las variables
#---------------------------------------------------------
MYSQL_PRIVATE_IP=172.31.95.202

WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=josefco
WORDPRESS_DB_PASSWORD=1234
WORDPRESS_DB_HOST=172.31.95.202
IP_CLIENTE_MYSQL=172.31.82.115

WORDPRESS_TITLE="Sitio web de IAW Jose"
WORDPRESS_ADMIN_USER=admin 
WORDPRESS_ADMIN_PASS=admin 
WORDPRESS_ADMIN_EMAIL=josefco@iaw.com

CB_MAIL=josefco@iaw.com
CB_DOMAIN=practica9frontjose.ddns.net

TEMA=sydney
PLUGIN=bbpress
PLUGIN2=wps-hide-login
~~~
### EL archivo de configuración que también hemos usadlo en prácticas anteriores que es el  000-default.conf que se vería así:
~~~
ServerSignature Off
ServerTokens Prod
<VirtualHost *:80>
  #ServerName www.example.com
  DocumentRoot /var/www/html
  DirectoryIndex index.php index.html

  <Directory "/var/www/html">
    AllowOverride All
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~
### Y el .htaccess que se debería ver así:
~~~
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
~~~
### Una vez bien puestos estos archivos vamos a crear los scripts que van a ir en la máquina del frontend(que solo hay que ejecutarlo en la máquina virtual del frontend)
### Primero creamos el install_lamp_frontend que paso por paso se vería así:
#### Muestra todos los comandos que se van ejecutando
~~~
set -ex
~~~
#### Actualizamos los repositorios
~~~
apt update
~~~
#### Actualizamos los paquetes
~~~
#apt upgrade -y
~~~
### #Instalamos el servidor web Apache
~~~
sudo apt install apache2 -y
~~~
#### Instalamos PHP
~~~
apt install php libapache2-mod-php php-mysql -y
~~~
#### Copiamos el archivo conf de apache 
~~~
cp ../conf/000-default.conf /etc/apache2/sites-available
~~~
#### Habilitammos el modúlo rewrite
~~~
a2enmod rewrite
~~~
#### Reiniciamos el servicio de Apache
~~~
systemctl restart apache2
~~~
#### Copiamos el archivo de php 
~~~
cp ../php/index.php /var/www/html
~~~
#### Modificamos el propietario y el grupo del directorio /var/www/html
~~~
chown -R www-data:www-data /var/www/html
~~~
### El siguiente que vammos a crear es el deploy del frontend, donde vamos a desplegar la aplicación web que en nuestro caso  va a ser wordpress que paso por paso se vería así:
#### Muestra todos los comandos que se van ejecutando
~~~
set -ex
~~~
#### Actualizamos los repositorios
~~~
apt update
~~~
#### Actualizamos los paquetes
~~~
#apt upgrade -y
~~~
#### Importamos el archivo de variables .env
~~~
source .env
~~~
#### Eliminamos descargas previas 
~~~
rm -rf /tmp/wp-cli.phar
~~~
#### Descargamos la utilidaad wp-cli 
~~~
wget  https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -P /tmp
~~~
#### Le asignamos permisos de ejecución al archivo wp-cli.phar
~~~
chmod +x /tmp/wp-cli.phar
~~~
#### Movemos la utilidad wp-cli al directorio /usr/local/bin
~~~
mv /tmp/wp-cli.phar /usr/local/bin/wp
~~~
#### Eliminamos instalaciones previas de wordpress
~~~
rm -rf /var/www/html/*
~~~
#### Descargamos el código fuente de WordPress en /var/www/html
~~~
wp core download --locale=es_ES --path=/var/www/html --allow-root
~~~
#### Creamos el archivo wp-config 
~~~
wp config create \
  --dbname=$WORDPRESS_DB_NAME \
  --dbuser=$WORDPRESS_DB_USER \
  --dbpass=$WORDPRESS_DB_PASSWORD \
  --dbhost=$WORDPRESS_DB_HOST \
  --path=/var/www/html \
  --allow-root
~~~
#### Instalamos worpress
~~~
  wp core install \
  --url=$CB_DOMAIN \
  --title="$WORDPRESS_TITLE" \
  --admin_user=$WORDPRESS_ADMIN_USER \
  --admin_password=$WORDPRESS_ADMIN_PASS \
  --admin_email=$WORDPRESS_ADMIN_EMAIL \
  --path=/var/www/html \
  --allow-root
~~~
#### Actualizamos los plugins 
~~~
  wp core update --path=/var/www/html --allow-root
~~~
#### Actualizamos los temas 
~~~
  wp theme update --all --path=/var/www/html --allow-root
~~~
#### Instalo un tema
~~~
  wp theme install $TEMA --activate --path=/var/www/html --allow-root
~~~
#### Actualizamos los plugins
~~~
  wp plugin update --all --path=/var/www/html --allow-root
~~~
#### Instalar y activar un  plugin
~~~
  wp plugin install $PLUGIN --activate --path=/var/www/html --allow-root
  wp plugin install $PLUGIN2 --activate --path=/var/www/html --allow-root
~~~
#### Reescribimos la estructura para elegir postname en la estructura de wp
~~~
  wp rewrite structure '/%postname%/' --path=/var/www/html --allow-root
~~~
#### Actualizamos la opción whl_page 
~~~
  wp option update whl_page 'acceso' --path=/var/www/html --allow-root
~~~
#### Copiamos el nuevo archivo .htaccess
~~~
  cp ../htaccess/.htaccess /var/www/html
~~~
#### Modificamos el propietario y el grupo del directorio /var/www/html
~~~
  chown -R www-data:www-data /var/www/html
~~~
### Ahora tendremos que hacer los ddos scripts también para el backend que lo vamos a usar para configurar la base de datos.
### Primero vamos a hacer el install_lamp_backend que paso por paso se vería así:
#### Muestra todos los comandos que se van ejecutando
~~~
set -ex
~~~
#### Actualizamos los repositorios
~~~
apt update
~~~
#### Actualizamos los paquetes
~~~
#apt upgrade -y
~~~
#### Importamos el archivo .env 
~~~
source .env
~~~
#### Instalamos el gestor de bases de datos MySQL
~~~
apt install mysql-server -y
~~~
#### Configuramos mysql para que solo acepte conexiones desde la Ip privada
~~~
sed -i "s/127.0.0.1/$MYSQL_PRIVATE_IP/" /etc/mysql/mysql.conf.d/mysqld.cnf
~~~
#### Reiniciamos el servicio de mysql
~~~
systemctl restart mysql
~~~
### Después tenemos que crear el script del deploy_backend para desplegar el mysql en la máquina que paso a paso se vería así:
#### Muestra todos los comandos que se van ejecutando
~~~
set -ex
~~~
#### Actualizamos los repositorios
~~~
apt update
~~~
#### Actualizamos los paquetes
~~~
#apt upgrade -y
~~~
#### Importamos el archivo de variables .env
~~~
source .env
~~~
#### Creamos la base de datos y el usuario de la base de datos 
~~~
mysql -u root <<< "DROP DATABASE IF EXISTS $WORDPRESS_DB_NAME"
mysql -u root <<< "CREATE DATABASE $WORDPRESS_DB_NAME"
mysql -u root <<< "DROP USER IF EXISTS $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
mysql -u root <<< "CREATE USER $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL IDENTIFIED BY '$WORDPRESS_DB_PASSWORD'"
mysql -u root <<< "GRANT ALL PRIVILEGES ON $WORDPRESS_DB_NAME.* TO $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
~~~
### Una vez ya creados los scripts los tenemosq que ejecutar de la siguiente manera: 
1. Ejecutamos el script del install_lamp_frontend en la máquina que tenemos creada del frontend
2. Ejecutamos el install_lamp_backend en la máquina del backend
3. Ejecutamos el setup_letsencrypt para asignarle el dominio a la máquina, lo ejecutamos en la maquina del frontend y en el no ip tenemos que poner la ip pública de la máquina del frontend
4. Ejecutamos el script dle deploy_backend en la máquina del backend
5. Y por último ejecutamops el deploy_frontend en la máquina del frontend
### Una vez ejecutados así ya tendriamos listo nuestro wordpress automatizado en nuestra página web
