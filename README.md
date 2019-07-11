# docker-compose_laradep
Laravel deployment using Docker Compose.

### First, let's setup our Laravel Application 
```
~$ laravel new laravel-app
~$ cd laravel-app
```

### Setting up a Ephimeral container to setup all the applications on the same image. 

```
~$ docker run --rm -v $(pwd):/App composer Install 
```
Now we have to create the docker-compose.yml 
```
~$ vim docker-compose.yml 
```
(vim for more pleasure...) 
We have to setup the file like this: 

```
version: '3'
services:

  #PHP Service
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: digitalocean.com/php
    container_name: app
    restart: unless-stopped
    tty: true
    environment:
      SERVICE_NAME: app
      SERVICE_TAGS: dev
    working_dir: /var/www
    volumes:
      - ./:/var/www
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - app-network

  #Nginx Service
  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    tty: true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      - app-network

  #MySQL Service
  db:
image: mysql:5.7.22
    container_name: db
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: admin123
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    volumes:
      - dbdata:/var/lib/mysql/
      - ./mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - app-network

#Docker Networks
networks:
  app-network:
    driver: bridge
#Volumes
volumes:
  dbdata:
    driver: local
                      
```


And next, whe have to setup the Dockerfile: 

```vim Dockerfile```
```
FROM php:7.2-fpm
  
# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    mysql-client \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl \
    nano

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
                                
```

Now whe have to create 3 specials dirs. 

```
~$ mkdir mysql php nginx

~$ vim mysql/my.cnf

```
Content 
```
    [mysqld]
    general_log = 1
    general_log_file = /var/lib/mysql/general.log
```

```
~$ vim php/local.ini 
```
Content
```
upload_max_filesize=40M
post_max_size=40M

```
```
~$ vim mysql/my.cnf 
```
Content 
```
    [mysqld]
    general_log = 1
    general_log_file = /var/lib/mysql/general.log

```
### Turning up the docker-compose service.

```
~$ docker-compose up -d 
```
### Troubleshooting 

You don't have to run nginx, mysql on your local machine because this will do a little issue with Docker-compose, if this happens just turn off the services.

```
sudo service mysql stop
sudo service nginx stop 
```

#### ShankyJS.      
