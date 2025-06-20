#### 3d文件解析环境配置

**安装驱动后重启电脑**

1. 检查主机显卡是否支持[cuda](https://developer.nvidia.cn/cuda-gpus)以及计算能力, 检查cupy[版本支持信息](https://cupy.dev/),  检查cuda支持的对应显卡驱动版本[对照表](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html), 下拉table3列出详细信息, 其中update后的数字对应toolkit版本第三位, cuda toolkit历史版本[下载表](https://developer.nvidia.com/cuda-toolkit-archive), cupy和其他解析库对应版本详细[对照信息](https://docs.cupy.dev/en/stable/install.html), 下载conda[安装包](https://www.anaconda.com/download/success)

2. 安装nvidia驱动, 查询下载自己的[版本](https://www.nvidia.com/en-us/drivers/)

   ```bash
   # 驱动版本信息及cuda信息查看
   nvidia-smi
   ```

3. 可能???????宿主机要安装cuda工具包, [官网下载](https://developer.nvidia.com/cuda-toolkit-archive)

4. 假如是宿主机运行, 环境准备好之后, 激活虚拟环境, 执行依赖安装命令

   ```bash
   conda create -n <name> python=3.12
   conda activate <name>
   conda install -c conda-forge cupy pythonocc-core -y
   ```

   

