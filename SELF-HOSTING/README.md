# WebServer by Ángel Serrano #

The objective of this practice is to perform hosting on your own servers placed at the student's home. This task is called self-hosting and has certain complications that are very interesting to learn for future administrators.

* ### **VAGRANTFILE** ###
  
#### To start we create a "Vagrantfile" by creating a Debian virtual machine. We will call this machine "WebServerAngel" and we will set a fixed IP. In addition to open ports. ####

#### We will perform a "provision" where we will make all file changes and copies of these. We will update the packages and changes made, we will apply the certificates. ####
#### We will edit the home page (index.php), a page in case an error occurs (error404.html) and a page for administration (admin.html) which can only be accessed by logging in. In addition, with the user "sysadmin" we will be able to enter the status of the server. ####
 
 
   
```conf
#-- mode: ruby --
#vi: set ft=ruby :
Vagrant.configure("2") do |config|
  
  #Virtual machine configuration
  config.vm.define "WebServerAngel" do |web|
    web.vm.box = "debian/bullseye64"
    web.vm.hostname = "WebServerAngel"
    web.vm.network "public_network", ip: "192.168.1.133"
    
    #Port configuration
    web.vm.network "forwarded_port", guest: 80, host: 8080
    web.vm.network "forwarded_port", guest: 443, host: 8443
    
    web.vm.provider "virtualbox" do |vb|
      vb.memory = "254" 
      vb.cpus = 2 
    end
    
    # Provision configuration
    web.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y apache2
    sudo apt-get install -y php libapache2-mod-php
    
    #Copy certificate and private key
    sudo mkdir /etc/apache2/ssl
    sudo cp /vagrant/certificadoAngel.cer /etc/apache2/ssl/
    sudo cp /vagrant/clavePrivadaAngel.key /etc/apache2/ssl/
    
    #SSL certificate configuration
    sudo cp /vagrant/config/ssl-site.conf /etc/apache2/sites-available/
    sudo a2ensite ssl-site.conf
    
    #Home page configuration
    cp -v /vagrant/config/WebServerAngel/index.html /var/www/html
    
    #Custom error page configuration
    cp -v /vagrant/config/WebServerAngel/error404.html /var/www/html
    cp -v /vagrant/config/AngelServer.conf /etc/apache2/sites-available
    
    #We disable the default Apache2 configuration file and enable the one for our web site
    sudo a2dissite 000-default.conf
    sudo a2ensite AngelServer.conf
    
    #Download the photo
    cp -v /vagrant/logo.png /var/www/html/
    
    #Admin page with photo with password
    cp -v /vagrant/config/WebServerAngel/admin.html /var/www/html

    #We create and establish users and passwords.
    sudo cp -v /vagrant/config/.htpasswd /etc/apache2/.htpasswd

    sudo htpasswd -c -b /etc/apache2/.htpasswd admin asir
    sudo htpasswd -b /etc/apache2/.htpasswd sysadmin risa
    sudo aenmod auth_digest
    sudo a2enmod ssl

    #Enable mod_status
    sudo a2enmod status

    #Restart the Apache service
    sudo systemctl restart apache2
    SHELL
  end
end
```

As we can see, in the "Vagrantfile" itself we have disabled the apache configuration file and enabled our own.

 ```conf
sudo a2dissite 000-default.conf
sudo a2ensite AngelServer.conf
```

We have named this file "AngelServer.conf", where we will configure the apache.

* ### **AngelServer.conf** ###

#### In this file, we will enable the error document (error404.html). ####
#### In addition, we will establish the basic authentication, which will take the users and passwords from the ".htpasswd" file. This authentication will be used both to log in to "admin.html" and to observe the server status. ####

  ```conf

   <VirtualHost *:80>
  
        ServerAdmin webmaster@angelsp.es
        DocumentRoot /var/www/html
        ErrorDocument 404 /error404.html

        <Directory /var/www/html/admin.html>
                AuthType Basic
                AuthName "Área restringida"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>

        <Location /status>
                SetHandler server-status
                AuthType Basic
                AuthName "Área restringida"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
                Require user sysadmin
        </Location>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  
   </VirtualHost>
```

* ### **ssl-site.conf** ###
  
#### In the following file we will introduce the users and their corresponding passwords to be able to authenticate. ####
#### As we can see, passwords must be encrypted. ####

```conf
admin:TEofM+NkPhIBKmmU+M6UrF686OrlkkU9Fok8juBSE/3Y2gl8eiwxAXiVA+9hwg4h
sysadmin:T5eDwuaCYLw42p2M094zHNY1YUsWnsoH31uex8ry2kjVby+SO6GzOVM3NZJgPURs
```

#### To enable the certificates it is not only enough to insert them in the respective folders as we have configured in the "Vagrantfile". It is also necessary to activate the SSL/TLS support and with it, we assign the paths of both the certificate and the private key. To do this, we configure the file "ssl-site.conf". ####

* ### **ssl-site.conf** ###

```conf
   <IfModule mod_ssl.c>
       <VirtualHost *:443>
           ServerName angelsp.es
           DocumentRoot /var/www/html

           SSLEngine on
           SSLCertificateFile /etc/apache2/ssl/certificadoAngel.cer
           SSLCertificateKeyFile /etc/apache2/ssl/clavePrivadaAngel.key
       </VirtualHost>
   </IfModule>
```

