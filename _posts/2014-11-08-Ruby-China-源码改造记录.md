---
layout: post
title: Ruby China 源码改造记录
published: True
categories: jekyll update
tags: []
---

### 安装环境
- Ruby 2.1.3 +
- Memcached 1.4 +
- Redis 2.2 +
- MongoDb 2.4.4 +
- ImageMagick 6.5+
- libpng
- javascript runtime

## 启动服务

启动app server，在根目录下，保证memcached和redis都已经开启

{% highlight shell%}
$ bundler exec thin start -C config/thin.yml
{% endhighlight %}

开启sidekiq服务

{% highlight shell%}
$ bundler exec sidekiq -C config/sidekiq.yml -e production -d -L log/sidekiq.log
{% endhighlight %}

开启faye服务，开启全站消息通知

{% highlight shell%}
$ start_faye_server
{% endhighlight %}


## Nginx配置

## 安装RVM & Ruby

安装RVM，网上很多方法有问题

{% highlight shell%}
$ \curl -sSL https://get.rvm.io | bash
{% endhighlight %}


使rvm生效，下面的代码取决于你的rvm安装目录

{% highlight shell%}
$ source ~/.rvm/scripts/rvm
{% endhighlight %}

用rvm安装ruby 2.1.4，并控制版本，这样会自动安装依赖包

{% highlight shell%}
$ rvm use --install --default 2.1.4
{% endhighlight %}


## 安装ImageMagick
图片处理的工具

## ruby-china源码修改