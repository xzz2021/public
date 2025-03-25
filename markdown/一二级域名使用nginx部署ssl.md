#### 一二级域名使用nginx部署ssl

1. 一级域名配置, 域名服务商解析记录到服务器ip

   > 如域名为`test.com`, 设置主机记录:@,记录类型:A, 记录值: 8.145.198.125, 则`test.com`会解析到服务器;
   >
   > 申请根域名证书, 配置nginx, 自动重定向到443接口
   >
   > ```
   > server {
   >     listen       80;
   >     listen  [::]:80;
   >     server_name  test.com;
   >  	location / {
   >         return 301 https://$host$request_uri;
   >     }
   > }
   > server {
   >     listen 443 ssl http2;
   >     server_name test.com;
   >     ssl_certificate_key   /etc/nginx/cert/test.com.key;
   >     ssl_certificate       /etc/nginx/cert/test.com.pem;
   > }
   > ```

2. 二级域名配置, 域名服务商解析记录到服务器ip, 和根域名解析一样, 区别只在nginx配置文件

   > 如域名为`aaa.test.com`, 设置主机记录:@,记录类型:A, 记录值也是: 8.145.198.125, 则`aaa.test.com`会解析到服务器;
   >
   > 申请aaa.test.com域名证书, 配置nginx; (一级域名服务对应服务器的80端口, 其他二级域名服务对应服务器其他接口上的服务, 如下为2020)
   >
   > ```
   > server {
   >     listen 443 ssl http2;
   >     server_name aaa.test.com;
   >     ssl_certificate_key   /etc/nginx/cert/aaa.test.com.key;
   >     ssl_certificate       /etc/nginx/cert/aaa.test.com.pem;
   >     location / {
   >         proxy_pass http://127.0.0.1:2020;  # 将流量转发到目标端口
   >         proxy_set_header Host $host;
   >         proxy_set_header X-Real-IP $remote_addr;
   >         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   >         proxy_set_header X-Forwarded-Proto $scheme;
   >         proxy_read_timeout 300s;
   >         proxy_send_timeout 300s;
   >     }
   > }
   > ```

3. 以上nginx配置项都是基础设置,其他根据实际情况相应自行增减