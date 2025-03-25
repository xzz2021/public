#### docker部署node项目

1. 服务器重启后项目自启

   ```bash
   # docker容器自启   restart: always
   # pm2自启   pm2 startup & pm2 save (保存自启时需启动的应用)
   ```

2. pm2常用命令

   ```bash
   # 列举项目
   pm2 list
   # 可视化仪表监视器
   pm2 monit
   # web诊断仪表板
   pm2 plus
   # 定时重启
   pm2 start app.js --cron-restart="0 0 * * *"
   
   # 多应用部署 类似docker compose  定义ecosystem.config.js文件
   pm2 ecosystem  # 执行
   pm2 init simple  # 生成初始化ecosystem.config.js文件
   # 相关执行命令  start stop  restart  reload  delete
   pm2 start ecosystem.config.js
   
   # 使用集群模式  max自动检测可用 CPU 的最大数量
   pm2 start app.js -i max
   
   # 远程部署 deploy
   
   # 平滑退出或重启
   
   ```

   