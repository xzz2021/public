#### prisma使用mongodb

1. **踩坑记录:** 必须开启副本集, 即使是单节点使用, 否则无法操作mongodb

2. docker compose 部署, mongodb.key用于多节点认证, 单节点理论上不需要

   ```yml
   services:
     mongo:
       image: mongo
       restart: always
       container_name: "mongodb"
       command: mongod --replSet rs0 --bind_ip_all --keyFile /mongodb.key
       entrypoint:
         - bash
         - -c
         - |
           chmod 400 /mongodb.key
           chown 999:999 /mongodb.key
           exec docker-entrypoint.sh $$@
       ports:
       - "27017:27017"
       volumes:
         - "./mongo/data:/data/db"
         - "./mongo/mongod.conf:/etc/mongod.conf"
         - "./mongo/logs:/var/log/mongodb"
         - "./mongo/mongodb.key:/mongodb.key"  # 集群必须使用一致的key
       environment:
         MONGO_INITDB_ROOT_USERNAME: root
         MONGO_INITDB_ROOT_PASSWORD: xzz...
   ```

3. 部署后,进入容器, 使用认证连接

   ```bash
   mongosh --authenticationDatabase "admin" -u "root" -p "xzz..."
   # 命令行执行初始化 副本集设置
   rs.initiate({_id: "rs0", members: [{_id: 0, host: "localhost:27017"}]})
   ```

4. 生成key

   ```bash
   openssl rand -base64 756 > mongodb.key   # 756可以是任意数字
   chmod 400 mongodb.key
   ```

5. 