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
- Open terminal:
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