#### 1. 设置客户端文件大小(可应用于http,server,location)
    ```
        http {
            ...
            client_max_body_size 100M; # 文件上传大小限制，默认1M
            ...
        }
    ```
#### 2. 超时配置 【原著】https://www.zhihu.com/question/554667479/answer/2828688297?utm_id=0
- 客户端超时设置
    - 对于客户端超时主要设置有读取请求头超时时间、读取请求体超时时间、发送响应超时时间、长连接超时时间。通过客户端超时设置避免客户端恶意或者网络状况不佳造成连接长期占用，影响服务端的可处理的能力。
    ``` 
      client_header_timeout time：设置读取客户端请求头超时时间，默认为60s，如果在此超时时间内客户端没有发送完请求头，则响应408（RequestTime-out）状态码给客户端。
    ```
    ```
      client_body_timeout time：设置读取客户端内容体超时时间，默认为60s，此超时时间指的是两次成功读操作间隔时间，而不是发送整个请求体的超时时间，
      如果在此超时时间内客户端没有发送任何请求体，则响应408（RequestTime-out）状态码给客户端。
    ```
    ```
      send_timeout time：设置发送响应到客户端的超时时间，默认为60s，此超时时间指的也是两次成功写操作间隔时间，而不是发送整个响应的超时时间。
      如果在此超时时间内客户端没有接收任何响应，则Nginx关闭此连接。
    ```
    ```
      keepalive_timeout timeout [header_timeout]：设置HTTP长连接超时时间，其中，第一个参数timeout是告诉Nginx长连接超时时间是多少，默认为75s。
      第二个参数header_timeout是用于设置响应头“Keep-Alive: timeout=time”，即告知客户端长连接超时时间。
      两个参数可以不一样，“Keep-Alive:timeout=time”响应头可以在Mozilla和Konqueror系列浏览器起作用，而MSIE长连接默认大约为60s，而不会使用“Keep-Alive: timeout=time”。
      如Httpclient框架会使用“Keep-Alive: timeout=time”响应头的超时（如果不设置默认，则认为是永久）。如果timeout设置为0，则表示禁用长连接。
    ```
- 代理超时设置
    ```
       upstream backend_server {
           server 192.168.61.1:9080 max_fails=2 fail_timeout=10s weight=1;
           server 192.168.61.1:9090 max_fails=2 fail_timeout=10s weight=1;
       }
       server {
           location /test {
              // 网络连接/读/写超时设置
              proxy_connect_timeout 5s;
              proxy_read_timeout 5s;
              proxy_send_timeout 5s;
 
              // 失败重试机制设置、upstream存活超时设置。
              proxy_next_upstream error timeout;
              proxy_next_upstream_timeout 0;
              proxy_next_upstream_tries 0;
 
             // backend_server定义了两个上游服务器192.168.61.1:9080（返回hello）和192.168.61.1:9090（返回hello2）。
             proxy_pass http://backend_server;
             add_header upstream_addr $upstream_addr;
           }
       }
    ```

#### 3. 动静分离配置
      server {
          listen       9291;
          server_name  localhost;
  
          location / {
             root /data-trading-platform/web/dist;
             index index.html index.php;
          }
  
          location /platform-core/ {
             proxy_pass http://localhost:9290;
             proxy_set_header Host $proxy_host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          }
      }

#### 4. 设置跨域
    #允许跨域请求的域，*代表所有
    add_header 'Access-Control-Allow-Origin' *;
    #允许带上cookie请求
    add_header 'Access-Control-Allow-Credentials' 'true';
    #允许请求的方法，比如 GET/POST/PUT/DELETE
    add_header 'Access-Control-Allow-Methods' *;
    #允许请求的header
    add_header 'Access-Control-Allow-Headers' *;

#### 5. 直接访问静态文件
    location /images {
       alias  /home/chenfei/images/;
    }
    # 访问 IP:PORT/images/1.jpg时，Nginx 会去 /home/chenfei/images/1.jpg 查找文件

#### 6. 过滤无效的请求header
    server {
        listen    80;
        server_name     testheader.com;
        underscores_in_headers on; #默认为off，比如请求头里面带有“_”会被过滤掉
        ignore_invalid_headers off; #默认为on，关闭过滤无效的请求header，
        proxy_http_version 1.1;

        location ~* ^/testheader {
            proxy_request_buffering off;
            proxy_next_upstream off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Connection "";
            proxy_read_timeout 180s;

            set $upstream 127.0.0.1:8888;
            proxy_pass http://$upstream ;
        }
    }
