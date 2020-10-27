# install and configure new blank server

## - log in server : 
```
$ ssh-keygen -f "/home/bychrisme/.ssh/known_hosts" -R "ip_address"
$ ssh username@ip_address
```
    
## - add user account :
make sure to replace "username" by your own name you want, set password and user account name
``` 
$ adduser username    
$ usermod -aG sudo username     // add username on sudo group
$ su - username     // to switch on recently account created
```
    
## - disable root connection
```
$ sudo nano /etc/ssh/sshd_config      // set PermitRootLogin to no
$ sudo service ssh restart
```
    
## - change port connection :
find line with #port 22 and uncomment this field and set your port number, save and exit example port 2222
```
$ find / -name "sshd_config" 2>/dev/null  // to locate sshd_config
$ sudo nano /etc/ssh/sshd_config
```

## - set firewall to accept ssh port :
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 2222/tcp
$ sudo ufw enable
$ sudo ufw allow 80
$ sudo ufw allow 443
$ sudo ufw status verbose     // to check ufw status
```
    
## - install curl :
```
$ sudo apt update && sudo apt upgrade
$ sudo apt install curl
```

## - install PHP7.3 :
```
$ sudo apt upgrade
$ sudo apt-get install php
$ sudo apt-get install php-pear php-fpm php-dev php-zip php-curl php-xmlrpc php-gd php-mysql php-mbstring php-xml libapache2-mod-php php-intl php-bcmath php-cli -y
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt update
$ sudo apt install php7.3-fpm -y
$ sudo apt install php7.3 -y
$ sudo apt install php7.3-common php7.3-mysql php7.3-xml php7.3-xmlrpc php7.3-curl php7.3-gd php7.3-imagick php7.3-cli php7.3-dev php7.3-imap php7.3-mbstring php7.3 opcache php7.3-soap php7.3-zip php7.3-intl php7.3-bcmath -y
```
$ 
## - intall nginx : 
```
$ sudo apt install nginx
$ sudo ufw allow 'Nginx HTTP'
```
    
## - configure nginx to use php on sample domain name
```
$ sudo nano /etc/nginx/sites-available/example.conf
```
    
paste the following code inside example.conf
```    
    server {
        listen 80;
        listen [::]:80;
        server_name www.example.com;
        index index.html;
        root /var/www/html;

        charset utf-8;

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        access_log off;
        error_log  /var/log/nginx/example-error.log error;

        sendfile off;

        client_max_body_size 100m;

        location / {
            try_files $uri $uri/ /index.html$is_args$args;
        }

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/run/php/php7.3-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_intercept_errors off;
            fastcgi_buffer_size 16k;
            fastcgi_buffers 4 16k;

        }

        location ~ /\.ht {
                deny all;
        }
    }
```
    

* root is directory who is locate php project
* server_name is your domain name
* `get more information` on https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-ubuntu-18-04

```
$ sudo ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/    //to make symbolic link
$ sudo unlink /etc/nginx/sites-enabled/default    // to unlink default config file
$ sudo nginx -t   // check nginx status
```
`relaod nginx`, make sure that apache isn't install or any application use the port 80, if it's case fix it on https://stackoverflow.com/questions/35868976/nginx-job-for-nginx-service-failed-because-the-control-process-exited
```
$ sudo systemctl reload nginx 
```


    
## - install mysql
```
$ sudo apt install mysql-server
$ sudo mysql_secure_installation
```
to enter on mysql and configure mysql root connection
```
$ sudo mysql
```
```
> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';  // update user password for root
> FLUSH PRIVILEGES;
> exit
```
```
$ sudo mysql -u root -p    // log at root account with password
```
    
## - install composer
```
$ sudo apt install zip unzip
$ cd ~
$ curl -sS https://getcomposer.org/installer -o composer-setup.php
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
    
## - install git
```
$ sudo apt-get install git
```
    
## - install npm / node
```
    sudo apt-get install build-essential libssl-dev
```
> version used is `0.36.0`, check the last version on https://github.com/creationix/nvm
```
$ curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.36.0/install.sh -o install_nvm.sh
$ bash install_nvm.sh
$ source ~/.profile
$ nvm ls-remote       // check available version
$ nvm install 12.19.0
$ nvm use 12.19.0
$ nvm alias default 12.19.0
```
    
## - install yarn
```
$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
$ sudo apt-get update && sudo apt-get install yarn  
```