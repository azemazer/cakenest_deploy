﻿# cakenest_deploy
Here is how to deploy Cakenest with its front and API on an Ubuntu server and Apache webserver to simulate a devloped environment!
Be sure to have all permissions within your Linux environment.

## Setup
Update ubuntu
```
$ sudo apt update
```
Install openssh server to allow communication with external sources, as well as git to clone repos
```
$ sudo apt-get install openssh-server
$ sudo apt-get install git
```
Install apache2 as the webserver we'll be using for the front and the back
```
$ sudo apt-get install apache2
```
## Front-end deployment
### Building the website
First of all, let's get ahead and install npm, which we will need in order to build the web app.
```
$ sudo apt-get install npm
```
Then clone [the front-end repo](https://github.com/azemazer/cakenest.git) into a folder (e.g. /var/www/)
You can then go into the folder and install the dependencies, then build the app.
```
$ sudo npm install
$ sudo npm run build
```
The built website is now in /var/www/cakenest/dist .

### Apache configuration
Now you need to configurate Apache and create a VirtualHost config linked to this directory.
Go to /etc/apache2/sites-available . If you type 'ls' you can see there is a file named '000-default.conf'. You can duplicate this file so we don't have to rewrite everything.

```
$ sudo cp 000-default.conf cakenest.conf
```
Open this file and edit the DocumentRoot path to the correct path (in this case, /var/www/cakenest/dist) .
You also need to add a ServerName like this: 'ServerName cakenest.test'
Once this is done, you need to enable the website and reload Apache so it takes the changes into account.
```
$ sudo a2ensite cakenest.conf
$ sudo systemctl reload apache2
```
Now you can access the front at cakenest.test! (provided you added it to your list of hosts in your OS with your VM's IP).

## Back-end deployment
### Setup
Clone [the back-end repo](https://github.com/azemazer/cakenest_api.git) into a folder (e.g. /var/www/)
For the back-end, we will need to install php, composer and mysql, so we can go ahead and do that.
```
$ sudo apt install php-fpm
$ sudo apt install composer
$ sudo apt install mysql
```

You will need some PHP extensions to make it run, e.g. in PHP 8.3 you need xml (and curl for composer):
```
$ sudo apt install php8.3-xml
$ sudo apt install curl
```

And you will need to allow yourself, the user, as well as the group of users www-data, to write in this folder. 
```
sudo usermod -G www-data -a owner
sudo chown -R owner:www-data cakenest_api
chmod -R g+w cakenest_api
```

### Apache configuration

Now you need to configurate Apache and create a VirtualHost config linked to this directory, just like for the front.
Go to /etc/apache2/sites-available. You can duplicate cakenest.conf.

```
$ sudo cp cakenest.conf cakenest_api.conf
```
Open this file and edit the DocumentRoot path to the correct path (in this case, /var/www/cakenest_api/public) .
You also need to add a ServerName like this: 'ServerName api.cakenest.test'
Then, add this line which will allow the api to rewrite URLs
```
<Directory /var/www/api-cakenest/public>
      AllowOverride All
      Require all granted
</Directory>
```

Once this is done, you need to enable the website and reload Apache so it takes the changes into account.
```
$ sudo a2ensite cakenest_api.conf
$ sudo systemctl reload apache2
```
### API environment & database setup
Let us set the correct values in the .env of the API. 
Might as well do it in the front.
```
API_URL = http://api.cakenest.test
```
Don't forget to rebuild the app!

And in the back, copy the .env.example into a .env and set these values:
```
APP_URL=http://api.cakenest.test
FRONTEND_URL=http://cakenest.test
DB_CONNECTION=mysql
DB_HOST=(server ip)
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=(your username)
DB_PASSWORD=(your password)
```
Now, there's only the database left.
Start the mysql service and create the user that will have access to the database
```
$ sudo systemctl start mysql.service
$ CREATE USER '(name)'@'localhost' IDENTIFIED BY '(password)';
$ GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO '(name)'@'localhost' WITH GRANT OPTION;
```
(This will grant all privileges to this user. For more security you can restrict the user's right to the app's database.)

Now all there's left to do is to migrate the database and seed it. In the API folder, run: 
```
$ php artisan migrate
$ php artisan db:seed
```
## SSL certificate
You can simulate a secure collection by using [mkcert](https://github.com/FiloSottile/mkcert). Just follow the instructions there. Don't forget if you want to do this, that in the .conf files and in the .env of the front and the back, you will have to put 'https' instead of 'http' instead.
