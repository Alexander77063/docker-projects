
# Use Amazon Linux 2023 as the base image
FROM amazonlinux:2023
# 1. Install PHP and dependencies with corrected package names
RUN dnf install -y \
    httpd \
    php \
    php-common \
    php-cli \
    php-curl \
    php-mbstring \
    php-gd \
    php-mysqlnd \
    php-json \
    php-xml \
    php-fpm \
    php-intl \
    php-zip \
    wget \
    git \
    unzip \
    mariadb-connector-c \
    && dnf clean all

# 2. Configure MySQL repository (EL9 compatible)
RUN wget https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm \
    && rpm -ivh mysql80-community-release-el9-5.noarch.rpm \
    && rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023 \
    && rm -f mysql80-community-release-el9-5.noarch.rpm

# Set working directory
WORKDIR /var/www/html

# Define build arguments
ARG PERSONAL_ACCESS_TOKEN
ARG GITHUB_USERNAME
ARG REPOSITORY_NAME
ARG WEB_FILE_ZIP
ARG WEB_FILE_UNZIP
ARG DOMAIN_NAME
ARG RDS_ENDPOINT
ARG RDS_DB_NAME
ARG RDS_MASTER_USERNAME
ARG RDS_DB_PASSWORD

# Set environment variables
ENV PERSONAL_ACCESS_TOKEN=$PERSONAL_ACCESS_TOKEN \
    GITHUB_USERNAME=$GITHUB_USERNAME \
    REPOSITORY_NAME=$REPOSITORY_NAME \
    WEB_FILE_ZIP=$WEB_FILE_ZIP \
    WEB_FILE_UNZIP=$WEB_FILE_UNZIP \
    DOMAIN_NAME=$DOMAIN_NAME \
    RDS_ENDPOINT=$RDS_ENDPOINT \
    RDS_DB_NAME=$RDS_DB_NAME \
    RDS_MASTER_USERNAME=$RDS_MASTER_USERNAME \
    RDS_DB_PASSWORD=$RDS_DB_PASSWORD

# Clone repository and set up files
RUN git clone https://$PERSONAL_ACCESS_TOKEN@github.com/$GITHUB_USERNAME/$REPOSITORY_NAME.git \
    && unzip $REPOSITORY_NAME/$WEB_FILE_ZIP -d $REPOSITORY_NAME/ \
    && cp -av $REPOSITORY_NAME/$WEB_FILE_UNZIP/. /var/www/html \
    && rm -rf $REPOSITORY_NAME

# Configure permissions and .env file
RUN chmod -R 777 /var/www/html storage/ \
    && sed -i \
    -e '/^APP_ENV=/ s/=.*$/=production/' \
    -e "/^APP_URL=/ s/=.*$/=https:\/\/$DOMAIN_NAME\//" \
    -e "/^DB_HOST=/ s/=.*$/=$RDS_ENDPOINT/" \
    -e "/^DB_DATABASE=/ s/=.*$/=$RDS_DB_NAME/" \
    -e "/^DB_USERNAME=/ s/=.*$/=$RDS_MASTER_USERNAME/" \
    -e "/^DB_PASSWORD=/ s/=.*$/=$RDS_DB_PASSWORD/" \
    .env

# Apache configuration
RUN sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Copy custom provider
COPY AppServiceProvider.php app/Providers/AppServiceProvider.php

# Expose and start Apache
EXPOSE 80
ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]