pnpm使用monorepo

1. 初始化目录 pnpm init

2. 指定运行引擎, 根目录package.json文件添加配置

   ```json
   {
       "engines": {
           "node": ">=20",
           "pnpm": ">=9"
       },
       "private": true,   // 防止根目录被当作包发布
   }
   ```

3. pnpm自身支持monorepo, 创建packages文件夹存放子包, 根目录创建`pnpm-workspace.yaml`文件,定义子包目录

   ```yaml
   packages:
     - 'packages/*'
   ```

4. 同时运行子项目,可以使用`concurrently`依赖库, 或者pnpm自带的递归功能`-r`

   ```json
   "scripts": {
       "start": "pnpm -r run dev"
     }
   ```

   