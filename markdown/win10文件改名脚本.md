### 文件改名脚本

1. 当前文件夹下所有文件名包含 `哈哈哈` 的, 去除 `哈哈哈`  部分进行重命名

   ```bash
   @echo off
   chcp 65001 >nul
   setlocal enabledelayedexpansion
   
   :: 遍历当前文件夹中的所有文件
   for %%f in (*.*) do (
       set "filename=%%~nf"
       set "extension=%%~xf"
       
       :: 检查文件名是否包含 哈哈哈
       if not "%%~nf"=="!filename:哈哈哈=!" (
           :: 去除文件名中的 哈哈哈 并重命名
           set "newname=!filename:哈哈哈=!"
           ren "%%f" "!newname!!extension!"
           echo 已重命名：%%f 为 !newname!!extension!
       )
   )
   
   echo 所有包含“哈哈哈”的文件已重命名完成。
   pause
   
   ```

   

2. 都是