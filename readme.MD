# install and configure new blank server with Nginx server

## - log in server : 
```
$ ssh-keygen -t rsa
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
$ find / -name "sshd_config" 2>/dev/null      // to locate sshd_config
$ sudo nano /etc/ssh/sshd_config              // set PermitRootLogin to no
$ sudo service ssh restart
```
    
## - change port connection :
find line with #port 22 and uncomment this field and set your port number, save and exit example port 2222
```
$ sudo nano /etc/ssh/sshd_config
```

## - set firewall to accept ssh port :
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 2222/tcp       // 2222 is your new port
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

## - install PHP8.1 :
### on Ubuntu 22.04
```
$ sudo apt update
$ sudo apt install --no-install-recommends php8.1
$ sudo apt-get install -y php8.1-cli php8.1-common php8.1-mysql php8.1-zip php8.1-gd php8.1-mbstring php8.1-curl php8.1-xml php8.1-bcmath

```
## - install PHP7.4 :
### on Ubuntu 20.04
```
$ sudo apt update
$ sudo apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath

```
### on Ubuntu 18.04/16.04
```
$ sudo apt-get update
$ sudo apt -y install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt-get update
$ sudo apt -y install php7.4
$ sudo apt-get install -y php7.4-{bcmath,bz2,intl,gd,mbstring,mysql,zip,common}
```
```
$ sudo php-fpm7.4 -t 
$ sudo service php7.4-fpm restart
```
## - intall nginx : 
```
$ sudo service apache2 stop
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

## - install phpmyadmin
```
$ mysql -u root -p

> create database phpmyadmin;
> CREATE USER 'phpmyadmin'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
> GRANT ALL PRIVILEGES ON phpmyadmin.* TO 'phpmyadmin'@'localhost';
> FLUSH PRIVILEGES;
> exit

$ sudo apt install phpmyadmin  // Choose none of web server proposed
```  
create phpmyadmin file inside /etc/nginx/snippets
```
$ cd /etc/nginx/snippets
$ sudo nano phpmyadmin
```
Paste the following code
```
location /phpmyadmin {
    root /usr/share/;
    index index.php index.html index.htm;
    location ~ ^/phpmyadmin/(.+\.php)$ {
        try_files $uri =404;
        root /usr/share/;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include /etc/nginx/fastcgi_params;
    }

    location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
        root /usr/share/;
    }
}
```
Create Nginx config file for phpmyadmin url
```
$ cd /etc/nginx/sites-available
$ sudo nano phpmyadmin.conf
```
Paste the following code
```
server {
        listen 80;
        server_name phpmyadmin.example.com www.phpmyadmin.example.com;

        root /usr/share/phpmyadmin;
        index index.html index.php;

        charset utf-8;

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        access_log off;
        error_log  /var/log/nginx/phpmyadmin-error.log error;

        sendfile off;

        client_max_body_size 100m;

        location / {
           try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {      
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/run/php/php8.1-fpm.sock;
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
Create symbolic link and restart Nginx
```
$ sudo ln -s /etc/nginx/sites-available/phpmyadmin.conf /etc/nginx/sites-enabled/
$ sudo service nginx restart
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

## - install Let's encrypt
```
$ curl -o- https://raw.githubusercontent.com/vinyll/certbot-install/master/install.sh | bash
$ certbot --version
$ sudo certbot --nginx -d example.com -d www.example.com
```
