# **Práctica Servidores web**
## 1er Trimestre

### 1. Instalación del servidor Apache

>Usaremos dos dominios mediante el archivo hosts: centro.intranet y departamentos.centro.intranet. El primero servirá el contenido mediante wordpress y el segundo una aplicación en python.

Como primer paso, desde el terminal de Linux de la máquina virtual, ejecutar los siguientes comandos:

```bash
sudo apt update
sudo apt install apache2
```

Si es la primera vez que utiliza sudo en esta sesión, se le pedirá que proporcione su contraseña de usuario para confirmar que tenga los privilegios adecuados para administrar los paquetes del sistema con apt. También se le solicitará que confirme la instalación de Apache al pulsar Y y ENTER.

Una vez que la instalación se complete, deberá ajustar la configuración de su firewall para permitir tráfico HTTP y HTTPS. UFW tiene diferentes perfiles de aplicaciones que puede aprovechar para hacerlo. Para enumerar todos los perfiles de aplicaciones de UFW disponibles, puede ejecutar lo siguiente:

```bash
sudo ufw app list
```

Verá un resultado como este:

```bash
Output
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

Para permitir tráfico únicamente en el puerto 80 utilice el perfil Apache:

```bash
sudo ufw allow in "Apache"
```

Puede realizar una verificación rápida para comprobar que todo se haya realizado según lo previsto dirigiéndose a la dirección IP pública de su servidor en su navegador web o, en su defecto, introduciendo "localhost" en la barra de navegación:

```
http://127.0.0.1
o
localhost
```
Se visualizará la siguiente página por defecto de Apache

![PruebaApache](https://github.com/davip95/Practica-Servidores-Web/blob/2e949fc0b873bfdb965dc88e075a6f8a160e34c5/Instalacion%20del%20servidor%20web%20apache/pruebaInstApache.PNG)

Para añadir los dos nuevos dominios, editamos el archivo *hosts* ubicado en /etc con permisos root:

```bash
sudo nano host
```

Añadiremos las dos siguientes líneas:

```
127.0.0.1 centro.intranet
127.0.0.1 departamentos.centro.intranet
```

Deberá quedar algo similar a esta imagen:

![DominiosHosts](https://github.com/davip95/Practica-Servidores-Web/blob/9f4b8089d4b982ae54e858edfd456639b4d67985/Instalacion%20del%20servidor%20web%20apache/dominioshosts.PNG)

A continuación guardaremos los cambios en el archivo.

### 2. Activación de módulos

>Activar los módulos necesarios para ejecutar php y acceder a mysql.

Para instalar MySQL, introduciremos el comando:

```bash
sudo apt install mysql-server
```

Cuando se le solicite, confirme la instalación al escribir Y y, luego, ENTER. Al terminar, compruebe si puede iniciar sesión en la consola de MySQL al escribir lo siguiente:

```bash
sudo mysql
```

Deberá ver una pantalla parecida a la siguiente:

![MySQL](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/mysql.PNG)

Para salir de la consola de MySQL, escriba lo siguiente:

```
mysql> exit
```

Como siguiente paso, instalaremos PHP.

Además del paquete php, necesitará php-mysql, un módulo PHP que permite que este se comunique con bases de datos basadas en MySQL. También necesitará libapache2-mod-php para habilitar Apache para gestionar archivos PHP. Los paquetes PHP básicos se instalarán automáticamente como dependencias.

Para instalar estos paquetes, ejecute lo siguiente:

```bash
sudo apt install php libapache2-mod-php php-mysql
```

Una vez que la instalación se complete, podrá ejecutar el siguiente comando para confirmar su versión de PHP:

```bash
php -v
```

Le tendrá que aparecer una pantalla con información similar a la siguiente:

![PHP](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/php.PNG)

### 3. WordPress

>Instala y configura wordpress.

#### Instalación de extensiones de PHP adicionales

Comenzaremos instalando las extensiones necesarias de PHP para Wordrpess y MySQL ejecutando los dos siguientes comandos:

```bash
sudo apt update 
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

