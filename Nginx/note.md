## 进程模型

master：主进程

worker：工作进程



配置文件修改进程数：`worker_process=1`, 也可以设置为`auto`



```bash
$ nginx -s reload # 重新加载配置文件
```



## worker 抢占机制

抢互斥锁，抢到其他 worker 不会处理请求

## nginx 时间处理

使用 linux 的 epoll 模型

异步非阻塞的事件模型，类似 node.js 的事件驱动



## nginx.conf 配置结构

结构:

- main: 全局配置
- event: 配置工作模式及连接数
- http: http 相关配置
  - server: 配置主机配置
    - loacation: 路由规则,表达式
    - upstream: 集群



```yaml
user root; # 运行用户
worker_processes 2 # 运行 worker 数

# debug info notice warn error crit
error_log /etc/logs/nginx/error.log info; # 错误日志, 日志位置, 日志级别

pid /etc/logs/nginx/nginx.pid; # pid 相关的信息

enents {
		use epoll; # 使用 epoll
		worker_connections  1024; # 客户端最大连接数
}

http {
		include /etc/nginx/conf.d/*.conf; # 导入文件
    include /etc/nginx/sites-enables/*; # 
    
		default_type application/octet-stream;
		
		# log_format # 日志格式化
		
		sendfile on;
		tcp_nopush on;
		tcp_nodelay on; # 这几个保证文件的高效传输
		
		keepalive_timeout 65; # tcp 连接保持时间
		
		gzip on; # 数据压缩,提高传输效率
		
		server {
				listen 88;
				server_name api.laoxu.com:4333;
				
				location / {
						root html;
						index index.html;
				}
				error_page 500 /50x.html;
				location = /50x.html {
						root html;
				}
		}
}
```

