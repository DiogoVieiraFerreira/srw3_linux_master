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
Install:
`apt-get install mariadb-server -y`