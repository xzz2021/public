docker镜像本地保存及上传加载

1. 在能访问外网的设备pull拉取image, 

   ```bash
   docker pull <image-name>
   ```

2. 保存tar镜像或压缩包到本地文件当前目录, win10没有gzip, 可以使用git bash运行

   ```bash
   docker save -o <image-name>.tar <image-name>:tag
   docker save <image-name> | gzip > <image-name>.tar.gz  # 压缩
   ```

3. 镜像上传到目标主机上并加载

   ```bash
   # 如果压缩了先解压再加载
   gunzip -c <image-name>.tar.gz | docker load
   # 未压缩直接执行加载
   docker load -i <image-name>.tar
   ```

   