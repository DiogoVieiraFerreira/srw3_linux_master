    Project : Shared Hosting - CLD1.1 
    Team : Diogo Vieira - Killian Viquerat - Yvann Butticaz 
    Date : 25.09.2019
    Version : 1.1

# Documentation only work with php 7.0

# Installation of Debian

## Hardware
- **Ram :** 2GB
- **Processor and core :** 1, 1
- **Hardrive :** 20GB


## Configuration

### Installation : 
_without a GUI_
- **Language, location, time and keyboard :** depends of your location and needs
- **Packet manager :** Depend of your location
- **Disk partition :** Guided - use entire disk, all files in one partition
- **Grub :** Install it 


## Commands
> `$` : Represent the user's privilege \
> `#` : Represent the root's privilege


## Users
> - **credentials :** root, root
> - **credentials :** administrator, password


### Software installation
 - SSH Server
 - Standard system utilities


### Linux configuration
#### Updating the debian system 
To search for new updates and install them :\
`# apt-get update`\
`# apt-get upgrade`

#### Config SSH
- Open SSH config file :
    - `# nano /etc/ssh/sshd_config`
- To deny the SSH access for root user, Add this line in the file : 
    - `DenyUsers root`
- Save and close the file
- Now restart the SSH services : 
    - `# systemctl restart sshd`


#### User repositories configuration
- Change home repository permission :
    - `# chmod 751 /home`
- Now the home repository can't be read or wrote by any user
- Then open the file to change the user directory rules `/etc/adduser.conf`
    - `# nano /etc/adduser.conf` 
- Find the line `DIR_MODE` and change the value to `0750`
- Now close and save the file
- Change home/administrator permission, because it was created before changes:
    - `# chmod 750 /home/administrator`


##### Default user's dir configuration
- Go in skel's dir : `# cd /etc/skel`

_All the users will have a personnal www dir and index.php with phpinfo()_

- `# mkdir www`

- `nano www/index.php`

- write `<?php phpinfo();` and exit file while save it.

#### Create a new user

-add a new user :\
`#adduser username`

#### Installation of PHP
Intall : \
`# apt-get install php-fpm -y`

#### PHP Configuration 

We're using PHP version `7.0`.

_Always replace `username` by the user you're adding_ 

Create the file 'username'.conf in /etc/php/7.0/fpm/pool.d/`username`.conf

`'username'.conf`

```bash
    [`username`]
    user = `username`
    group = `username`
    listen = /var/run/php7.0-fpm-`username`.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off
    pm = dynamic
    pm.max_children = 5
    pm.start_servers = 2
    pm.min_spare_servers = 1
    pm.max_spare_servers = 3
    chdir = /
```


#### Installation of Nginx
Intall :\
`# apt-get install nginx -y`

To start on boot :\
`# systemctl enable nginx`

> Useful commands for nginx server
> -  To start : `# systemctl start nginx`
> -  To restart : `# systemctl restart nginx`
> -  To stop : `# systemctl stop nginx`
> -  To reload : `# systemctl reload nginx`


#### Nginx Configuration (thanks to : [nginxconfig.io](https://www.nginxconfig.io))
- Create the dir `nginxconfig.io` in `/etc/nginx/` 
    `# mkdir /etc/nginx/nginxconfig.io`
            
- add `www-data` to user group :  
`# usermod -a -G 'username' www-data`

- In Nginx add a new config file for the web client in `/etc/nginx/sites-available/'client_name'.conf`

    `client_username.conf`
    ```bash
        server {
            listen 80;
            listen [::]:80;

            server_name 'username'.ch;
            set $base /home/'username'/www;
            root $base;

            # security
            include nginxconfig.io/security.conf;

            # index.php
            index index.php;

            # index.html fallback
            location / {
                try_files $uri $uri/ /index.html;
            }

            # index.php fallback
            location ~ ^/api/ {
                try_files $uri $uri/ /index.php?$query_string;
            }

            # handle .php
            location ~ \.php$ {
                include nginxconfig.io/php_fastcgi_`username`.conf;
            }

            # additional config
            include nginxconfig.io/general.conf;
        }

    ```

    To activate the configuration of the client's site, you must enable the site by creating a symbolic link.

    `# ln -s /etc/nginx/sites-available/'client_name'.conf /etc/nginx/sites-enabled/'client_name'.conf`
    


