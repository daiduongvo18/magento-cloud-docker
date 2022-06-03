{%note%}
FROM php:{%version%}-cli

ARG COMPOSER_VERSION={%composer_version%}
ARG MAGENTO_ROOT=/app
ARG COMPOSER_ALLOW_SUPERUSER=1
ARG COMPOSER_HOME=/composer
ARG CRONTAB=""
ARG SONAR_SCANNER_HOME=/opt/sonar-scanner

ENV COMPOSER_MEMORY_LIMIT -1
ENV COMPOSER_ALLOW_SUPERUSER ${COMPOSER_ALLOW_SUPERUSER}
ENV COMPOSER_HOME ${COMPOSER_HOME}
ENV PHP_MEMORY_LIMIT -1
ENV PHP_VALIDATE_TIMESTAMPS 1
ENV DEBUG false
ENV UPDATE_UID_GID false
ENV MAGENTO_RUN_MODE production
ENV SENDMAIL_PATH /dev/null
ENV PHPRC ${MAGENTO_ROOT}/php.ini
ENV SONAR_SCANNER_HOME=${SONAR_SCANNER_HOME}
ENV SONAR_USER_HOME=${SONAR_SCANNER_HOME}/.sonar

{%env_php_extensions%}

# Configure Node.js version
RUN curl -sL https://deb.nodesource.com/setup_lts.x | bash

# Install dependencies
RUN apt-get update \
  && apt-get upgrade -y \
  && apt-get install -y --no-install-recommends \
  {%packages%} \
  && rm -rf /var/lib/apt/lists/*

# Install PyYAML
RUN pip3 install --upgrade setuptools \
    && pip3 install pyyaml

# Install Grunt
RUN npm install -g grunt-cli

# Install MailHog
RUN curl -L -O https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 \
    && sudo chmod +x mhsendmail_linux_amd64 \
    && sudo mv mhsendmail_linux_amd64 /usr/local/bin/mhsendmail

# Install Sonar Scanner
RUN curl -L -O https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747.zip \
    && unzip sonar-scanner-cli-4.7.0.2747.zip -d /opt \
    && mv /opt/sonar-scanner-4.7.0.2747 /opt/sonar-scanner \
    && chmod +x /opt/sonar-scanner/bin/sonar-scanner \
    && ln -sf /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner \
    && rm -f sonar-scanner-cli-4.7.0.2747.zip

# Configure the gd library
{%docker-php-ext-configure%}

# Install required PHP extensions
{%docker-php-ext-install%}

{%php-pecl-extensions%}

{%installation_scripts%}

ADD etc/php-cli.ini /usr/local/etc/php/conf.d/zz-magento.ini
ADD etc/php-xdebug.ini /usr/local/etc/php/conf.d/zz-xdebug-settings.ini
ADD etc/php-pcov.ini /usr/local/etc/php/conf.d/zz-pcov-settings.ini
ADD etc/mail.ini /usr/local/etc/php/conf.d/zz-mail.ini
ADD etc/php-gnupg.ini /usr/local/etc/php/conf.d/gnupg.ini

# Get composer installed to /usr/local/bin/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --version=${COMPOSER_VERSION} --filename=composer

ADD bin/* /usr/local/bin/

RUN groupadd -g 1000 www && useradd -g 1000 -u 1000 -d ${MAGENTO_ROOT} -s /bin/bash www

ADD docker-entrypoint.sh /docker-entrypoint.sh

RUN ["chmod", "+x", \
    "/docker-entrypoint.sh", \
    "/usr/local/bin/magento-installer", \
    "/usr/local/bin/magento-command", \
    "/usr/local/bin/mftf-command", \
    "/usr/local/bin/ece-command", \
    "/usr/local/bin/cloud-build", \
    "/usr/local/bin/cloud-deploy", \
    "/usr/local/bin/cloud-post-deploy", \
    "/usr/local/bin/run-cron", \
    "/usr/local/bin/run-hooks" \
]

{%volumes_cmd%}

{%volumes_def%}

run mkdir /root/.ssh

RUN chown -R www:www /usr/local /var/www /var/log /usr/local/etc/php/conf.d /etc/cron.d ${MAGENTO_ROOT} ${COMPOSER_HOME}
RUN if [[ ! -z "${CRONTAB}" ]]; then echo "${CRONTAB}" > /etc/cron.d/magento && touch /var/log/cron.log ; fi

ENTRYPOINT ["/docker-entrypoint.sh"]

WORKDIR ${MAGENTO_ROOT}

USER root

CMD ["bash"]
