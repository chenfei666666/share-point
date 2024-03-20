#### 
    user  nginx; # 运行的用户
    worker_processes  auto; # 工作进程数，"auto" 表示自动根据 CPU 核心数确定, 一般为CPU核心的倍数
    
    #error_log  logs/error.log  error; # 错误日志路径和级别
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;
    
    pid        logs/nginx.pid; # 进程 ID 文件路径
    
    
    events {
        #use epoll; 使用epoll的I/O模型
        worker_connections  1024; # 每个工作进程的最大并发连接数
    }

    http {
        include       mime.types;
        default_type  application/octet-stream;
    
        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
        #access_log  logs/access.log  main; # 访问日志路径和格式
    
        sendfile        on; # 开启高效传输模式
    
        tcp_nopush          on;   # 减少网络报文段的数量
        tcp_nodelay         on;
        keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
        types_hash_max_size 2048;
        client_max_body_size 100M; # 文件上传大小限制，默认1M

        include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
        default_type        application/octet-stream;   # 默认文件类型

        include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    
        #gzip  on;
        server {
            listen       80; # 监听的端口
            server_name  localhost; # 配置的域名
            #精确匹配： server_name www.nginx.com ;
            #左侧通配： server_name *.nginx.com ;
            #右侧统配： server_name www.nginx.* ;
            #正则匹配： server_name ~^www\.nginx\.*$ ;
        
            location / {
                proxy_pass  http://localhost:8080;
                index  index.html index.htm;
                #这里配置宕机检测，都设置为1秒，这是有了负载均衡过后配置的，如果访问时挂了一个服务器，1秒不响应就自动切换到另外应用服务器进行访问
                proxy_connect_timeout 1;
                proxy_send_timeout 1;
                proxy_read_timeout 1;
        }

        error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }   