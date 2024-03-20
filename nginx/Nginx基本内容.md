### 一、Nginx的优点
#### 1. 高并发连接
    官方测试能够支持5万并发连接，实际生产环境中跑到2-3万并发连接数。
#### 2. 内存消耗少
    Nginx服务器在3万并发下，开启的10个nginx进程消耗150M内存。
#### 3. 配置文件非常简单
    Nginx的配置文件（nginx.conf）非常简单，容易上手，同时也可以通过简单的命令实现热部署。
#### 4. 免费且开源
    官方版本是免费且开源的，这也是为何许多企业选择用Nginx作为web服务器的原因。
#### 5. 社区活跃
    Nginx官方本身就是一个社区，这个社区非常活跃，遇到问题随时可以通过邮件列表或者IRC(freenode)联系到其他人，很容易找到解决方案。
#### 6. 支持Rewrite重写规则
    能够根据域名、URL不同，将HTTP请求分发到不同的后端服务器上。
#### 7. 节省带宽
    支持GZIP压缩，可以添加浏览器本地缓存的Header头，节省带宽资源。
#### 8. 稳定性高
    宕机的概率非常小，目前还没有发现重大的BUG和漏洞。
#### 9. 支持热部署
    如果Nginx配置错误，不会导致网站无法访问，可以随时通过命令停止nginx，修改配置文件，然后重新加载配置文件，重启nginx，完成部署。
#### 10. 处理请求效率高
    nginx处理请求是异步非阻塞的，支持一个进程处理多个请求，每个请求之间不会造成阻塞，这个是nginx高并发的最核心的地方。

### 二、Nginx在Linux上的安装和卸载
#### 1. 安装步骤
    tar -zxvf nginx-0.x.xx.tar.gz （http://nginx.org/en/download.html）
    cd nginx-0.x.XX
    ./configure --prefix=<path>（如果没有指定，则默认安装到 /usr/ocal/nginx）
    make && make install
#### 2. 是否安装成功检测
    whereis nginx,可看到 /usr/local/nginx 目录。进入到 /usr/local/nginx/sbin 目录，输入命令 ./nginx -V，可看到版本信息和配置信息。
#### 3. 卸载步骤
    systemctl stop nginx或者./nginx -s stop
    find / -name nginx*
    yum remove nginx （卸载依赖项）
    
### 三、Nginx启动、停止、平滑重启
#### 1. Nginx启动
    ./nginx -c /usr/local/nginx/conf/nginx.conf
#### 2. 强制停止所有Nginx进程
    pkill -9 nginx
#### 3. 平滑重启
    ./nginx -s reload
    