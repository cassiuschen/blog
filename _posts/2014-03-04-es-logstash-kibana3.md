---
layout: post

title: "ES + Logstash + Kibana3 日志分析系统部署"
cover_image: es.jpg
tags:
- programming
- unix
- log

excerpt: ""

author:
  name: 陈小紫 Cassius Chen
  twitter: cassiuschen
  bio: Ruby工程师，摄影设计爱好者.
  image: lg.png
---

Kibana3是一个基于Node.js的Web端数据处理终端，其2.x版本曾经是Rails搭建的，后在Node上重构，整个程序也变得轻量化起来。

Logstash是Java写的优秀的日志采集系统，通过写Json文件配置其采集信息。

ES是Elastic Search的简称，是一个语义化搜索框架，在这套日志分析系统中使用ES作为数据检索及过滤。

这三件全部都属于开源软件，有兴趣的可以在Github上找到源码
我自己用这套日志系统用了几个月，期间没有崩溃过，非常稳定。在此介绍一下它的部署方式。

> 实例环境：OS: Ubuntu 12.04.4 LTS (amd64)

###1. 安装JRE

 因为Logstash是Java程序，所以必不可少的需要Java Runtime。

 首先下载jre：[在此下载](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

 然后解压：
{% highlight sh %}
     sudo tar -zxvf jdk-7#{你下载的版本号}-linux-x64.tar.gz
{% endhighlight %}
 接下来配置环境变量：（我的版本是1.7.0_51-b13，故下列以此为例）
{% highlight sh %}
     sudo vim /etc/profile
{% endhighlight %}
 在/etc/prifile文件后面添加(提示：这里JDK的路径改为你自己的安装路径):
{% highlight sh %}
     export JAVA_HOME=/usr/jdk1.7.0_51-b13
     export JRE_HOME=/usr/jdk1.7.0_51-b13/jre
     export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
     export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
{% endhighlight %}
 使/etc/profile文件立刻生效
{% highlight sh %}
     . /etc/profile
{% endhighlight %}
 修改系统默认JDK，并使之立马生效:
{% highlight sh %}
     sudo update-alternatives --install "/usr/bin/java" "java" "/usr/jdk1.7.0_51-b13/bin/java" 300
     sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/jdk1.7.0_51-b13/bin/javac" 300
     sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/jdk1.7.0_51-b13/bin/javaws" 300
     sudo update-alternatives --config java
     sudo update-alternatives --config javac
     sudo update-alternatives --config javaws
{% endhighlight %}
 确定确实安装成功：
{% highlight sh %}
     $ java -version
     java version "1.7.0_51"
     Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
     Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
{% endhighlight %}
 至此，安装JRE成功

###2. 安装ES

ES默认会占用9200端口，请特别留意一下。

下载：[官方地址](https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.0.1.deb)

安装：
{% highlight sh %}
     sudo dpkg -i elasticsearch-1.0.1.deb
{% endhighlight %}
 enjoy~

###3. 安装Logstash

下载：[官方地址](http://download.elasticsearch.org/logstash/logstash/logstash-1.3.3-flatjar.jar)

配置，我按照我的习惯把文件放在了/etc/logstash/local.conf，你可以自行改动：
{% highlight ruby %}
input {
    file {
       type => "linux-syslog"
       path => [ "/var/log/*.log" ]
   }
   file {
      type => "nginx"
      path => "/var/log/nginx/*.log"
   }
   file {
      type => "postgresql"
      path => "/var/log/postgresql/*.log"
   }
   file {
      type => "mongodb"
      path => "/var/log/mongodb/*.log"
   }
   file {
      type => "redis"
      path => "/var/log/redis/*.log"
   }
   file {
      type => "writings"
      path => "/var/mirrors/writing/log/*.log"
   }
   udp {
      type => "access_log"
      port => 3333
   }
}
output {
   stdout {
      debug => true
      debug_format => json
   }
   elasticsearch {
      embedded => true
   }
}
{% endhighlight %}
以上是我的部分配置，按照这样的格式就可以。网上有在ES与Logstash中间使用Redis做储存的，我没有这么用，因为日志量没有到足以阻塞磁盘IO的地步，如果感兴趣可以自己搜来看看。

运行：
{% highlight sh %}
java -jar logstash.jar agent -f /etc/logstash/local.conf
{% endhighlight %}
###4.安装Kibana3

因为更新频率比较快，所以推荐直接clone下来源码：
{% highlight sh %}
git clone https://github.com/elasticsearch/kibana
{% endhighlight %}
如果ES不在本机，可以修改config.js: 将其中的 http://localhost:9200 改成你ES所在的地址。

从此可以看出，Kinana这三件套都是相对独立的，可以分别在不同的服务器上跑，只要修改地址即可。

安装依赖：
{% highlight sh %}
sudo bundle
sudo npm install
{% endhighlight %}
运行：

node server.js

到此，全部完成。

可以设置开机启动：在crontab -e中加入下列：
{% highlight sh %}
@reboot java -jar /usr/local/bin/logstash/logstash.jar agent -f /etc/logstash/local.conf
@reboot cd /var/info && node server.js
{% endhighlight %}
上述地址改成自己的即可。