Necesitaremos reiniciar Apache para cargar estas nuevas extensiones:

```bash
sudo systemctl restart apache2
```

#### Creación de una base de datos de MySQL y un usuario para WordPress

A continuación, crearemos una base de datos de MySQL y un usuario para WordPress. Para comenzar, inicie sesión en la cuenta root de MySQL (administrativa) emitiendo este comando:

```bash
mysql -u root -p
```

Si no puede acceder a su base de datos MySQL a través de root, como usuario sudo puede actualizar la contraseña de su usuario root iniciando sesión en la base de datos de esta forma:

```bash
sudo mysql -u root
```

Una vez que reciba la instrucción de MySQL, puede actualizar la contraseña del usuario root. Aquí, sustituya new_password por una contraseña segura de su elección.

```SQL
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password';
```

Ahora puede escribir *EXIT;* y puede volver a iniciar sesión en la base de datos a través de la contraseña con el siguiente comando:

```bash
mysql -u root -p
```

En la base de datos, puede crear una base de datos exclusiva para que WordPress la controle. Puede ponerle el nombre que quiera, pero usaremos el nombre wordpress en esta guía. Cree la base de datos para WordPress escribiendo lo siguiente:

```SQL
mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

A continuación, crearemos una cuenta de usuario separada de MySQL que usaremos exclusivamente para realizar operaciones en nuestra nueva base de datos. Crear bases de datos y cuentas específicas puede ayudarnos desde el punto de vista de administración y seguridad. Usaremos el nombre wordpressuser en esta guía, pero puede usar el nombre que sea más relevante para usted.

Crearemos esta cuenta, configuraremos una contraseña y concederemos acceso a la base de datos que hemos creado. Podemos hacerlo escribiendo el siguiente comando. Recuerde elegir una contraseña segura aquí para su usuario de base de datos donde tenemos password:

```SQL
mysql> CREATE USER 'wordpressuser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

A continuación, deje saber a la base de datos que nuestro wordpressuser debería tener acceso completo a la base de datos que configuramos:

```SQL
mysql> GRANT ALL ON wordpress.* TO 'wordpressuser'@'%';
```

Ahora tiene una base de datos y una cuenta de usuario, creadas específicamente para WordPress. Debemos eliminar los privilegios de modo que la instancia actual de MySQL sepa sobre los cambios recientes que hemos realizado:

```SQL
mysql> FLUSH PRIVILEGES;
```

Por último, cierre MySQL escribiendo *EXIT;*.

####  Ajuste de la configuración de Apache para permitir reemplazos y reescrituras .htaccess

A continuación, realizaremos algunos ajustes de menor importancia en nuestra configuración de Apache. Debe tener un archivo de configuración para su sitio en el directorio */etc/apache2/sites-available/*.

Utilizaremos */etc/apache2/sites-available/wordpress.conf*, pero debe sustituir la ruta a su archivo de configuración cuando proceda. Además, emplearemos */var/www/wordpress* como el directorio root de nuestra instalación de WordPress. Debería usar el root web especificada en su propia configuración.

Con nuestras rutas identificadas, podemos pasar a trabajar con .htaccess de forma que Apache pueda manejar los cambios en la configuración directorio por directorio.

Abra el archivo de configuración de Apache para su sitio web con un editor de texto como nano.

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Para permitir archivos .htaccess, debemos configurar la directiva AllowOverride dentro de un bloque Directory orientado a nuestro root de documentos. Agregue el siguiente bloque de texto dentro del bloque VirtualHost en su archivo de configuración. Asegúrese de utilizar el directorio root web correcto:

```bash
<Directory /var/www/wordpress/>
	AllowOverride All
</Directory>
```

![ConfigWordPress](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWordpress.PNG)

Cuando termine, guarde y cierre el archivo. 

