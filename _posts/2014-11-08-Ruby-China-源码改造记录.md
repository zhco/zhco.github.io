---
layout: post
title: "Ruby China 源码改造记录"
published: True
categories: jekyll update
date: 2014-11-08 20:32:16
---

### 环境依赖
- Ruby 2.1.3 +
- Memcached 1.4 +
- Redis 2.2 +
- MongoDb 2.4.4 +
- ImageMagick 6.5+
- libpng
- JavaScript Runtime

## 安装RVM & Ruby

安装RVM，网上很多方法有BUG

{% highlight bash %}
$ \curl -sSL https://get.rvm.io | bash
{% endhighlight %}

使rvm在terminal中生效，不用重启终端，下面的代码取决于你的rvm安装目录

{% highlight bash %}
$ source ~/.rvm/scripts/rvm
{% endhighlight %}

用rvm安装ruby 2.1.4版本，并控制版本，这样会自动安装依赖包

{% highlight bash %}
$ rvm use --install --default 2.1.4
{% endhighlight %}

## 安装Javascript Runtime
如果服务器有安装`NodeJS`，这一步可略过

{% highlight bash %}
$ sudo apt-get install nodejs
{% endhighlight %}

## 网站初始化安装
在网站根目录执行

{% highlight bash %}
$ ./bin/setup
{% endhighlight %}

如果还有缺失的gem包，可以用如下命令安装

{% highlight bash %}
$ bundle install
{% endhighlight %}

## 安装ImageMagick
图片处理的工具ImageMagick，与上传头像然后进行裁剪相关，依赖图片库libpng

{% highlight bash %}
$ sudo apt-get install libpng-dev
{% endhighlight %}

{% highlight bash %}
$ sudo apt-get install imagemagick
{% endhighlight %}

## ruby-china源码修改

#### 配置文件
config/config.yml

## Nginx配置

在Nginx的sites-available目录下新建配置文件your.site.domain，然后在文件中配置如下

{% highlight text %}
upstream ruby-china {  
    server 127.0.0.1:7000;  
    server 127.0.0.1:7001;  
}  
server {  
    listen       80;
    server_name your.site.domain;  
   
    access_log /var/www/ruby-china/log/access.log;  
    error_log  /var/www/ruby-china/log/error.log;  
    root       /var/www/ruby-china/public;  
    index      index.html;
    client_max_body_size 5m;
   
    location / {  
      proxy_redirect     off;
      proxy_set_header   Host $host;
      proxy_set_header   X-Forwarded-Host $host;
      proxy_set_header   X-Forwarded-Server $host;
      proxy_set_header   X-Real-IP        $remote_addr;
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      proxy_buffering    on;
      proxy_pass         http://ruby-china;
    }  

    location ~* ^(/assets|/favicon.ico|/.*\.(png|jpg|ico|gif|jpeg)) {
      expires max;
      access_log  off;
    }
}
{% endhighlight %}

然后在sites-enabled目录下创建这个配置文件的软链接，注意软链接的路径，不然会无效

{% highlight bash %}
$ sudo ln -s your.site.domain /etc/nginx/sites-available/your.site.domain
{% endhighlight %}

然后重新载入nginx配置

{% highlight bash %}
$ nginx -s reload
{% endhighlight %}

## 启动服务

启动app server，在根目录下，保证memcached和redis都已经开启

{% highlight bash %}
$ bundler exec thin start -C config/thin.yml
{% endhighlight %}

开启sidekiq服务，注意配置环境为你的服务器环境development或者test或者production，要配置一个log文件，以deamon进程运行，不然终端废掉了，ctrl + c都停止不了

{% highlight bash %}
$ bundler exec sidekiq -C config/sidekiq.yml -e production -d -L log/sidekiq.log
{% endhighlight %}

开启faye服务，开启全站消息通知

{% highlight bash %}
$ start_faye_server
{% endhighlight %}