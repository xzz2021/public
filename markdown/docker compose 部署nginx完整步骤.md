**docker compose 部署nginx完整步骤**

1. 配置文件

   ```yml
   services:
     nginx:
       image: nginx
       restart: always
       container_name: nginx
       ports:
        - "80:80"
        - "443:443"
       volumes:
         - ./nginx/html:/usr/share/nginx/html
         - ./nginx/www:/var/www
         - ./nginx/logs:/var/log/nginx
         - ./nginx/cert:/etc/nginx/cert
         # 配置文件无法自动挂载, 需要手动复制
         - ./nginx/conf.d:/etc/nginx/conf.d # 这是文件夹 conf.d/default.conf
         - ./nginx/nginx.conf:/etc/nginx/nginx.conf # 这是文件
   ```

2. 启动后手动执行命令将容器内文件复制出来,conf.d文件夹下是默认的配置文件

   ```bash
   docker compose up -d # 部署服务
   docker cp nginx:/etc/nginx/conf.d ./conf.d
   docker cp nginx:/etc/nginx/nginx.conf ./nginx.conf
   ```

3. 停止容器或删除容器, 重启或重新执行部署