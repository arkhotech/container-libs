FROM php:7.0.29-fpm
RUN apt-get update -y \
  && apt-get install -y \
    libxml2-dev \
  && apt-get clean -y \
  && docker-php-ext-install mysqli && docker-php-ext-install soap
RUN apt-get update \
   && apt-get -y install \
           libmagickwand-dev \
       --no-install-recommends \
   && pecl install imagick \
   && docker-php-ext-enable imagick \
   && rm -r /var/lib/apt/lists/*
RUN apt-get update \
  && apt-get -y install python-qt4 libqt4-webkit xvfb \
  && apt-get -y install git-core \
  && apt-get -y install python-pip \
  && pip install webkit2png
RUN mkdir python-webkit2png \
  && git clone https://github.com/adamn/python-webkit2png.git python-webkit2png \
  && cd python-webkit2png \
  && python ./setup.py install
RUN touch /var/log/webkit.log \
  && chown www-data:www-data /var/log/webkit.log \
  && chmod 655 /var/log/webkit.log
RUN apt-get -y install mcrypt libmcrypt-dev \
   && docker-php-ext-install mcrypt \
   && docker-php-ext-enable mcrypt
