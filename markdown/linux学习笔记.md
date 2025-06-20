### linux 学习笔记

1. 常用命令

   ```bash
   cd ls rm
   pwd #查看当前目录
   touch #创建一个文件
   echo "内容" > 文件.txt  #添加内容到文件
   cat  #查看文件内容
   head -3 #查看文件头部3行
   tail -3 #查看文件底部3行
   cp 源文件  目标文件  #复制文件到新文件
   mv 源文件  目标文件  # 移动文件  # mv aa.txt .. #移动aa到上级目录 #也可用于更换文件名称
   ln  #建立链接  类似快捷方式 分为 软链接-s 硬链接
   # ln 硬链名称 目标文件    <--硬链接   相当于创建了一个可以同步更新的新文件
   # ln -s 目标文件 软链名称    <--软链接   相当于指针
   wc 文件.txt # 统计文件行数,字符数,单词数
   #=========压缩与解压=======
   tar xvf test.tar   #解压
   tar cvf test.tar  /dir1/dir2  file    # 压缩到指定目录  保留源文件
   tar xzvf/czvf test.tar.gz  #  操作gz后缀的压缩文件
   # .gz/.z/.Z/.tgz  使用gzip   只能压缩单个文件???
   gzip 文件1  文件2  #分别压缩2个文件
   gzip -d testfile.gz  # 解压文件到当前目录同时会移除原来的压缩包
   gzip -r  dir1/dir2   #压缩当前目录下所有文件
   gzip -rd  dir1/dir2  #解压当前目录下所有压缩文件
   #===========
   which python   # 查询可执行程序的路径及别名
   time  #计算程序运行时长   time tar cvf test.tar  /dir1
   date  # 查看当前系统时间
   uptime #系统相关信息
   du  # 统计文件和目录占用磁盘空间
   cal   #查看日历
   bc # 计算机
   top  # cpu使用率
   #========查找========
   find . -name test #  查找当前目录所有名称为test的文件   # -iname  忽略大小写
   grep "内容" -r . # 过滤 查找当前目录下内容里包含"内容"的文件
   ps -A | grep odoo # 过滤查找进程
   cat test.txt | grep "odoo内容" # 查找定位内容在文件内的位置
   #=========重定向=======

   ```

2. vi 和 vim

   > i 插入
   > o 新增空白行
   > :wq 保存退出 :q 未修改直接退出 :q! 修改后不保存退出
   > 配置文件 /etc/vim/vimrc.local

3. ```bash
   chown user:group file  # 修改文件所有者
   r #读
   w #写
   x #执行
   -rwx--xr-- # u-所有者  g-组 o-其他用户 a-所有用户
   chmod o+x filename  # 给指定用户或组或所有者添加+相应权限
   chmod o=rx filename # 给指定用户或组或所有者更改=相应权限
   chmod a-w filename # 给指定用户或组或所有者减少-相应权限
   chmod 666 filename # 权限对应数字 r 4 w 2 x 1
   ```

4. 进程管理

   ```bash
   ps a  #终端相关进程
   ps x  #非终端  服务相关进程
   top   #查看cpu利用率
   top -d 1 -p 12 -p 34 # -d 进程刷新率2秒刷新一次  -p 要查看的进程id
   ```

5. 软件源码 编译 安装 卸载

   ```bash
   autoconf  #自动生成软件配置文件 configure
   ./configure  #进行配置
   make install  #执行安装
   
   ```
