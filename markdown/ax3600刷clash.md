刷机流程记录备用,省的每次要重新找答案

1. 确定路由器固件版本1.0.17,验证管理密码进入路由后台管理的pc界面,浏览器url里拿到stok=后面的参数,以下是自己路由的stok,不同路由都不一样

   ```js
   http://192.168.31.1/cgi-bin/luci/;stok=457e3d16bbc8d8cf60fe263c6d9bd181/web/home#router457e3d16bbc8d8cf60fe263c6d9bd181
   ```

2. 分别执行获取ssh和修改默认密码为admin

   ```
   http://192.168.31.1/cgi-bin/luci/;stok=457e3d16bbc8d8cf60fe263c6d9bd181/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B
   ```

   ```
   http://192.168.31.1/cgi-bin/luci/;stok=457e3d16bbc8d8cf60fe263c6d9bd181/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B
   ```

3. 理论还需要固化ssh,之前开启过,此处暂且略过

4. 一键安装shellclash

   ```
   export url='https://fastly.jsdelivr.net/gh/juewuy/ShellCrash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
   
   ```

   