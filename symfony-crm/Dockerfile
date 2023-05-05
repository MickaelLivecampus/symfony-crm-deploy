################################################# Builder
FROM php:fpm-alpine AS builder

# Ajouter git et yarn
RUN apk --update --no-cache add git && apk add yarn

# Install pdo_mysql
RUN docker-php-ext-install pdo_mysql

# Copie composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Change de répértoire
WORKDIR /app

# Copie les fichiers de l'application Symfony dans le conteneur
RUN git clone https://github.com/Factors25/symfony-crm.git

# Change de répértoire
WORKDIR /app/symfony-crm

# Définit les variables d'environnement pour Symfony
ENV COMPOSER_ALLOW_SUPERUSER=1

# Copie .env >.env.local
RUN cp .env .env.local

# Réécris la varaible DATABASE_URL avec la variable d'environnement
RUN sed -ri -e "s%DATABASE_URL=.*%DATABASE_URL=mysql:\/\/\$MYSQL_USER:\$MYSQL_PASSWORD@db:3306\/\$MYSQL_DATABASE%" .env.local

# Installe les dépendances front-end avec Yarn
RUN yarn install
# Construit les fichiers front-end avec Yarn
RUN yarn run build

# Install les dépendances de composer
RUN composer install
RUN composer require symfony/apache-pack

################################################# PROD
FROM php:8.1-apache as prod

RUN apt-get update && \
    apt-get install -y libicu-dev libzip-dev libpq-dev unzip git && \ 
    docker-php-ext-install intl pdo pdo_mysql zip && \
    rm -rf /var/lib/apt/lists/*

# Configure le répertoire racine d'Apache pour l'application Symfony
ENV APACHE_DOCUMENT_ROOT /app/symfony-crm/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# Copie le répertoire depuis le builder
COPY --from=builder /app/symfony-crm /app/symfony-crm/

# Copie le htaccess
COPY ./symfony-crm/.htaccess /app/symfony-crm/public/

# Mets les droits
RUN chown -R www-data:www-data /app/symfony-crm
RUN chmod 777 -R /app/symfony-crm/
USER www-data

# RUN !
CMD ["apache2-foreground"]