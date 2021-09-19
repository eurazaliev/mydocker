FROM php:7.4-fpm-alpine
RUN apk --update --no-cache add git bash mc mysql-client
RUN docker-php-ext-install pdo_mysql
COPY --from=composer /usr/bin/composer /usr/bin/composer
WORKDIR /var/www
#CMD composer install ; bin/console doctrine:migrations:migrate ;  php-fpm 
EXPOSE 9000