#### The next files that we will edit will be the web pages themselves. ####

* ### **index.html** ###

#### This file will be the home page of my web server. ####

```conf
<!DOCTYPE html>
<html>
    <head>
        <title>WebServerAngel</title>
        <meta charset="UTF-8"/>
        <style type="text/css">
            *{
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		font-family: 'Montserrat Alternates', sans-serif;
            }
            html{
                scroll-behavior: smooth;
            }
            body{
                width: 100%;
                background: linear-gradient(135deg, green, black);
                color: white;
            }
            .header{
                width: 100%;
                height: 100px;
                position: fixed;
                top: 0;
                left: 0;
                z-index: 2;
                background:rgb(65, 65, 65);
            }
            .container{
                width: 90%;
                max-width: auto;
                margin: auto;
            }
            .container .btn-menu, .logo{
                float: left;
                line-height:100px;
            }
            .container .btn-menu label{
                color: #fff;
                font-size: 25px;
                cursor: pointer;
            }
            .logo h1{
                color: #fff;
                font-weight: 400;
                font-size: 22px;
                margin-left: 10px;
            }
            .container .menu{
                float: right;
                line-height: 100px;
            }
            .container .menu a{
                display: inline-block;
                padding: 15px;
                line-height: normal;
                text-decoration: none;
                color: #fff;
                transition: all 0.3s ease;
                border-bottom: 2px solid transparent;
                font-size: 15px;
                margin-right: 5px;
            }
            .container .menu a:hover{
                border-bottom: 2px solid #c7c7c7;
                padding-bottom: 5px;
            }

            #btn-menu{
                display: none;
            }
            .container-menu{
                position: absolute;
                background: rgba(68, 68, 68,);
                width: 100%;
                height: 100vh;
                top: 0;
                left: 0;
                transition: all 500ms ease;
                opacity: 0;
                visibility: hidden;
                z-index: 12;
            }
            #btn-menu:checked ~ .container-menu{
                opacity: 1;
                visibility: visible;
            }
            .cont-menu{
                width: 100%;
                position: sticky;
                top: 0;
                max-width: 250px;
                background: #1c1c1c;
                height: 100vh;
                position: relative;
                transition: all 500ms ease;
                transform: translateX(-100%);
                z-index: 12;
            }
            #btn-menu:checked ~ .container-menu .cont-menu{
                transform: translateX(0%);
            }
            .cont-menu nav{
                transform: translateY(15%);
            }
            .cont-menu nav a{
                display: block;
                text-decoration: none;
                padding: 20px;
                color: #c7c7c7;
                border-left: 5px solid transparent;
                transition: all 400ms ease;
            }
            .cont-menu nav a:hover{
                border-left: 5px solid #c7c7c7;
                background: #1f1f1f;
            }
            .cont-menu label{
                position: absolute;
                right: 5px;
                top: 10px;
                color: #fff;
                cursor: pointer;
                font-size: 18px;
            }

            .contenedor-principal{
                height: 100vh;
                overflow: auto;  
                scroll-snap-type: y mandatory;
            }
            h2{
                font-size: 3em;
                text-align: center;
                color: white;
                padding: 10% 0 0 0;
                letter-spacing: 5px;
            }
            .secciones{
                width: 100%;
                height: 100vh;
                scroll-snap-align: start;
                min-height: 60vh;
                display: block;
            }
            .contenedor-informacion{
                display: flex;
                justify-content: center;
            }
            p{
                margin-top: 25vh;
                margin-left: 5vh;
                font-size: 1.5em;
            }
           
        </style>
    </head>
    <body>
        <header class="header">
            <div class="container">
                <div class="btn-menu">
                    <label for="btn-menu">☰</label>
                </div>
                <div class="logo">
                    <h1>Servidor de Ángel</h1>
                </div>
                <nav class="menu">
                    <a href="index.html">Intro</a>
                    <a href="admin.html">Admin</a>
                </nav>
            </div>
        </header>
        <input type="checkbox" id="btn-menu">
        <div class="container-menu">
            <div class="cont-menu">
                <nav>
                    <a href="https://github.com/Angelserpra">GitHub</a>
                </nav>
                <label for="btn-menu">✖️</label>
            </div>
        </div>
        <div class="contenedor-principal">
            <div class="secciones" id="seccion1">
                <div class="contenedor-informacion">
                    <p>Hola, este es primer servidor web</p>
                </div>
            </div>
        </div>
    </body>
</html>  
```

* ### **error404.html** ###

#### This file will be the web page that will be displayed when an error occurs. ####

  ```conf
<!-- error404.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Error 404 - Página no encontrada</title>
</head>
<body>
    <h1>Error 404</h1>
    <p>Lo sentimos, la página que buscas no se pudo encontrar.</p>
</body>
</html>
```

* ### **admin.html** ###

#### This file will be the web page where you will need to log in to display the web page. ####
#### In this web page we can see an image that we have imported in the file "Vagrantfile". ####

```conf
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Administración</title>
</head>
<body>
    <h1>Administración</h1>
    <img src="/logo.png" alt="Imagen" width="400">
</body>
</html>
```




  