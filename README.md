# **Práctica Servidores web**
## 1er Trimestre

### 1. Instalación del servidor Apache

>Usaremos dos dominios mediante el archivo hosts: centro.intranet y departamentos.centro.intranet. El primero servirá el contenido mediante wordpress y el segundo una aplicación en python.

Como primer paso, desde el terminal de Linux de la máquina virtual, ejecutar los siguientes comandos:

```
sudo apt update
sudo apt install apache2
```

Si es la primera vez que utiliza sudo en esta sesión, se le pedirá que proporcione su contraseña de usuario para confirmar que tenga los privilegios adecuados para administrar los paquetes del sistema con apt. También se le solicitará que confirme la instalación de Apache al pulsar Y y ENTER.

Una vez que la instalación se complete, deberá ajustar la configuración de su firewall para permitir tráfico HTTP y HTTPS. UFW tiene diferentes perfiles de aplicaciones que puede aprovechar para hacerlo. Para enumerar todos los perfiles de aplicaciones de UFW disponibles, puede ejecutar lo siguiente:

```
sudo ufw app list
```

Verá un resultado como este:

```
Output
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

Para permitir tráfico únicamente en el puerto 80 utilice el perfil Apache:

```
sudo ufw allow in "Apache"
```

Puede realizar una verificación rápida para comprobar que todo se haya realizado según lo previsto dirigiéndose a la dirección IP pública de su servidor en su navegador web o, en su defecto, introduciendo "localhost" en la barra de navegación:

```
http://127.0.0.1
o
localhost
```
Se visualizará la siguiente página por defecto de Apache

![PruebaApache](/Instalación-del-servidor-web-apache/pruebaInstApache.PNG)


### 2. Activación de módulos

>Activar los módulos necesarios para ejecutar php y acceder a mysql.



### 3. WordPress

>Instala y configura wordpress.



### 4. Activación de "wsgi"

>Activar el módulo “wsgi” para permitir la ejecución de aplicaciones Python.



### 5. Aplicación Python

>Crea y despliega una pequeña aplicación python para comprobar que funciona correctamente.



### 6. Protección de acceso a la aplicación Python

>Adicionalmente protegeremos el acceso a la aplicación python mediante autenticación.



### 7. AWStat

>Instala y configura awstat.



### 8. Instalación de segundo servidor y phpmyadmin

>Instala un segundo servidor de tu elección (nginx, lighttpd) bajo el dominio “servidor2.centro.intranet”. Debes configurarlo para que sirva en el puerto 8080 y haz los cambios necesarios para ejecutar php. Instala phpmyadmin.
