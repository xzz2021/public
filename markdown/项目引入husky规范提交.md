项目引入husky规范提交

##### 实现功能: typescript及eslint检查, 代码自动格式化,提交信息内容校验

##### 实现步骤

1. 安装4个依赖

   ````bash
   pnpm i husky lint-staged @commitlint/cli @commitlint/config-conventional  -D
   ````

2. 执行初始化命令,会自动在项目根目录生成`.husky`文件夹

   ```bash
   pnpm husky
   ```

3. `package.json` 文件添加以下命令

   ````json
    "scripts": {
       "prepare": "husky",
       "ts:check": "tsc --noEmit --skipLibCheck",
       "lint:lint-staged": "lint-staged -c ./.husky/lintstagedrc.cjs"
     }
   ````

4. 自行建立以下3个文件, 添加到`.husky` 文件夹下

   ```json
   // 文件名 pre-commit  
   //  这里表示需要执行的操作
   npm run ts:check
   npm run lint:lint-staged
   ```

   ````json
   //  文件名 lintstagedrc.cjs  这里自定义需要检查的文件类型
   module.exports = {
     '*.{js,jsx,ts,tsx}': ['eslint --fix', 'prettier --write'],
     '{!(package)*.json,*.code-snippets,.!(browserslist)*rc}': ['prettier --parser json --write'],
     'package.json': ['prettier --write'],
     '*.md': ['prettier --write'],
   };
   ````

   ```json
   //  文件名  commit-msg
   npx --no -- commitlint --edit $1
   ```

5. 再新建一个文件放入项目根目录

   ````js
   //  commitlint.config.cjs
   //   这里表示定义好的 提交信息文本格式
   module.exports = {
     extends: ['@commitlint/config-conventional'],
     rules: {
       'type-enum': [
         2,
         'always',
         [
           'feat', // 新功能(feature)
           'fix', // 修补bug
           'docs', // 文档(documentation)
           'style', // 格式、样式(不影响代码运行的变动)
           'refactor', // 重构(即不是新增功能，也不是修改BUG的代码)
           'perf', // 优化相关，比如提升性能、体验
           'test', // 添加测试
           'ci', // 持续集成修改
           'chore', // 构建过程或辅助工具的变动
           'revert', // 回滚到上一个版本
           'workflow', // 工作流改进
           'mod', // 不确定分类的修改
           'wip', // 开发中
           'types', // 类型修改
           'release' // 版本发布
         ]
       ],
       'subject-full-stop': [0, 'never'],
       'subject-case': [0, 'never']
     }
   }
   
   ````

   