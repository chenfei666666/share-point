#### 1. root：指定静态资源目录位置，它可以写在 http 、 server 、 location 等配置中。
* 在server中使用 
  ```
    root    /home/chenfei/images; 
    # 当访问http://182.254.141.64/test/1.jpg时(没有路由匹配)，Nginx 将在/home/chenfei/images/test/1.jpg下寻找相应的文件
  ```
* 在location中使用
  ```
    location /static {
          root  /home/chenfei;
    }
    
    # location + uri; 当访问http://182.254.141.64/static/2.jpg时(有路由匹配)，Nginx 将在/home/chenfei/static/2.jpg下寻找相应的文件
  ```
#### 2. alias：它也是指定静态资源目录位置，它只能写在 location 中。
  ```
  location /images {
        alias  /home/chenfei/images/;
  }
  
  # url - location + uri; 当访问http://182.254.141.64/images/1.jpg时(有路由匹配)，Nginx 将在/home/chenfei/images/1.jpg下寻找相应的文件
  ```
#### 3. index：用来指定默认访问的文件名。index指令可以在http、server和location块中使用，以覆盖全局或特定上下文中的默认设置。

  ```
  location / {
        index  index.html index.htm;
  }
  ```
#### 4. location：配置路径。
* 匹配语法：
  ```
  location [ = | ~ | ~* | ^~ ] uri {
      ...
  }
  ```
  - = 精确匹配；
  - ~ 正则匹配，区分大小写；
  - ~* 正则匹配，不区分大小写；
  - ^~ 匹配到即停止搜索；
  ```
  匹配优先级： = > ^~ > ~ > ~* > 不带任何字符。
  ```
  #####使用实例：  
  ```
  server {
    listen	80;
    server_name	www.nginx-test.com;
    
    # 只有当访问 www.nginx-test.com/match_all/ 时才会匹配到/usr/share/nginx/html/match_all/index.html
    location = /match_all/ {
        root	/usr/share/nginx/html
        index index.html
    }
    
    # 当访问 www.nginx-test.com/1.jpg 等路径时会去 /usr/share/nginx/images/1.jpg 找对应的资源
    location ~ \.(jpeg|jpg|png|svg)$ {
      root /usr/share/nginx/images;
    }
    
    # 当访问 www.nginx-test.com/bbs/ 时会匹配上 /usr/share/nginx/html/bbs/index.html
    location ^~ /bbs/ {
      root /usr/share/nginx/html;
      index index.html index.htm;
    }
  }
  ```
* location中的反斜线  **[原著]<https://blog.csdn.net/weixin_53315414/article/details/129670803>**
  - proxy_pass URL后面加斜杠和不加斜杠的区别
    ```
    在配置Nginx反向代理时，需要根据实际情况选择是否在URL后面加斜杠，以确保请求能够正确地转发到目标服务器。当在后面的url加上了/，相当于是绝对根路径，则 nginx不会把location中匹配的路径部分代理走；如果没有/，则会把匹配的路径部分也给代理走。
    ```
  - location与proxy_pass结合实现反向代理的四种问题
    - 问题1：location后面规则带斜杠，proxy_pass URL后面也带斜杠
    - 问题2：location后面规则不带斜杠，proxy_pass URL后面带斜杠
    - 问题3：location后面规则带斜杠，proxy_pass URL后面不带斜杠
    - 问题4：location后面规则不带斜杠，proxy_pass URL后面不带斜杠
    
    实例应用：
    ```
      @Controller
      public class DemoController {
      
          @GetMapping("/{path}")
          @ResponseBody
          public String  hello(@PathVariable("path") String path) {
              return path;
          }
      }
    ```
    - location后面规则和proxy_pass URL后面都不带斜杠时
    ```
      location /proxy1 {
            proxy_pass  http://localhost:8090;
      }
    
      # 请求http://182.254.141.64/proxy1/test1，代理后实际访问地址http://localhost:8090/proxy1/test1，返回404
    
      location /proxy6 {
            proxy_pass  http://localhost:8090/server6;
      }
    
      # 请求http://182.254.141.64/proxy6/test6，代理后实际访问地址http://localhost:8090/server6/test6，返回404
    
    ``` 
    - location后面规则带斜杠proxy_pass URL后面不带斜杠时
    ```
      location /proxy2/ {
            proxy_pass  http://localhost:8090;
      }
    
      # 请求http://182.254.141.64/proxy2/test2，代理后实际访问地址http://localhost:8090/proxy2/test2，返回404
    
      location /proxy5/ {
            proxy_pass  http://localhost:8090/server5;
      }
    
      # 请求http://182.254.141.64/proxy5/test5，代理后实际访问地址http://localhost:8090/server5test5，返回proxy5test5
    ``` 
    - location后面规则不带斜杠proxy_pass URL后面带斜杠时
    ```
      location /proxy3 {
            proxy_pass  http://localhost:8090/;
      }
      
      # 请求http://182.254.141.64/proxy3/test3，代理后实际访问地址http://localhost:8090//test3，返回test3  
    ```
    - location后面规则和proxy_pass URL后面都带斜杠时
    ```
      location /proxy4/ {
            proxy_pass  http://localhost:8090/;
      }
      
      # 请求http://182.254.141.64/proxy4/test4，代理后实际访问地址http://localhost:8090/test4，返回test4
    ```
    记忆法：
    - 针对alias和root
      ```
        location /images { 
          alias /home/chenfei/images/; 
        }
    
        # 请求http://182.254.141.64/images/1.jpg时，Nginx会去掉匹配的location后面的规则，然后去/home/chenfei/images/1.jpg目录下找文件；
      ```
      ```
        location /images01 { 
          root /home/chenfei/; 
        }
    
        # 请求http://182.254.141.64/images01/1.jpg时，Nginx会将匹配的location后面的规则添加到转发的路径中，即去/home/chenfei/images01/1.jpg目录下找文件；
      ```
    - 针对proxy_pass
      ```
      (1) 当location后面规则和proxy_pass URL后面都带斜杠时，如果请求的URL中也带有斜杠，Nginx会将请求转发到proxy_pass指定的URL；如果请求的URL中没有斜杠，Nginx会自动加上一个斜杠后再进行转发。
      (2) 当location后面规则不带斜杠，但proxy_pass URL后面带斜杠时，如果请求的URL中带有斜杠，Nginx会将请求转发到proxy_pass指定的URL；如果请求的URL中不带斜杠，Nginx会自动加上一个斜杠后再进行转发。
      (3) 当location后面规则带斜杠，但proxy_pass URL后面不带斜杠时，如果请求的URL中也带有斜杠，Nginx会将请求转发到proxy_pass指定的URL；如果请求的URL中没有斜杠，Nginx会自动去掉location后面规则中的斜杠后再进行转发。
      (4) 当location后面规则和proxy_pass URL后面都不带斜杠时，如果请求的URL中也不带斜杠，Nginx会将请求转发到proxy_pass指定的URL；如果请求的URL中带有斜杠，Nginx会自动去掉proxy_pass URL后面的斜杠后再进行转发。
    ```
      ```