A continuación, podemos habilitar mod_rewrite para usar la característica de permalink de WordPress:

```bash
sudo a2enmod rewrite
```

Antes de implementar los cambios realizados, compruebe que no hay errores de sintaxis ejecutando la siguiente prueba.

```bash
sudo apache2ctl configtest
```

Puede recibir un resultado como el siguiente:

![ConfigWordPress2](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWordpress2.PNG)

En tanto el resultado contenga Sintaxis OK, podrá continuar.

Reinicie Apache para implementar los cambios: Asegúrese de reiniciar ahora, incluso si ha reiniciado anteriormente en este tutorial.

```bash
sudo systemctl restart apache2
```

#### Descargar WordPress

A continuación, descargaremos y configuraremos el propio WordPress.

Cambie a un directorio que permita la escritura (recomendamos uno temporal como /tmp) y descargue la versión comprimida.

```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
```

Extraiga el archivo comprimido para crear la estructura de directorios de WordPress:

```bash
tar xzvf latest.tar.gz
```

Moveremos estos archivos a nuestro root de documentos por ahora. Antes de hacerlo, podemos añadir un archivo ficticio .htaccess de modo que esté disponible para que WordPress lo use más adelante.

Cree el archivo escribiendo lo siguiente:

```bash
touch /tmp/wordpress-6.1.1/wordpress/.htaccess
```

También copiaremos sobre el archivo de configuración de muestra al nombre de archivo que lee WordPress:

```bash
cp /tmp/wordpress-6.1.1/wordpress/wp-config-sample.php /tmp/wordpress-6.1.1/wordpress/wp-config.php
```

También podemos crear el directorio de actualización, de modo que WordPress no tenga problemas de permisos al intentar hacerlo por su cuenta siguiendo una actualización a su software:

```bash
mkdir /tmp/wordpress-6.1.1/wordpress/wp-content/upgrade
```

Ahora podemos copiar todo el contenido del directorio en nuestro root de documentos. Usaremos un punto al final de nuestro directorio de origen para indicar que todo lo que está dentro del directorio debe copiarse, incluyendo archivos ocultos (como el archivo .htaccess que hemos creado):

```bash
sudo cp -a /tmp/wordpress-6.1.1/wordpress/. /var/www/wordpress
```

#### Configurar el directorio de WordPress

Antes de realizar la configuración basada en web de WordPress, debemos ajustar algunos elementos en nuestro directorio de WordPress.

Un paso importante que debemos lograr es configurar permisos de archivo razonables y la propiedad.

Empezaremos por dar la propiedad de todos los archivos al usuario y al grupo www-data. Este es el usuario como el que se ejecuta el servidor web Apache, y este último deberá poder leer y escribir archivos de WordPress para presentar el sitio web y realizar actualizaciones automáticas.

Actualice la propiedad con el comando chown que le permite modificar la propiedad del archivo. Asegúrese de apuntar al directorio relevante de su servidor.

```bash
sudo chown -R www-data:www-data /var/www/wordpress
```

A continuación, ejecutaremos dos comandos find para establecer los permisos correctos de los directorios y archivos de WordPress:

```bash
sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;
sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;
```

Estos permisos deberían hacer que pueda trabajar de forma efectiva con WordPress, pero tenga en cuenta que algunos complementos y procedimientos pueden requerir ajustes adicionales.

Ahora, debemos realizar algunos cambios en el archivo de configuración principal de WordPress.

Cuando abramos el archivo, nuestra primera tarea será ajustar algunas claves secretas para proporcionar un nivel de seguridad a nuestra instalación. WordPress proporciona un generador seguro para estos valores, para que no tenga que crear valores correctos por su cuenta. Solo se utilizan internamente, de modo que no dañará la usabilidad el tener valores complejos y seguros aquí.

Para obtener valores seguros del generador de claves secretas de WordPress, escriba lo siguiente:

