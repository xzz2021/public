系统学习笔记!

1. 路由指向 <RouterLink> 路由渲染的切换 <RouterView> 

2. 路由模式

   > 1. hash     `createWebHashHistory()`
   > 2. Memory     `createMemoryHistory()`
   > 3. HTML5     `createWebHistory()`

3. 编程式导航

   > 当前路由 useRoute    全局路由实例useRouter 

4. 路径参数, :id 可以通过params.id访问, 带参组件会被重复利用, 不会唤起生命周期, 需自行watch变化

   ```js
   { path: '/users/:id', component: User }
   ```

5. 路由匹配

   ```js
    // 捕获所有路由或404路由
   { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound }
   // /:orderId -> 仅匹配数字
   { path: '/:orderId(\\d+)' },
   // /:productName -> 匹配其他任何内容
   { path: '/:productName' },
   // sensitive 大小写敏感   strict  尾部斜线\
   ```

   公共前缀路由,可以省去父级component

   ```js
   const routes = [
     {
       path: '/admin',
       children: [
         { path: '', component: AdminOverview },
         { path: 'users', component: AdminUserList },
         { path: 'users/:id', component: AdminUserDetails },
       ], 
     },
   ]
   ```

   `params` 不能与 `path` 一起使用

6. 重定向

   ```js
   const routes = [{ path: '/home', redirect: { name: 'homepage' } }]
   // 动态返回重定向目标
   const routes = [
     {
       // /search/screens -> /search?q=screens
       path: '/search/:searchText',
       redirect: to => {
         // 方法接收目标路由作为参数
         // return 重定向的字符串路径/路径对象
         return { path: '/search', query: { q: to.params.searchText } }
       },
     },
   ]
   //  相对重定向
   const routes = [
     {
       // 将总是把/users/123/posts重定向到/users/123/profile
       path: '/users/:id/posts',
       redirect: to => {
         // 该函数接收目标路由作为参数
         // 相对位置不以`/`开头
         // 或 { path: 'profile'}
         return 'profile'
       },
     }
   ]
   // 别名
   //  将 / 别名为 /home，意味着当用户访问 /home 时，URL 仍然是 /home，但会被匹配为用户正在访问 /
   const routes = [{ path: '/', component: Homepage, alias: '/home' }]
   ```

7. 路由传参   props

   ```js
   const routes = [ { path: '/user/:id', component: User, props: true } ]
   //开启此选型时  route.params 将被设置为组件的 props
   ```

8. 导航守卫

   > 1.  beforeEach   全局前置守卫
   >
   > 2. beforeResolve  全局解析守卫 在进入页面后 要处理页面的异步请求时使用
   >
   > 3. afterEach    全局后置钩子
   >
   > 4.  全局注入变量 
   >
   >    ```js
   >    // main.ts
   >    const app = createApp(App)
   >    app.provide('global', 'hello injections')
   >    // router.ts or main.ts
   >    router.beforeEach((to, from) => {
   >      const global = inject('global') // 'hello injections'
   >      // a pinia store
   >      const userStore = useAuthStore()
   >    })
   >    ```
   >
   > 5. 组件内的守卫  `beforeRouteEnter ` `beforeRouteUpdate`  `beforeRouteLeave`

9. 路由元信息

10. 路由懒加载

