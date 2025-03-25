vite + vue + js 添加eslint和prettier

1. 安装eslint

   ````bash
   npm init @eslint/config@latest # 初始化创建config文件
   ````

2. 安装prettier, 编译器安装插件

   ```
   pnpm add --save-dev --save-exact prettier
   ```

3. 创建一个空的配置文件, 同时创建.prettierignore文件, 添加忽略检查文件

   ```
   node --eval "fs.writeFileSync('.prettierrc','{}\n')"
   ```

4. 安装eslint-config-prettier, 以兼容二者,eslint.config.js文件添加配置

   ```
   pnpm add -D eslint-config-prettier
   ```

   ```
   import eslintConfigPrettier from "eslint-config-prettier";
   
   export default [
     // ...其他配置
     eslintConfigPrettier,
   ];
   ```

   

5. 编译器配置保存自动格式化,根目录建立.vscode/setting.json文件

   ```json
   {
       "editor.codeActionsOnSave": {
           "source.fixAll": "explicit",
           "source.fixAll.eslint": "explicit"
       },
       "editor.defaultFormatter": "esbenp.prettier-vscode",
       "editor.formatOnSave": true,
       "eslint.alwaysShowStatus": true
   }
   
   ```

   

6. 