```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Obtendrá valores únicos que se parecen al resultado del bloque siguiente.

![configWordpress3](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWP3.PNG)

**Advertencia: Debe solicitar valores únicos cada vez. NO copie los siguientes valores.**

Son líneas de configuración que podemos pegar directamente en nuestro archivo de configuración para establecer claves seguras. Copie el resultado que obtuvo ahora.

A continuación, abra el archivo de configuración de WordPress:

```bash
sudo nano /var/www/wordpress/wp-config.php
```
Busque la sección que contiene los valores de ejemplo para esos ajustes. Donde pone *'put your unique phrase here');*, elimine esas líneas y pegue los valores que copió de la línea de comandos anteriores:

![configWordpress4](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWP4.PNG)

A continuación, vamos a modificar algunos de los ajustes de conexión de la base de datos al principio del archivo. Debe ajustar el nombre de la base de datos, su usuario y la contraseña asociada que configuramos dentro de MySQL.

El otro cambio que debemos realizar es configurar el método que debe emplear WordPress para escribir el sistema de archivos. Debido a que hemos dado permiso al servidor web para escribir donde debe hacerlo, podemos fijar de forma explícita el método del sistema de archivos a “direct”. Si no lo configuramos con nuestros ajustes actuales, WordPress solicitaría las credenciales de FTP cuando realicemos algunas acciones.

Este ajuste se puede agregar debajo de los ajustes de conexión de la base de datos o en cualquier otra parte del archivo:

```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define('FS_METHOD', 'direct');
```

Guarde y cierre el archivo cuando termine.

#### Completar la instalación a través de la interfaz web

Como paso previo, asigno el dominio *centro.intranet* al directorio de WordPress. Para ello, modifico el archivo de configuración de WordPress en el directorio *sites-available* de Apache (*wordpress.conf*) con  nano:

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

![ConfigWP5](https://github.com/davip95/Practica-Servidores-Web/blob/a82c71022570c0e7c18f94342f7cd8763e0acd30/Instalacion%20del%20servidor%20web%20apache/configWP5.PNG)

Ahora que la configuración del servidor está completa, podemos finalizar la instalación a través de la interfaz web.

En su navegador web, vaya al nombre de dominio *centro.intranet*

```
https://centro.intranet
```

Seleccione el idioma que desee usar:

![Idioma](https://github.com/davip95/Practica-Servidores-Web/blob/6c81f8db00759b9a9b97f55abd51f395c6513dcc/Instalacion%20del%20servidor%20web%20apache/idioma.PNG)

A continuación, accederá a la página principal de configuración.

Seleccione un nombre para su sitio WordPress y seleccione un nombre de usuario. Se recomienda elegir algo único y evitar nombres de usuario comunes como “admin” por motivos de seguridad. De forma automática, se generará una contraseña segura. Guárdela o seleccione una contraseña segura alternativa.

Introduzca su dirección de correo electrónico y defina si quiere que los motores de búsqueda no indexen su sitio:

![WPRegistro](https://github.com/davip95/Practica-Servidores-Web/blob/a82c71022570c0e7c18f94342f7cd8763e0acd30/Instalacion%20del%20servidor%20web%20apache/wpregistro.PNG)

Cuando haga clic para seguir, irá a una página que le pide que inicie sesión. Tras iniciar sesión, accederá al panel de administración de WordPress.

### 4. Activación de "wsgi"

>Activar el módulo “wsgi” para permitir la ejecución de aplicaciones Python.

#### Instalación de mod_wsgi en Apache

Primero habilitamos *mod_wsgi* en Apache, para lo que bastará con instalar el paquete *libapache2-mod-wsgi*:

```bash
sudo apt-get install libapache2-mod-wsgi-py3
```

### 5. Aplicación Python

>Crea y despliega una pequeña aplicación python para comprobar que funciona correctamente.

#### Crear la aplicación de Python

Luego, crearemos el script para la aplicación Python:

```bash
sudo nano /var/www/html/application.py
```

Dicha aplicación sólo se encargará de definir una función, que actúe con cada petición del usuario. Esta función, deberá ser una función WSGI aplicación válida. Esto significa que:

  1. Deberá llamarse *application*
  2. Deberá recibir dos parámetros: *environ*, del módulo *os*, que provee un diccionario de las peticiones HTTP estándar y otras variables de entorno, y la función *start_response*, de WSGI, encargada de entregar la respuesta HTTP al usuario

```python
def application(environ, start_response):
    status = '200 OK'
    output = b'<p>Bienvenido a mi <b>PythonApp</b>!!!</p>\n'
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    start_response(status, response_headers)
    return [output]
