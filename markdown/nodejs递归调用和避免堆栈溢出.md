#### nodejs递归调用和避免堆栈溢出

1. 每次调用 `processItems`，都会压入一个新的调用帧到调用栈中, 如果递归深度过大（例如处理一个包含 10,000 个元素的数组），调用栈可能会耗尽，从而引发 **堆栈溢出错误**

   ```ts
   function processItems(items) {
     if (items.length === 0) return; // 递归终止条件
     const item = items.pop(); // 从数组中取一个元素
     console.log(`Processing: ${item}`);
     processItems(items); // 递归调用
   }
   processItems([1, 2, 3, 4, 5]);
   ```

2. 改进, `process.nextTick()` 调度一个回调，将递归调用 `processItems` 推入下一个队列,当前的调用栈会清空，然后从 `下一个队列` 中取出任务并执行, 主线程调用栈不会因为递归而变深

   ```ts
   function processItems(items) {
     if (items.length === 0) return; // 递归终止条件
     const item = items.pop(); // 从数组中取一个元素
     console.log(`Processing: ${item}`);
     // 用 process.nextTick 推迟递归调用
     process.nextTick(() => processItems(items));  // 异步但优先于其他任务执行
   }
   processItems([1, 2, 3, 4, 5]);
   ```

   

