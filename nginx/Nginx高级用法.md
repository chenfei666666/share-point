#### 1. Rewrite重写
- 定义
    ```aidl
    Rewrite功能就是使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。
    ```
    ```aidl
    Rewrite只能放在 server { }, location { }, if { }中，并且只能对域名后边的除去传递的参数外的字符串起作用。 例如 http://seanlook.com/a/we/index.php?id=1&u=str 只对/a/we/index.php重写。
    ```
- 语法
    ```aidl
    rewrite regex replacement [flag];
    flag标记说明：
    last  #本条规则匹配完成后，继续向下匹配新的location URI规则
    break  #本条规则匹配完成即终止，不再匹配后面的任何规则
    redirect  #返回302临时重定向，浏览器地址栏会显示跳转后的URL地址
    permanent  #返回301永久重定向，浏览器地址栏会显示跳转后的URL地址
    ```  
- 正则表达式
  
  | 正则   |      说明      |   正则     |     说明  |
  |----------|:---------|:----------|:----------|
  | . |    匹配除换行符以外的任意字符| $ | 匹配字符串的结束 <img width=500/>
  | ? |    重复0次或1次   |   {n} |重复n次
  | + | 重复1次或更多次 |    {n,} |重复n次或更多次
  | *| 重复0次或更多次	 |    [c] |匹配单个字符c
  | \d | 匹配数字 |    [a-z] |匹配a-z小写字母的任意一个
  | ^ | 匹配字符串的开始 |   (pattern) |(pattern)之间匹配的内容，可以在后面通过$1来引用，$2表示的是前面第二个()里的内容

- 举例

    ```
      server {
        listen 80;
        server_name www.mfc.com;
        rewrite  ^/api/(.*)  http://www.test.com/$1 permanent;
      }
      #访问地址：www.mfc.com/api/login, URL就会被重写为：www.test.com/login
    ```

- 应用场景
  - 不能废除旧域名，从旧域名跳转到新域名，且保持其参数不变。（公司旧域名，因业务需求有变更，需要使用新域名www.kgc.com 代替）
  - 当用户通过http的方式访问时，将其跳转至https的方式访问，或者通过https访问跳转到http的方式访问。

#### 2. SSL配置
  为了提供安全的连接，可以配置Nginx支持SSL，需要生产SSL需要的证书文件（[原著]https://blog.csdn.net/qq_41481367/article/details/132331520）。
  ```aidl
    server {
      listen 443 ssl;
      server_name secure-example.com;
  
      ssl_certificate /etc/nginx/ssl/secure-example.com.crt;
      ssl_certificate_key /etc/nginx/ssl/secure-example.com.key;
  
      location / {
          root /var/www/html/secure-example;
          index index.html;
      }
    }

  ```

#### 3. Nginx可视化页面
  https://nginxproxymanager.com/

#### 4. Nginx中$host、$http_host、$proxy_host的区别
  ```aidl
  server{
	listen       80;
	#server_name  localhost;
	
	location / {
                root /data/run/aqscweb/tyxx/;
                index index.html /tyxx/index.html;
        }
	
	location /api/ {
        proxy_pass http://172.19.101.29:30000/;
        proxy_set_header   Host $http_host;
  		proxy_set_header  X-real-ip $remote_addr;
  		proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
        	proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }

	location /plug/video/ {
        proxy_pass http://172.19.101.36:7080/;
  		proxy_set_header  Host  $host;
  		proxy_set_header  X-real-ip $remote_addr;
  		proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
 		proxy_http_version 1.1;
 		proxy_set_header Upgrade $http_upgrade;
  		proxy_set_header Connection "upgrade";
        }
	
    location /portal {
        proxy_pass http://172.19.101.31:8080;
        proxy_set_header  Host  $proxy_host;
        proxy_set_header  X-real-ip $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        }
}

   ```
  | 变量   |      是否显示端口      |   值     |
  |:----------:|:---------:|:----------:|
  | $host |    不显示端口 | 浏览器请求的ip，不显示端口 | 
  | $http_host |    端口存在则显示   |  浏览器请求的ip和端口号 |
  | $proxy_host | 默认80端口不显示，其它显示 |    被代理服务的ip和端口号 |