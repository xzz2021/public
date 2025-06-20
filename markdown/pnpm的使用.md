##### pnpm的使用

1. win10清理当前项目依赖

   ```bash
   Remove-Item -Recurse -Force node_modules  # 删除node_modules
   pnpm store prune # 清未使用的包
   pnpm store clear # 清缓存
   ```

   