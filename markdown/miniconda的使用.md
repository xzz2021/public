#### miniconda的使用

##### 官网[安装包](https://www.anaconda.com/download/success)

1. 创建虚拟环境

   ```bash
   conda create -n <name> python=<version>
   ```

2. 查看虚拟环境列表

   ```bash
   conda env list
   ```

3. 激活指定环境

   ```bash
   conda activate <name>
   ```

4. 退出当前虚拟环境

   ```bash
   conda deactivate
   ```

5. 删除虚拟环境

   ```bash
   conda env remove -n <name>
   ```

6. 查看当前环境依赖库

   ```bash
   conda list
   # 删除库
   conda remove <name>   
   ```

   