_For all the next files, you have to save them in `/etc/nginx/nginxconfig.io`_

- Create the files : `security.conf`, `general.conf` and `php_fastcgi.conf`

    `security.conf`
    ```bash
            # security headers
            #Frame/iframe of content is only allowed from the same site origin
            add_header X-Frame-Options "SAMEORIGIN" always;
            #XSS filter enabled and prevented rendering the page if attack detected
            add_header X-XSS-Protection "1; mode=block" always;
            #Instruct browser to consider files types as defined and disallow content sniffing
            add_header X-Content-Type-Options "nosniff" always;
            #The default setting where referrer is sent to the same protocol as HTTP to HTTP, HTTPS to HTTPS.
            add_header Referrer-Policy "no-referrer-when-downgrade" always;
            #Prevent XSS, clickjacking, code injection
            add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    ```



    `general.conf`
    ```bash
            # favicon.ico
            location = /favicon.ico {
                log_not_found off;
                access_log off;
            }

            # robots.txt
            location = /robots.txt {
                log_not_found off;
                access_log off;
            }

            # assets, media
            location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
                expires 7d;
                access_log off;
            }

            # svg, fonts
            location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
                add_header Access-Control-Allow-Origin "*";
                expires 7d;
                access_log off;
            }

            # gzip
            gzip on;
            gzip_vary on;
            gzip_proxied any;
            gzip_comp_level 6;
            gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;
    ```

    _Create one file per user_
    
    `php_fastcgi_`username`.conf`
    ```bash
            # 404
            try_files $fastcgi_script_name =404;

            # default fastcgi_params
            include fastcgi_params;

            # fastcgi settings
            fastcgi_pass			unix:/var/run/php7.0-fpm-`username`.sock;
            fastcgi_index			index.php;
            fastcgi_buffers			8 16k;
            fastcgi_buffer_size		32k;

            # fastcgi params
            fastcgi_param DOCUMENT_ROOT		$realpath_root;
            fastcgi_param SCRIPT_FILENAME	$realpath_root$fastcgi_script_name;
            fastcgi_param PHP_ADMIN_VALUE	"open_basedir=$base/:/usr/lib/php/:/tmp/";
    ```

    Now reload the nginx server and PHP-FMP.\
    `# systemctl reload nginx && systemctl reload php7.0-fpm`
    

#### Installation of MariaDB
Intall : \
`# apt-get install mariadb-server -y`

Secure MariaDB server :\
`# mysql_secure_installation`

> Credentials :
> - root, root

- Change the root password : no

- Remove anonymous users :  yes

- Disallow root login remotly : yes

- Remove test database and access to it : yes

- Reload privilege tables now : yes

#### MariaDB Configuration

#### Open remote access

- Open SSH config file:\
     `nano /etc/mysql/mariadb.conf.d/50-server.cnf`
     
- search the line: \
    `bind-address            = 127.0.0.1`

- change the ip by your ip

- exit and save your file

- restart mariadb
    `systemctl restart mariadb`

#### Create new database and new user

Access to MariaDB console :

- `# mysql`

Then create a database for the user :

    MariaDB[(none)]> CREATE DATABASE `dbname`;

Now you need to create a user for this database

    MariaDB[(none)]> CREATE USER `username`@'%' IDENTIFIED BY 'password';

Grant privileges to the user for his database

    MariaDB[(none)]> GRANT EXECUTE, SELECT, SHOW VIEW, ALTER, ALTER ROUTINE, CREATE, CREATE ROUTINE, CREATE TEMPORARY TABLES, CREATE VIEW, DELETE, INDEX, INSERT, REFERENCES, TRIGGER, UPDATE ON `dbname`.* TO 'username'@'%';

Apply changes made

    MariaDB[(none)]> FLUSH PRIVILEGES;

To exit mariaDB console:

    MariaDB[(none)]> exit



