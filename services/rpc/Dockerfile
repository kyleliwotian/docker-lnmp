FROM php:7.2-cli
MAINTAINER qingkong

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN cat /etc/issue

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
ADD sources.list /etc/apt/
RUN cat /etc/apt/sources.list
# 更新安装依赖包和PHP核心拓展
RUN apt-get update && apt-get install -y automake bison flex g++ git libboost-all-dev libevent-dev libssl-dev libtool make pkg-config wget 
		
# 安装 thrift
RUN wget http://mirrors.tuna.tsinghua.edu.cn/apache/thrift/0.11.0/thrift-0.11.0.tar.gz \
    && tar -xzvf thrift-0.11.0.tar.gz \
    && cd thrift-0.11.0 && ./configure && make && make install

RUN apt-get install -y git vim procps curl libfreetype6-dev libjpeg62-turbo-dev libpng-dev libmcrypt-dev 

# 安装PHP扩展    
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	&& docker-php-ext-install -j$(nproc) gd \
        && docker-php-ext-install zip \
        && docker-php-ext-install pdo_mysql \
        && docker-php-ext-install opcache \
        && docker-php-ext-install mysqli \
        && rm -r /var/lib/apt/lists/*

# 将预先下载好的拓展包从宿主机拷贝进去
COPY ./pkg/redis.tgz /home/redis.tgz
COPY ./pkg/libmemcached.tar.gz /home/libmemcached.tar.gz
COPY ./pkg/memcached.tgz /home/memcached.tgz
COPY ./pkg/mcrypt-1.0.1.tgz /home/mcrypt-1.0.1.tgz

# pecl 安装mcrypt扩展，兼容老代码
RUN pecl install /home/mcrypt-1.0.1.tgz && echo "extension=mcrypt.so" > /usr/local/etc/php/conf.d/mcrypt.ini

# pecl 安装 Redis
RUN pecl install /home/redis.tgz && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini

# 安装libmemcached
RUN tar zxvf /home/libmemcached.tar.gz \
    && cd libmemcached-1.0.18 \
    && ./configure --prefix=/usr/local/libmemcached --with-memcached \
    && make && make install

# 安装memcached
RUN tar zxvf /home/memcached.tgz \
    && cd memcached-3.0.4 \
    && phpize \
    && ./configure -enable-memcached -with-php-config=/usr/local/bin/php-config -with-zlib-dir -with-libmemcached-dir=/usr/local/libmemcached -prefix=/usr/local/phpmemcached -disable-memcached-sasl \
    && make && make install \
    && echo "extension=memcached.so" > /usr/local/etc/php/conf.d/memcached.ini

# 安装 Swoole
RUN cd /home \
    && git clone https://gitee.com/swoole/swoole.git \
    && cd swoole \
    && phpize \
    && ./configure --enable-openssl -with-php-config=/usr/local/bin/php-config \
    && make \
    && make install \
    && echo "extension=swoole.so" > /usr/local/etc/php/conf.d/swoole.ini

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH


# 删除扩展包
RUN rm -f /home/redis.tgz
RUN rm -f /home/libmemcached.tar.gz
RUN rm -f /home/memcached.tgz
RUN rm -f /home/mcrypt-1.0.1.tgz

RUN rm -f /thrift-0.11.0.tar.gz

WORKDIR /data

# 写权限
RUN usermod -u 1000 www-data
