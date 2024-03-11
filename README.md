![](https://p19.f3.n0.cdn.getcloudapp.com/items/NQuvEepO/68747470733a2f2f636c2e6c792f3038335a31723358323331462f646f776e6c6f61642f496d616765253230323031372d30352d32332532306174253230322e31362e3035253230414d2e706e67.png?v=c6b67635a93a2309b8e90953c1466713)

# Welcome to TUGBOAT with NGINX. <br> Run multiple websites on a single Server



> Built on top of the [Official](https://docs.docker.com/docker-hub/official_images/) Docker Images for [Php](https://hub.docker.com/_/php) and [MySQL](https://hub.docker.com/_/mysql), these preconfigured containers will jumpstart your php website or application development. If the environment requires further configurations dont sweat it, TUGBOAT can be extended to provided access to additional packages and configurations to the environment using [build scripts](https://github.com/bryanlittlefield/TUGBOAT/wiki/Running-Scripts-in-the-Web-Container-on-Build).  

- - - -
# Requirements 
> This documentation is based inside the root directory under the folder created called 'tugboat' on your VPS hosting, which is running Ubuntu. Please ensure that Docker is installed.
# Installing Laravel:
1.  Add the following new records to the .env file in the root directory. <br/>
File directory: /root/tugboat/.env <br/>
Example:
```
# ==============================
# EXAMPLE WEBSITE
# ==============================
EXAMPLE_ROOT_PATH=/mount/example_folder
EXAMPLE_HTML_DOCUMENT_ROOT="${DONUTFY_BACKEND_ROOT_PATH_STAGING}${DOCUMENT_ROOT}"
EXAMPLE_URL=example.com
```
2. Add a new service to the docker-compose file like below. Make sure spacing is correct. Change everything that says "example" to your project name. <br/>
File directory: /root/tugboat/docker-compose.yml <br/>
```
  example_name:
      env_file:
          - .${EXAMPLE_ROOT_PATH}/.env
      image: bryanjlittlefield/tugboat-php:${PHP_VERSION}
      container_name: ${PROJECT_NAME}_example_name
      volumes:
          # Mount Files to Container on build
          - .${EXAMPLE_ROOT_PATH}/scripts/build-files:/usr/local/bin/build-files
          # Mount Apaches Log Files on Host
          - .${EXAMPLE_ROOT_PATH}/var/log/apache2:/var/log/apache2
          # Example of a Host Mounted Volume
          - .${EXAMPLE_HTML_DOCUMENT_ROOT}:${DOCUMENT_ROOT}
          # Mount PHP configuration file
          - .${EXAMPLE_ROOT_PATH}/config/php/php.ini:/usr/local/etc/php/php.ini
          # Mount SSL configuration file
          - .${EXAMPLE_ROOT_PATH}/config/ssh/sshd_config:/etc/ssh/sshd_config
          # Example of a Named Data Volume
          #- web-data-volume:/var/www/html
      links:
          - mysqldb1:mysql
          - redis:redis
          - mailhog:mailhog
      restart: on-failure
      networks:
        - app
```
3. Create new folder for project based on the ROOT_PATH varible you created above for example this will be in the mount folder
<br/>
example : /mount/example_folder 
<br>
cd into this directory and run tugboat:

```
 git clone https://github.com/bryanlittlefield/TUGBOAT.git .
```

Add the following to the following and change everything that says "example"
```

# ==============================
# WEB
# ==============================
VIRTUAL_HOST="example.com"
#PHP
MAX_EXECUTION_TIME=0
MAX_INPUT_TIME=0
MAX_INPUT_VARS=1500
MEMORY_LIMIT=-1
POST_MAX_SIZE=0
UPLOAD_MAX_FILESIZE=2048M
DATE_TIMEZONE=America/Los_Angeles
XDEBUG=FALSE

# ==============================
# APACHE
# ==============================
SERVER_NAME="example.com"
WEB_ROOT="/var/www/html/"
SKIP_PERMISSIONS=true
DIRECTORY_PERMISSION=775
FILE_PERMISSION=664
INCLUDE_HTPASSWD=false
HTPASSWD_USER=tugboat
HTPASSWD_PASS=tugboat
WHITELIST_IP=
SSL_CERT_TYPE=CERTBOT
SSL_EMAIL=tugboat@coolblueweb.net
SSL_CERTIFICATE_FILE=/etc/apache2/ssl/apache.crt
SSL_CERTIFICATE_KEY_FILE=/etc/apache2/ssl/apache.key
SSL_CERTIFICATE_CHAIN_FILE=/etc/apache2/ssl.crt/server-ca.crt

#USERS
ROOT_USER_PASS=tugboat_web_root
DEV_USER_PASS=tugboat_web_dev

#CUSTOM SCRIPTS
BUILD_FILES=false
#WEBMIN
USE_WEBMIN=false
```
 Clean up: You can remove the docker-compose.yml since we are using the one in the root directory. 

4. Run the following:
```docker-compose down & docker compose up -d ```
At this point you can create a hello world app in
``` /var/www/html/index.html ```
5.  zsh into the container to do npm and artisan things. Where it says "tugboat_example" change this with the correct container name which can be find be using this command
``` docker exec -it tugboat_example zsh ```
6. Install Laravel:
``` composer create-project laravel/laravel ./ ```
Give permissions:
``` 
chown -R www-data:www-data \
        /var/www/html/storage \
        /var/www/html/bootstrap/cache \
        /var/www/html/storage/logs

```
Install dependencies 
```
composer install && php artisan key:generate
```


# Installing Wordpress App
1. Add a new record to .env file in the root directory as follows.
```
# ==============================
# EXAMPLE WORDPRESS
# ==============================
WORDPRESS_ROOT_PATH=/mount/example
WORDPRESS_URL=example.com
```
2. Create a new container in the docker.yml file:
```
  donutfy_store_staging:
    env_file:
        - .${DONUTFY_STORE_ROOT_PATH_STAGING}/.env  
    image: bryanjlittlefield/tugboat-php:${PHP_VERSION}
    container_name: ${PROJECT_NAME}_donutfy_store_staging
    volumes:
        - .${DONUTFY_STORE_ROOT_PATH_STAGING}:${DOCUMENT_ROOT}
    restart: on-failure
    environment:
        - VIRTUAL_HOST=store-staging.donutfy.com
    networks:
      - app
```
3. Run tugboat inside new directory
cd inside new directory and run
```
git clone https://github.com/bryanlittlefield/TUGBOAT.git .
```
4. Inside /var/www/html install Wordpress.
```
wget http://wordpress.org/latest.tar.gz && tar xfz latest.tar.gz && mv wordpress/* ./ && rmdir ./wordpress/ && rm -f latest.tar.gz
```
Give permissions:
```
chmod -R 777  path_to_wpcontent/www/html
```
5. Edit Files
<br/>
Copy and paste this .env inside the root directory of your wordpress project directory edit virtual host, server name and webroot:

```
# ==============================
# WEB
# ==============================
VIRTUAL_HOST="${DONUTFY_STORE_URL_PROD}"
#PHP
MAX_EXECUTION_TIME=0
MAX_INPUT_TIME=0
MAX_INPUT_VARS=1500
MEMORY_LIMIT=-1
POST_MAX_SIZE=0
UPLOAD_MAX_FILESIZE=2048M
DATE_TIMEZONE=America/Los_Angeles
XDEBUG=FALSE


# ==============================
# APACHE
# ==============================
SERVER_NAME="${DONUTFY_STORE_URL_PROD}"
WEB_ROOT="${DOCUMENT_ROOT}"
SKIP_PERMISSIONS=true
DIRECTORY_PERMISSION=775
FILE_PERMISSION=664
INCLUDE_HTPASSWD=false
HTPASSWD_USER=tugboat
HTPASSWD_PASS=tugboat
WHITELIST_IP=
SSL_CERT_TYPE=CERTBOT
SSL_EMAIL=tugboat@coolblueweb.net
SSL_CERTIFICATE_FILE=/etc/apache2/ssl/apache.crt
SSL_CERTIFICATE_KEY_FILE=/etc/apache2/ssl/apache.key
SSL_CERTIFICATE_CHAIN_FILE=/etc/apache2/ssl.crt/server-ca.crt


#USERS
ROOT_USER_PASS=tugboat_web_root
DEV_USER_PASS=tugboat_web_dev


#CUSTOM SCRIPTS
BUILD_FILES=false


#WEBMIN
USE_WEBMIN=false


# ==============================
# WORDPRESS
# ==============================


WORDPRESS_DB_HOST=mysqldb1
WORDPRESS_DB_USER=admin
WORDPRESS_DB_PASSWORD=tugboat_mysql_admin
WORDPRESS_DB_NAME=wordpress

```
Edit wp-config.php
```
define( 'DB_NAME', 'store-wordpress' );
/** Database username */
define( 'DB_USER', 'username' );
/** Database password */
define( 'DB_PASSWORD', 'password' );
/** Database hostname */
define( 'DB_HOST', 'mysqldb1' );
/** Allow plugins and themes to be installed */
define('FS_METHOD', 'direct');

```
Create .htaccess file in root directory
```
Create .htaccess file in root directory
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
```

# MYSQL env 
Go to this directory to find env variables and passwords to edit
mount/mysqldb1/.env
- - - -

##  Supported Versions:

- **PHP**:
  - 8.1
  - 8.0 (`stable`)
  - 7.4
- **MySQL**:
  - 8.1
  - 8.0 (`stable`)
  - 5.7

###  No Longer Maintained:

- **PHP**:
  - 7.3
  - 7.2
  - 7.1
  - 7.0
  - 5.6
- **MySQL**:
  - 5.6
  - 5.5
- - - -

## See our [Wiki](https://github.com/bryanlittlefield/TUGBOAT/wiki) for more information.
