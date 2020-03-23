    Project : SRW3
    Team : Diogo Vieira - Killian Viquerat
    Date : 16.03.2020
    Version : 1.0
    OS utilisé : Debian 9.12
    Serveur web : NGNIX

# Installation of Debian

## Hardware
- **Ram :** 4GB
- **Processor and core :** 2, 4
- **Hardrive :** 15GB


## Configuration

### Installation :   
_with a GUI_  
- **Language, location, time and keyboard :** depends of your location and needs  
- **Packet manager :** Depend of your location  
- **Disk partition :** Guided - use entire disk, all files in one partition  
- **Interface :** KDE  
- **Grub :** Install it  



## Commands
> `$` : Represent the user's privilege  
> `#` : Represent the root's privilege


## Users
> - **credentials :** root, ...
> - **credentials :** administrator, ...


### Software installation
 - Standard system utilities


### Linux configuration
#### Updating the debian system 
To search for new updates and install them :  
`# apt-get update`  
`# apt-get upgrade`

#### Config SSH
- Open terminal :
    `$ sudo apt install openssh-server`
- Open SSH config file :
    - `# nano /etc/ssh/sshd_config`
- To deny the SSH access for all IP except specific IP add the next line : 
    - `AllowUsers = @noxcaedibux.internet-box.ch,@192.168.1.44`
    _noxcaedibux.internet-box.ch is a DNS of external administrator_
- Save and close the file
- Now restart the SSH services : 
    - `# systemctl restart sshd`

#### Installation of PHP
Install :  
`# apt-get install php-fpm -y`

#### Installation of Nginx
Install :  
`apt-get install nginx -y`

#### Installation of MariaDB
Install :  
`apt-get install mariadb-server -y`

### Installation nextcloud
#### 1. Télécharger nextcloud
Go to tmp :  
`cd /tmp`

Download nextcloud zip :  
`wget https://download.nextcloud.com/server/releases/nextcloud-18.0.2.zip`

Unzip the nextcloud zip :  
`unzip nextcloud-18.0.2.zip`

move `nextcloud` in `/var/www` and change the owner :  
`sudo chown -R www-data: /var/www/nextcloud`

#### 2. Mise en place de la DB nextcloud
Ouvrir mariaDB :  
`sudo mariadb`

Créer un nouvel utilisateur et lui donner tous les privilèges :  
```bash
CREATE USER nxtcloudadmin@localhost IDENTIFIED BY 'admin123';
GRANT ALL PRIVILEGES ON nextcloud.* TO nxtcloudadmin@localhost IDENTIFIED BY 'admin123';
flush privileges;
exit;
```

Change global format to `Barracuda` :  
_Enabling MySQL 4-byte support_  
```mysql
    SET GLOBAL innodb_file_format=Barracuda;
    SELECT NAME, SPACE, FILE_FORMAT FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES WHERE NAME like "nextcloud%";
```

Select all DB for convert to baracuda format use the next command to generate commands :  
```mysql
    USE INFORMATION_SCHEMA;
    SELECT CONCAT("ALTER TABLE ", TABLE_SCHEMA,".", TABLE_NAME, " ROW_FORMAT=DYNAMIC;") AS MySQLCMD FROM TABLES WHERE TABLE_SCHEMA = "nextcloud";
```
**Use previous generated commands!**

Go to the next cloud dir :  
    `cd /var/www/nextcloud`

Enable the maintenance mode :  
    `sudo -u www-data php occ maintenance:mode --on`

Restart mariaDB to apply changes :   
    `sudo systemctl restart mariadb`

Change the format of nextcloud :  
    `ALTER DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`

Modif the mysql config to use `utf8mb` :  
```bash
    sudo -u www-data php occ config:system:set mysql.utf8mb4 --type boolean --value="true"
    sudo -u www-data php occ maintenance:mode --off
```

Disable maintenance mode :  
    `sudo -u www-data php occ maintenance:mode --off`

#### 3. Configuration du serveur ngnix
Mise en place de php pour ngnix :  
```bash
    sudo mkdir /etc/nginx/nginxconfig.io
    sudo nano /etc/nginx/nginxconfig.io/php_fastcgi.conf
```

Inserer les données de configuration pour php :  
```bash
    # 404
    try_files $fastcgi_script_name =404;

    # default fastcgi_params
    include fastcgi_params;

    # fastcgi settings
    fastcgi_pass            unix:/var/run/php/php7.2-fpm.sock;
    fastcgi_index            index.php;
    fastcgi_buffers            8 16k;
    fastcgi_buffer_size        32k;

    # fastcgi params
    fastcgi_param DOCUMENT_ROOT        $realpath_root;
    fastcgi_param SCRIPT_FILENAME    $realpath_root$fastcgi_script_name;
    fastcgi_param PHP_ADMIN_VALUE    "open_basedir=$base/:/usr/lib/php/:/tmp/";
```

Install php modules :   
```bash
    sudo apt-get install php7.2-mysql
    sudo apt-get install php7.2-zip
    sudo apt-get install php7.2-dom
    sudo apt-get install php7.2-mbstring
    sudo apt-get install php7.2-gd
    sudo apt-get install php7.2-curl
    sudo apt-get install php7.2-intl
    sudo apt-get install php7.2-imagick
```
#### 4. Configuration du client
créer le fichier nextcloud.conf :   
`sudo nano  /etc/nginx/sites-available/nextcloud.conf`

Insérer les données relative à notre site :  
```bash
    server {
        listen 192.168.1.54:80;
        listen [::]:80;
        root /var/www/nextcloud;

        # index.php
        index index.php;

        # index.php fallback
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        # handle .php
        location ~ \.php$ {
            include nginxconfig.io/php_fastcgi.conf;
    }
```

Créer un lien symbolique pour activer le site :  
`sudo ln -s /etc/nginx/sites-available/nextcloud.conf /etc/nginx/sites-enabled/`

Supprimer le site par défaut :  
`sudo rm /etc/nginx/sites-enabled/default`

### SSL

Install ssl :  
```bash
    sudo apt install letsencrypt
    systemctl restart nginx
```
### DNS

Install dns and tools :  
```bash
    sudo apt-get install -y bind9 bind9utils bind9-doc dnsutils
    sudo nano /etc/bind/named.conf.local
```