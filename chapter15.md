# Nginx拷贝流量

利用Ngnix将线上流量拷贝到测试机（一个http请求视为一份流量）

开发环境：CentOS 6.4

nginx安装目录：/usr/local/nginx-1.4.2

各模块安装包下载目录：/data/nginxinstall

本方法的局限性：模拟的多线程需要都返回时，才返回最终结果，因此测试线程延迟会影响线上。



1. 安装pcre

 \#yum -y install pcre pcre-devel





2. 安装zlib

yum -y install zlib zlib-devel





3. 安装LuaJIT

\# cd /data/nginxinstall

\# wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz

\# tar -xzvf LuaJIT-2.0.2.tar.gz

\# cd LuaJIT-2.0.2

\# make

lib和include是直接放在/usr/local/lib和usr/local/include

配置环境变量

export LUAJIT\_LIB=/usr/local/lib

export LUAJIT\_INC=/usr/local/include/luajit-2.0

export LD\_LIBRARY\_PATH=/usr/local/lib:$LD\_LIBRARY\_PATH



4. 下载准备httpLuaModule

\# cd /data/nginxinstall

\# wget https://github.com/chaoslawful/lua-nginx-module/archive/v0.8.6.tar.gz

\# tar -xzvf v0.8.6



5. 下面开始安装Nginx\(尝试采用nginx1.10.0遇到编译有错误\)：



\# cd /data/nginxinstall/

\# wget http://nginx.org/download/nginx-1.4.2.tar.gz 

\# tar -xzvf nginx-1.4.2.tar.gz

\# cd nginx-1.4.2

\# ./configure --prefix=/usr/local/nginx-1.4.2 --add-module=../lua-nginx-module-0.8.6

\# make -j2

\# make install



6. 编辑nginx/conf/nginx.conf文件（本文件是一个json格式，可以看做一棵树）：

在http结点下添加：

    upstream online {

         server  10.10.12.7:80;

    }



    upstream test {

         server  10.10.10.103:80;

    }

在server结点下添加

location ~\* ^/antispam {

    client\_body\_buffer\_size 2m;

    set $svr     "on";               \#开启或关闭copy功能

    content\_by\_lua\_file   copy\_flow.lua;

}



location ~\* ^/online {

     log\_subrequest on;

     rewrite ^/s1\(.\*\)$ $1 break;

     proxy\_pass http://s1; -- 反向代理，跳转到upstream online

}



location ~\* ^/test {

    log\_subrequest on;

    rewrite ^/test\(.\*\)$ $1 break;

    proxy\_pass http://test; -- 反向代理，跳转到upstream test

}





7. 编辑nginx/copy\_flow.lua 文件：

local online, test, action

action = ngx.var.request\_method



if action == "POST" then

    ngx.req.read\_body\(\) -- 解析 body 参数之前一定要先读取 body

    local arg = ngx.req.get\_post\_args\(\) -- post需要参数传递

    arry = {method = ngx.HTTP\_POST, body = ngx.var.request\_body, args=arg}

else

    arry = {method = ngx.HTTP\_GET}

end



if ngx.var.svr == "on" then

online, test = ngx.location.capture\_multi {

   { "/online" .. ngx.var.request\_uri, arry},

   { "/test" .. ngx.var.request\_uri, arry},

}

else

    online = ngx.location.capture\_multi {

       { "/online" .. ngx.var.request\_uri, arry}

    }

end



if online.status == ngx.HTTP\_OK then -- 只返回online server的结果

ngx.say\(online.body\)

else

ngx.status = ngx.HTTP\_NOT\_FOUND

end







8. POST参数中有中文，编码问题：

Nginx配置UTF8 ： 在server节点下配置 charset utf-8;

tomcat配置URL编码格式： &lt;Connector port="8080"  protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" /&gt;





9. test 出现延迟，从而Lua模拟出的线程出现延迟，影响线上, 需要添加如下参数：

location ~\* ^/test {

  proxy\_connect\_timeout 100ms;

  proxy\_read\_timeout    100ms;

  proxy\_send\_timeout    100ms;



  proxy\_pass http://test;

}