```

Tras crearla, le doy los permisos:

```bash
sudo chown www-data:www-data /var/www/html/application.py
sudo chmod 775 /var/www/html/application.py
```

#### Configurar el VirtualHost

Una variable del *VirtualHost*, será la encargada de redirigir todas las peticiones públicas del usuario, hacia nuestra aplicación. Y la variable que se encargue de esto, será el alias *WSGIScriptAlias*:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Una vez allí, escribimos el contenido del nuevo virtual host:

```bash
<VirtualHost*:80>
  ServerName departamentos.centro.intranet
  ServerAlias www.departamentos.centro.intranet
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
  WSGIScriptAlias /application /var/www/html/application.py
</VirtualHost>
```

Una vez configurado nuestro VirtualHost:

  1. Habilitamos el sitio web: **sudo a2ensite 000-default**
  2. Recargamos Apache: **sudo service apache2 reload**
  3. Si no lo hicimos en el punto 1 de este tutorial, habilitamos el sitio en nuestro host: **sudo nano /etc/hosts** y allí agregamos la siguiente línea: **127.0.0.1 departamentos.centro.intranet**

A partir de ahora, si abrimos nuestro navegador Web e ingresamos la url http://departamentos.centro.intranet/application veremos la frase: "Bienvenido a mi PythonApp":

![AppPython](https://github.com/davip95/Practica-Servidores-Web/blob/6a0943c7ac0245803afdfc5bc05b3fb70fe56609/Instalacion%20del%20servidor%20web%20apache/appPython.PNG)

### 6. Protección de acceso a la aplicación Python

>Adicionalmente protegeremos el acceso a la aplicación python mediante autenticación.

Para el acceso mediante autenticación, crearemos primero un usuario y una contraseña para probarla:

```bash
sudo htpasswd -c /etc/apache2/.htpasswd davidpy
```

Después de ejecutar el comando, nos pedirá la contraseña para el nuevo usuario.

A continuación, accederemos al archivo de configuración y añadiremos una directiva *Directory* para obligar el uso de autenticación.

```bash
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

```bash
<Directory /var/www/html>
        AuthType Basic
        AuthName "Secure area - Authentication required"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
</Directory>
```

Por último, comprobaremos que nos pide la autenticación cuando se inicia la aplicación Python:

