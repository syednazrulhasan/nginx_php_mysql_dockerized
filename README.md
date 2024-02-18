# Containerize Nginx/PHP/MySQL
This repo contains code that helps to containerize(dockerize) Nginx/PHP/MySQL that can be spun up with *docker-compose up -d* 
===================================

### Intro
Learn how to containerize(dockerize) Apache,PHP and MySQL quickly for testing or demo purposes using Docker Compose and Dockerfile. Deploy your application environment in moments with flexibility of PHP.ini and Apache.conf files, mirroring your development computer's structure with just one command 
```
docker-compose up -d
```

There are 6 simple files that you can clone from this repo's main branch

```
/nginx_php_mysql_dockerized/
.
├── docker-compose.yml
├── nginx
│ ├── default.conf
│ └── Dockerfile
├── php
│ ├── Dockerfile
│ └── php.ini
├── README.md
└── www
    └── html
        └── index.php
```

Once this structure is replicated next step is to install docker on server OS followed by docker-compose followed by running command ```docker-compose up -d``` to spin up the Nginx PHP MySQL as seperate docker containers on the host machine. If you replicate exact same structure it would also create a database with the name `kaveri` also create corresponding user and password associated with that database which can be found under index.php example below file. This combination of `dockerfile` and `docker-compose.yml` makes use of persistent volume for database filesysem and web site file systems. 


The following code attempts to connect to a MySQL database using the mysqli interface from PHP. There is `phpinfo()` at the bottom to show what possible values do php have on this system which is also overridden by `php.ini` in `www/html` folder

#### index.php
```
<!DOCTYPE html>  
     <head>  
      <title>Hello World!</title>
     </head>   

     <body>  
      <h1>Hello World!</h1>  
      <p><?php echo 'We are running PHP, version: ' . phpversion(); ?></p>  
      <?php  
       $database ="kaveri";  
       $user = "krishna";  
       $password = "godavari";  
       $host = "mysql";  


       $connection = new PDO("mysql:host={$host};dbname={$database};charset=utf8", $user, $password);  
       $query = $connection->query("SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE='BASE TABLE'");  
       $tables = $query->fetchAll(PDO::FETCH_COLUMN);  

        if (empty($tables)) {
          echo "<p>There are no tables in database \"{$database}\".</p>";
        } else {
          echo "<p>Database \"{$database}\" has the following tables:</p>";
          echo "<ul>";
            foreach ($tables as $table) {
              echo "<li>{$table}</li>";
            }
          echo "</ul>";
        }
        ?>

        <?php phpinfo(); ?>
    </body>
</html>
```

The following is a simple `docker-compose.yml` that facilitates the dockerization of Nginx/PHP/MySQL
#### `docker-compose.yml`
```
  version: "3.9"
     nginx:    
      build: ./nginx/  
      container_name: nginx-container  
      ports:  
       - 80:80  
      links:  
       - php  
      volumes_from:  
       - app-data  

     php:    
      build: ./php/  
      container_name: php-container  
      expose:  
       - 9000  
      links:  
       - mysql  
      volumes_from:  
       - app-data  

     app-data:    
      image: php:7.4-fpm  
      container_name: app-data-container  
      volumes:  
       - ./www/html/:/var/www/html/  
      command: "true"  

     mysql:    
      image: mysql:5.7  
      container_name: mysql-container
      restart: always
      ports:
      - "3306:3306"  
      volumes_from:  
       - mysql-data  
      environment:  
       MYSQL_ROOT_PASSWORD: CmKcCx  
       MYSQL_DATABASE: kaveri  
       MYSQL_USER: krishna  
       MYSQL_PASSWORD: godavari  

     mysql-data:    
      image: mysql:5.7  
      container_name: mysql-data-container  
      volumes:  
       - /var/lib/mysql  
      command: "true"
```

#### `nginx/Dockerfile` enable installation of latest nginx
```
FROM nginx:latest   
COPY ./default.conf /etc/nginx/conf.d/default.conf 
```

#### `php/Dockerfile` enable neccesary php extentions and custom user.ini file to overide php defaults
```
FROM php:7.4-fpm

# Update packages and install necessary packages and PHP extensions
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
        curl \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
        libzip-dev \
        zip \
        unzip

RUN docker-php-ext-install \
    mysqli \
    gd \
    pdo_mysql \
    zip

# Copy custom php.ini file
COPY ./php.ini /usr/local/etc/php/conf.d/custom.ini
```

#### `nginx/default.conf`  enables setup of virtual host or implementaion of SSL 
```
server {  

     listen 80 default_server;  
     #server_name example.com;
     root /var/www/html;  
     index index.html index.php;  

     charset utf-8;  

     location / {  
      try_files $uri $uri/ /index.php?$query_string;  
     }  

     location = /favicon.ico { access_log off; log_not_found off; }  
     location = /robots.txt { access_log off; log_not_found off; }  

     access_log off;  
     error_log /var/log/nginx/error.log error;  

     sendfile off;  

     client_max_body_size 100m;  

     location ~ .php$ {  
      fastcgi_split_path_info ^(.+.php)(/.+)$;  
      fastcgi_pass php:9000;  
      fastcgi_index index.php;  
      include fastcgi_params;  
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  
      fastcgi_intercept_errors off;  
      fastcgi_buffer_size 16k;  
      fastcgi_buffers 4 16k;  
    }  

     location ~ /.ht {  
      deny all;  
     }  
    } 
```

### Volumes
We have demonstrated use of persistent volume in case your container crashes your data is safe in persistent volumes but make sure to keep everything backup **there is no best alternate ever to backups so please retain them before messing around with anything**

### Demonstration of docker-compose up!

Step 1 : Pull latest updates of OS

```
sudo apt-get update
```

Step 2: Install docker by following below steps

```
sudo apt install ca-certificates curl gnupg lsb-release
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Step 3: Install all required software packkages

```
sudo apt install  docker.io unzip mysql-client-core-8.0
```

Step 4: Chek the docker version to ensure docker is installed
```
docker --version
```

Step 5: Start and enable the docker

```
sudo systemctl start docker
sudo systemctl enable docker
```

Step 6: Install docker compose by following the below commands.
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```
sudo chmod +x /usr/local/bin/docker-compose
```

Step 7: Verify docker compose version
```
docker-compose --version
```

Step 8: Clone the setup files from git
```
git clone https://github.com/syednazrulhasan/apache_php_mysql_dockerized.git
```

Step 9: cd into the cloned directory and run following command
```
docker-compose up -d
```
Step 10 Run following to update user anf gropup of html folder
```
chown -R ubuntu:www-data  www/html
```


### Docker Commands

To stop all running containers
```
docker stop $(docker ps -a -q)
```

To remove all containers that are stopped
```
docker rm -f $(docker ps -a -q)
```

To remove images that are not in use
```
docker rmi -f $(docker images -q)
```

To clear system of all cluttered containers
```
docker system prune -a
```





