学习笔记

1. 迭代器特性是数据每次调用后即在内存中清空

2. 进程与线程, 池与锁; 软件运行需要占用cpu及内存(食物/能源);  线程负责执行业务逻辑(大脑分配任务处理); 

   ```python
   a_p = multiprocessing.Process(target=work_a)
   b_p = multiprocessing.Process(target=work_b)
   for i in (a_p, b_p)
   	p.start()  # 创建子进程  与主进程按先后顺序同步执行
   # 假如使用p.join()则会阻塞主进程,等子进程结束后再执行主进程任务
   ```

   >  使用多进程弊端: 执行函数不会获取到返回值, 同时操作文件会导致错误(进程锁), 资源不足(进程池)

   ````python
   pool = multiprocessing.Pool(processes=5)
   # 进程池会管理子进程
   pool.join()  ## 需要join  不然主进程退出结束  子进程会来不及执行
   #  获取返回值
   result = pool.apply_async(func=worak, args=(6,))
   res = result.get()  ## 获取函数return的返回值
   ````

   > 进程锁: acquire()锁住进程, release()开锁

   ```python
   multiprocessing.Pool(processes=5)
   lock = manager.Lock()
   ```

   > 进程通信: 使用队列传入信息 ???复杂6.1.6.1

   ```python
   queue = multiprocessing.Queue(5)
   ```

   > 线程Thread:  start join setDaemon-守护线程
   >
   > 使用线程可以让循环命令几乎同时进行,线程都是在主进程中执行的
   >
   > 线程池/线程锁: submit-加入    

   ```python
   from concurrent.futures import ThreadPoolExecutor
   t = ThreadPoolExecutor(3)  # 创建线程
   
   ```

   

3. 