![AutPython](https://github.com/davip95/Practica-Servidores-Web/blob/6a0943c7ac0245803afdfc5bc05b3fb70fe56609/Instalacion%20del%20servidor%20web%20apache/autenticacion.PNG)

### 7. AWStats

>Instala y configura awstats.

#### Instalación del paquete AWStats

Por defecto, el paquete AWStats está disponible en el repositorio Ubuntu. Puede instalarlo ejecutándolo:

```bash
sudo apt-get install awstats
```

A continuación, deberá habilitar el módulo CGI en Apache. Puede hacerlo corriendo:

```bash
sudo a2enmod cgi
```

Ahora, reinicie Apache para reflejar los cambios.

```bash
sudo service apache2 restart
```

#### Configuración de AWStats

Debemos crear un archivo de configuración para cada dominio o sitio web del que deseemos ver estadísticas. En este ejemplo crearemos un archivo de configuración para *test.com*.

Puede hacer esto duplicando el archivo de configuración por defecto de AWStats a uno con su nombre de dominio:

```bash
sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.test.com.conf
```

Ahora, necesitamos hacer algunos cambios en el archivo de configuración:

```bash
sudo nano /etc/awstats/awstats.test.com.conf
```

Actualizaremos la configuración que se muestra a continuación:

```bash
# Cambiar al archivo de registro de Apache, por defecto es /var/log/apache2/access.log
LogFile="/var/log/apache2/access.log"
# Cambiar el nombre de dominio del sitio web
SiteDomain=test.com
HostAliases=test.com localhost 127.0.0.1
# Cuando este parámetro se establece en 1, AWStats añade un botón en la página del informe para permitir «actualizar» las estadísticas desde un navegador web.
AllowToUpdateStatsFromBrowser=1
```

Guardaremos y cerraremos el archivo.

Después de estos cambios, necesitamos construir las estadísticas iniciales que se generarán a partir de los registros actuales en nuestro servidor. Podemos hacerlo utilizando:

```bash
sudo /usr/lib/cgi-bin/awstats.pl -config=test.com -update
```

La salida se verá algo así:

![AWStats](https://github.com/davip95/Practica-Servidores-Web/blob/6495333a868fb85d6c9c50260b145a9c0fbc01b6/Capturas/awstats.PNG)

#### Configuración de Apache para AWStats

A continuación, debemos configurar Apache2 para que muestre estas estadísticas. Ahora copiaremos el contenido de la carpeta *cgi-bin* en el directorio raíz del documento por defecto de nuestra instalación de Apache. Por defecto se encuentra en la carpeta */usr/lib/cgi-bin*:

```bash
sudo cp -r /usr/lib/cgi-bin /var/www/html/
sudo chown www-data:www-data /var/www/html/cgi-bin/
sudo chmod -R 755 /var/www/html/cgi-bin/
```

#### Prueba AWStats

Ahora podemos acceder a las AWStats visitando la url *http://127.0.0.1/cgi-bin/awstats.pl?config=test.com*

Se verá una pantalla similar a esta:

![AWStats2](https://github.com/davip95/Practica-Servidores-Web/blob/6495333a868fb85d6c9c50260b145a9c0fbc01b6/Capturas/awstats2.PNG)

### 8. Instalación de segundo servidor y phpmyadmin

>Instala un segundo servidor de tu elección (nginx, lighttpd) bajo el dominio “servidor2.centro.intranet”. Debes configurarlo para que sirva en el puerto 8080 y haz los cambios necesarios para ejecutar php. Instala phpmyadmin.

#### Instalación y configuración de nginx

En esta práctica, instalaremos el servidor *nginx*. 

Dado que esta es nuestra primera interacción con el sistema de empaquetado apt en esta sesión, actualizaremos nuestro índice de paquetes local para que tengamos acceso a las listas de paquetes más recientes. Posteriormente, podemos instalar nginx:

```bash
sudo apt update
sudo apt upgrade
sudo apt install nginx
```

![nginx](https://github.com/davip95/Practica-Servidores-Web/blob/bc29c9bf5a2235523e7533a77bf25e7bd9630e88/Capturas/nginx.PNG)

A continuación, configuraremos el servidor para que sirva bajo el dominio *servidor2.centro.intranet* desde el puerto 8080. Para ello, modificaremos el archivo de configuración por defecto:

```bash
sudo nano /etc/nginx/sites-available/default
```
Pegue el siguiente bloque de configuración, que es similar al predeterminado, pero actualizado para nuestro nuevo puerto y nombre de dominio, así como para archivo .php:

```bash
server {
        listen 8080;
        listen [::]:8080;

        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;

        server_name servidor2.centro.intranet;

        location / {
                try_files $uri $uri/ =404;
        }
        location ~ .php$ {
          include /etc/nginx/fastcgi_params;
          fastcgi_pass unix:/run/php/php8.1-fpm.sock;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
}
```

Puede verificar si hay errores en la sintaxis de su configuración al escribir:

```bash
sudo nginx -t
```

Si se detecta algún error, vuelva al archivo de configuración para revisar el contenido antes de continuar.

Reiniciamos Nginx para habilitar sus cambios:

```bash
sudo systemctl restart nginx
```

#### Instalación y configuración de phpmyadmin

Para instalar phpMyAdmin, escribimos:

```bash
sudo apt install phpmyadmin
```

Escogemos el servidor web Apache2 y damos Enter para que continúe la instalación:

![phpmyadmin](https://github.com/davip95/Practica-Servidores-Web/blob/bc29c9bf5a2235523e7533a77bf25e7bd9630e88/Capturas/phpmyadmin.PNG)

Se abrirá el siguiente mensaje, escogemos Sí y damos a la tecla Enter:

![phpmyadmin2](https://github.com/davip95/Practica-Servidores-Web/blob/bc29c9bf5a2235523e7533a77bf25e7bd9630e88/Capturas/phpmyadmin2.PNG)

Inmediatamente se abrirá una nueva pantalla, ingresamos el password para nuestro phpMyAdmin y en la siguiente ventana confirmamos el password y damos a la tecla Enter:

![phpmyadmin3](https://github.com/davip95/Practica-Servidores-Web/blob/bc29c9bf5a2235523e7533a77bf25e7bd9630e88/Capturas/phpmyadmin3.PNG)

**Atención: Al instalar anteriormente MySQL, es posible que haya decidido habilitar el complemento Validate Password. En el momento en que se redactó este documento, habilitar este componente generará un error cuando intente establecer una contraseña para el usuario phpmyadmin.**

Para resolver esto, seleccione la opción abort a fin de detener el proceso de instalación. Luego, abra su línea de comandos de MySQL:

```bash
sudo mysql
```

O bien, si habilitó la autenticación de contraseña para el root user de MySQL, ejecute este comando y luego ingrese su contraseña cuando se le solicite:

```bash
mysql -u root -p
```

Desde la línea de comandos, ejecute el siguiente comando para deshabilitar el componente Validate Password. Tenga en cuenta que con esto en realidad no se desinstalará, sino solo se evitará que el componente sea cargado en su servidor MySQL:

```SQL
UNINSTALL COMPONENT "file://component_validate_password";
```

Después de esto, puede cerrar el cliente MySQL:

```sql
exit
```

Luego, vuelva a instalar el paquete phpmyadmin, que funcionará según lo previsto:

```bash
sudo apt install phpmyadmin
```

Una vez que se completa el comando *apt install*, phpMyAdmin se instalará por completo. Sin embargo, para que el servidor web de Nginx encuentre y sirva los archivos phpMyAdmin correctamente, deberá crear un enlace simbólico desde los archivos de instalación hasta el directorio raíz del documento de Nginx (en el caso de esta práctica, dicho directorio es */var/www/html/*):

```bash
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

Su instalación de phpMyAdmin ya está operativa. Antes de comprobarlo, reiniciaremos Nginx y PHP:

```bash
sudo systemctl restart nginx php8.1-fpm 
```

Para acceder a la interfaz, vaya al nombre de dominio de su servidor o dirección IP pública seguido de /phpmyadmin en su navegador web (en el caso de esta práctica el dominio y puerto configurados para el servidor son *servidor2.centro.intranet:8080*):

```
https://servidor2.centro.intranet:8080/phpmyadmin
```

![phpmyadmin4](https://github.com/davip95/Practica-Servidores-Web/blob/bc29c9bf5a2235523e7533a77bf25e7bd9630e88/Capturas/phpmyadmin4.PNG)

Como se mencionó anteriormente, phpMyAdmin maneja la autenticación usando las credenciales de MySQL. Esto significa que para iniciar sesión en phpMyAdmin, usaremos el mismo nombre de usuario y contraseña que normalmente usaríamos para conectarnos a la base de datos usando la línea de comando o con una API.