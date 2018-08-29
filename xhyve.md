# ngx\_lua构建高并发应用

1. Memcached

        在Nginx中访问Memcached需要模块的支持，这里选用HttpMemcModule，这个模块可以与后端的Memcached进行非阻塞的通信。我们知道官方提供了Memcached，这个模块只支持get操作，而Memc支持大部分Memcached的命令。



        Memc模块采用入口变量作为参数进行传递，所有以$memc\_为前缀的变量都是Memc的入口变量。memc\_pass指向后端的Memcached Server。



  配置：



\#使用HttpMemcModule

location = /memc {

	set $memc\_cmd $arg\_cmd;

	set $memc\_key  $arg\_key;

	set $memc\_value $arg\_val;

	set $memc\_exptime $arg\_exptime;

		

	memc\_pass '127.0.0.1:11211';

}

        输出：



$ curl  'http://localhost/memc?cmd=set&key=foo&val=Hello'

$ STORED

$ curl  'http://localhost/memc?cmd=get&key=foo'

$ Hello

        这就实现了memcached的访问，下面看一下如何在lua中访问memcached。

  配置：



\#在Lua中访问Memcached

location = /memc {

	internal;	\#只能内部访问

	set $memc\_cmd get;

	set $memc\_key  $arg\_key;

	memc\_pass '127.0.0.1:11211';

}

location = /lua\_memc {

	content\_by\_lua '

		local res = ngx.location.capture\("/memc", {

			args = { key = ngx.var.arg\_key }

		}\)

		if res.status == 200 then

			ngx.say\(res.body\)

		end

	';

}

        输出：

$ curl  'http://localhost/lua\_memc?key=foo'

$ Hello

        通过lua访问memcached，主要是通过子请求采用一种类似函数调用的方式实现。首先，定义了一个memc location用于通过后端memcached通信，就相当于memcached storage。由于整个Memc模块时非阻塞的，ngx.location.capture也是非阻塞的，所以整个操作非阻塞。

2. Redis

        访问redis需要HttpRedis2Module的支持，它也可以同redis进行非阻塞通行。不过，redis2的响应是redis的原生响应，所以在lua中使用时，需要解析这个响应。可以采用LuaRedisModule，这个模块可以构建redis的原生请求，并解析redis的原生响应。



        配置：



\#在Lua中访问Redis

location = /redis {

	internal;	\#只能内部访问

	redis2\_query get $arg\_key;

	redis2\_pass '127.0.0.1:6379';

} 

location = /lua\_redis {	\#需要LuaRedisParser

	content\_by\_lua '

		local parser = require\("redis.parser"\)

		local res = ngx.location.capture\("/redis", {

			args = { key = ngx.var.arg\_key }

		}\)

		if res.status == 200 then

			reply = parser.parse\_reply\(res.body\)

			ngx.say\(reply\)

		end

	';

}

        输出：

$ curl  'http://localhost/lua\_redis?key=foo'

$ Hello

        和访问memcached类似，需要提供一个redis storage专门用于查询redis，然后通过子请求去调用redis。

3. Redis Pipeline

        在实际访问redis时，有可能需要同时查询多个key的情况。我们可以采用ngx.location.capture\_multi通过发送多个子请求给redis storage，然后在解析响应内容。但是，这会有个限制，Nginx内核规定一次可以发起的子请求的个数不能超过50个，所以在key个数多于50时，这种方案不再适用。

        幸好redis提供pipeline机制，可以在一次连接中执行多个命令，这样可以减少多次执行命令的往返时延。客户端在通过pipeline发送多个命令后，redis顺序接收这些命令并执行，然后按照顺序把命令的结果输出出去。在lua中使用pipeline需要用到redis2模块的redis2\_raw\_queries进行redis的原生请求查询。



        配置：



\#在Lua中访问Redis

location = /redis {

	internal;	\#只能内部访问

 

	redis2\_raw\_queries $args $echo\_request\_body;

	redis2\_pass '127.0.0.1:6379';

} 

	

location = /pipeline {

	content\_by\_lua 'conf/pipeline.lua';

} 

        pipeline.lua

-- conf/pipeline.lua file

local parser = require\(‘redis.parser’\)

local reqs = { 

	{‘get’, ‘one’}, {‘get’, ‘two’} 

}

-- 构造原生的redis查询，get one\r\nget two\r\n

local raw\_reqs = {}

for i, req in ipairs\(reqs\)  do

      table.insert\(raw\_reqs, parser.build\_query\(req\)\)

end

local res = ngx.location.capture\(‘/redis?’..\#reqs, { body = table.concat\(raw\_reqs, ‘’\) }\)

	

if res.status and res.body then

       -- 解析redis的原生响应

       local replies = parser.parse\_replies\(res.body, \#reqs\)

       for i, reply in ipairs\(replies\)  do 

	      ngx.say\(reply\[1\]\)

       end

end

        输出：

$ curl  'http://localhost/pipeline'

$ first

  second

4. Connection Pool

        前面访问redis和memcached的例子中，在每次处理一个请求时，都会和后端的server建立连接，然后在请求处理完之后这个连接就会被释放。这个过程中，会有3次握手、timewait等一些开销，这对于高并发的应用是不可容忍的。这里引入connection pool来消除这个开销。



        连接池需要HttpUpstreamKeepaliveModule模块的支持。



        配置：



http {

	\# 需要HttpUpstreamKeepaliveModule

	upstream redis\_pool {

		server 127.0.0.1:6379;

		\# 可以容纳1024个连接的连接池

		keepalive 1024 single;

 	}

	

	server {

		location = /redis {

			…

			redis2\_pass redis\_pool;

		}

	}

}

        这个模块提供keepalive指令，它的context是upstream。我们知道upstream在使用Nginx做反向代理时使用，实际upstream是指“上游”，这个“上游”可以是redis、memcached或是mysql等一些server。upstream可以定义一个虚拟server集群，并且这些后端的server可以享受负载均衡。keepalive 1024就是定义连接池的大小，当连接数超过这个大小后，后续的连接自动退化为短连接。连接池的使用很简单，直接替换掉原来的ip和端口号即可。

        有人曾经测过，在没有使用连接池的情况下，访问memcached（使用之前的Memc模块），rps为20000。在使用连接池之后，rps一路飙到140000。在实际情况下，这么大的提升可能达不到，但是基本上100-200%的提高还是可以的。



5. 小结

        这里对memcached、redis的访问做个小结。

        1. Nginx提供了强大的编程模型，location相当于函数，子请求相当于函数调用，并且location还可以向自己发送子请求，这样构成一个递归的模型，所以采用这种模型实现复杂的业务逻辑。

        2. Nginx的IO操作必须是非阻塞的，如果Nginx在那阻着，则会大大降低Nginx的性能。所以在Lua中必须通过ngx.location.capture发出子请求将这些IO操作委托给Nginx的事件模型。

        3. 在需要使用tcp连接时，尽量使用连接池。这样可以消除大量的建立、释放连接的开销。



