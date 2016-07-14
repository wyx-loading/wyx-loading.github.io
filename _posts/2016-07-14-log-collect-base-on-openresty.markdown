---
layout: post
title:  利用OpenResty实现简单的前端反馈信息收集
date:   2016-07-14 15:06:00 +0800
categories: ['blog', 'tools']
---

## 简单说一说

A: 哎呀，有必要收集前端的报错信息来分析一下程序的bug啊，不是每个用户都会看控制台报错的，不是每个终端都可以打开控制台的。  
B: 好啊，设置埋点在抛异常的地方，通过http推送到后端servlet就可以了。  
C: 额。感觉这东西很简单啊，但是还需要后端的工作时间就不太爽了，最好前端可以一手包办。

## [OpenResty][OpenResty cn]

> OpenResty 是一个基于 NGINX 和 LuaJIT 的 Web 平台。
> OpenResty ™ 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。

想到一个开源项目OpenResty，应该可以通过简单的nginx配置实现这个功能。前端大神对nginx的配置都比较熟悉，可以！！！  
关于OpenResty的巧妙之处，请查看其具体原理，这里就不多说了。  
由于nginx的高性能和lua的高性能结合，性能不会差，还有就是配置简单。

## 动手

直接上码。

{% highlight nginx %}
http {

    log_format tick "$msec^A$remote_addr^A$u_domain^A$u_url^A$u_title^A$u_referrer^A$u_sh^A$u_sw^A$u_cd^A$u_lang^A$http_user_agent";

    server {
        listen       9099;

        location /warn.gif {
        default_type image/gif;
        access_log off;

        access_by_lua "
        if ngx.var.arg_domain then
            ngx.location.capture('/i-log?' .. ngx.var.args)
        end
        ";

        add_header Expires "Fri, 01 Jan 1980 00:00:00 GMT";
        add_header Pragma "no-cache";
        add_header Cache-Control "no-cache, max-age=0, must-revalidate";

        empty_gif;
  }

        location /i-log {
            internal;

            set_unescape_uri $u_domain $arg_domain;
            set_unescape_uri $u_url $arg_url;
            set_unescape_uri $u_title $arg_title;
            set_unescape_uri $u_referrer $arg_referrer;
            set_unescape_uri $u_sh $arg_sh;
            set_unescape_uri $u_sw $arg_sw;
            set_unescape_uri $u_cd $arg_cd;
            set_unescape_uri $u_lang $arg_lang;

            log_subrequest on;
            access_log /mydata/server/log_collect/logs/collect.log tick;
            # access_log C:/Users/wuyuxiang/Desktop/openresty/openresty-1.9.7.3-win32/log_collect_2/logs/myex.log tick buffer=32k;

             echo '';
        }

    }

}
{% endhighlight %}

解释一下

`log_format tick ....`
（声明一个log格式，nginx就有的语法。）

埋点是warn.gif，留意到有设置为no-cache，这样可以避免浏览器缓存导致收集不到数据。

```
access_by_lua "
     if ngx.var.arg_domain then
          ngx.location.capture('/i-log?' .. ngx.var.args)
     end
";
```

这块就是openresty的巧妙了。access_by_lua是定义的lua代码block
里面很简单，判断是否有domain这个参数传上来，如果有，将请求转到/i-log，/i-log为内部接口，多加了一层验证。

接下来就是/i-log中将请求参数转为log_format的参数，在写到access_log中，开头有关闭access_log。

`log_subrequest on;`

这个可以打开log，如果不加上这句，会写不到log。

好了，基本就是这样，就可以快速实现前端的需求。总体来说，在这个需求上看，比servlet快捷多了，性能上来看，基本就是nginx vs tomcat等容器的性能对比。

---

<br/>
关于输出的日志的管理，nginx的access_log会无限扩大，我们想要每天分一个文件，写个定时任务实现。

{% highlight console %}
common_setting.sh
nginxPath=/openresty/nginx
nginxBin=$nginxPath/sbin/nginx
logCollectDir=/log_collect
confDir=$logCollectDir/conf/nginx.conf
pidfile=$logCollectDir/logs/nginx.pid
cut_log.sh
source /log_collect/common_setting.sh
time=`date +%Y%m%d`

mv $logCollectDir/logs/collect.log $logCollectDir/logs/collect-$time.log
kill -USR1 `cat $pidfile`
{% endhighlight %}

逻辑就是将log文件改名，加上日期后缀，再根据nginx进程ID发送USR1信号，以重新打开日志文件。利用crontab设置执行任务周期即可。


[OpenResty cn]: https://openresty.org/cn/

