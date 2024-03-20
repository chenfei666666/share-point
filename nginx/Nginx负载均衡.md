### 负载均衡策略
- 轮询（Round Robin）策略
  - 轮询是最常见的负载均衡策略之一，也是Nginx的默认策略。它将请求均衡地分发到后端的多个服务器上，每个请求按顺序依次分发到不同的服务器上。当有服务器宕机时，Nginx会自动将其排除在负载均衡的范围外。
    ```
    http {
      upstream backend {
        server 192.168.1.1;
        server 192.168.1.2;
        server 192.168.1.3;
      }

      server {
        listen       80;
        server_name  example.com;

        location / {
          proxy_pass http://backend;
        }
      }
    }
    ```    
- 最少连接（Least Connections）策略
  - 最少连接策略会将请求发送到当前连接数最少的服务器上，以实现负载均衡。这样可以确保每个服务器上的连接数相对均衡，避免某个服务器被过度压力。Nginx提供了一个模块least_conn来实现最少连接策略。
    ```
    http {
      upstream backend {
        least_conn;
        server 192.168.1.1;
        server 192.168.1.2;
        server 192.168.1.3;
      }

      server {
        listen       80;
        server_name  example.com;

        location / {
          proxy_pass http://backend;
        }
      }
    }
    ```
- IP哈希（IP Hash）策略
  - IP哈希策略会根据客户端的IP地址将请求分发到后端服务器上。这样可以确保同一个客户端的请求都会被发送到同一个后端服务器上，提高缓存的效果。Nginx提供了一个模块ip_hash来实现IP哈希策略。
    ```
    http {
      upstream backend {
        ip_hash;
        server 192.168.1.1;
        server 192.168.1.2;
        server 192.168.1.3;
      }

      server {
        listen       80;
        server_name  example.com;

        location / {
          proxy_pass http://backend;
        }
      }
    }
    ```
- 加权轮询（Weighted Round Robin）策略
  - 加权轮询策略允许给不同的服务器设置不同的权重，服务器的权重越高，被选中的概率就越大。这样可以有效地分配服务器的负载压力。
    ```
    http {
      upstream backend {
        #weight 值越大,负载权重越大,请求次数越多             
        #max_fails 允许请求失败的次数，超过失败次数后，转发到下一个服务器，当有max_fails个请求失败，就表示后端的服务器不可用，默认为1，将其设置为0可以关闭检查   
        #fail_timeout 指定时间内无响应则失败, 在以后的fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器
        #down 表示当前server不参与负载
        #backup 其他非backup server都忙的时候，backup server作为备用服务器，将请求转发到backup服务器
        server 192.168.1.1 weight=3 max_fails=2 fail_timeout=30s;
        server 192.168.1.2 weight=2 max_fails=2 fail_timeout=30s;
        server 192.168.1.3 weight=1 max_fails=2 fail_timeout=30s;
      }

      server {
        listen       80;
        server_name  example.com;

        location / {
          proxy_pass http://backend;
        }
      }
    }
    ```