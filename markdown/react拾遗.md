### react拾遗

##### `Class组件`

1. Class组件onClick合成点击事件函数传参

   ```jsx
   <button onClick={this.handleClick(123)}>点击</button>
   // 高阶函数
   const handleClick = num => {
       return ev => { console.log(num)}
   }
   //  ??箭头函数
   <button onClick={(ev) => this.handleClick(ev, 123)}>点击</button>
   ```

2. 响应式数据实现state, 通过setState()触发render()重新渲染

   ```jsx
   this.setState({count:4})  // 异步执行
   this.setState({count:4}, ()=>{})  // 异步执行结束之后的回调函数
   this.setState(state => {count:state.count + 1})  //  批处理多次执行数据累加处理
   ```

3. shouldComponentUpdate控制是否重新渲染组件, PureComponent

   ```jsx
   Class Welcome extends React.PureComponent{}  //  自动判定,数据有变化才重新render
   ```

4. 复杂多层级数据使用immutable(深拷贝优化版)

5. 受控组件处理

   ```jsx
   <input type="text" value={this.state.msg} onChange={this.handleChange} />
   handleChange = e => {
       this.setState({ msg: e.targe.value })
   }
   ```

6. children组合模式, 内容分发

   ```jsx
   // 组件内通过this.props.children挂载  相当于插槽
   <Welcome> <div>你好</div></Welcome>   
   // 组件内通过this.props.title和content 相当于获取到多个children
   <Welcome title={<h2>标题</h2>} content={<P>段落</P>} /> 
   
   ```

7. 复用组件RenderProps ★★★重要★★★

   ```jsx
   render(){ <div> {this.props.render(x,y)} </div> }   //  MouseXY组件 
   
   <MouseXY render={(x,y)=> (<div>鼠标坐标: {x},{y}</div>)}>
   ```

8. 跨组件通信, 深层传递context

9. 数组遍历挂载组件并传递参数

   ```jsx
   {list.map(v => <Item key={v.id} {...v} /></It>)}
   ```

10. 父子组件高阶函数绑定事件并传参

   ```jsx
   <div onClick={handleAdd(id)}></div>   // 子  //  需要注意避免不必要的提前执行
   handleAdd = id=> {   // 父  // 获取闭包id
       return() => {
           console.log("id:", id)
           // ...
       } }
   ```

##### `函数式组件`

1. 惰性初始值

   ```jsx
   //  initCount每次重新渲染都会被调用一次
   const initCount =　() => 345*345
   const [count, setCount] = useState(initCount())
   //   只调用一次
   const [count, setCount] = useState(() => initCount())
   ```

2. useEffect: 组件渲染Dom挂载后,异步执行

   ```jsx
   //  在初始mount和update时都会调用, return 回调函数,可以在update前和unmount前清理副作用
   useEffect(()=> {
       console.log("didMount or didUpdate")
       return () => {
           console.log(" beforeUpdate or willUnmount")
       }
   })
   // 只有初始化时执行一次
   useEffect(()=> {
     // ....
   }, [])
   // 添加依赖项 优化性能  符合规范
   useEffect(()=> {
       setInterval(()=> { setCount(count+1) 1000)
   }, [count])  // 此处依赖性count会不断变化导致多个interval累加
   // 正确处理   传入回调函数
   useEffect(() =>{
       setInterval(()=>{setCount(precount => preCount+1)},1000)
   }, [])
   ```

3. useLayoutEffect: 组件渲染时, dom挂载前,同步执行

4. ref绑定, forwardRef转发绑定

   ```jsx
   //  绑定原生dom
   const myRef = useRef()
   <div ref={myRef}>6666</div>
   // 组件绑定
   const Head = React.forwardRef((props,ref) => { return <div ref={props.myRef}>6666</div>})
   <Head ref={myRef} />
   // 给普通变量增加 记忆功能
   let count = useRef(0)
   count.current++
   // 只在组件更新时触发
   let isUpdate = useRef(false)
   useEffect(()=> {
       if(isUpdate.current){ 
           //..... 
       } 
   })
   ```

5. context 跨组件通信

   ```jsx
   let MyContext = React.createContext()
   <MyContext.Provider value="welcomer的问候~~~"> <Head /> </MyContext.Provider>
   const Head = () => <div><Title /></div>
   const Title = () => {
       let value = useContext(MyContext)
   }
   ```

6. memo包裹组件:  优化性能,数据不变时,跳过dom重新挂载

7. useCallback返回一个可记忆的函数, useMemo返回一个可记忆的值

   ```jsx
   // dom绑定点击事件函数,尽管函数名相同,但每次都会重新渲染组件, 从而生成一个函数名函数体相同但内存地址不同的函数
   // dom绑定函数变化了,dom就会重新挂载, 组件内的其他变量定义也是同理
   const foo = useCallback(()=>{aa: 77}, [])
   const bar = useMemo(()=> [2,3,56], [])
   const foo2 = useMemo(()=> ()=>{aa: 77}, [])  // 值为 一个返回 函数 的函数
   ```

8. useReducer: 复杂响应式数据的联合处理

9. startTransition: 非紧急任务; useTransition useDeferredValue延迟获得变量

10. rate评分组件的实现 ☆☆☆☆重要☆☆☆☆

    ```jsx
    // 代码实现
    ```

11. ErrorBoundary 错误边界处理

##### `react-router`

1. 动态路由 `"foo/:id"` 组建内使用`useParams`接收

2. 编程式路由跳转携带参数

   ```jsx
   navigate("/about", {state: {id: 3}})   // 
   const location = useLocation()   //  location.state
   ```

3. 获取查询参数 `useSearchParams`

   ```jsx
   const [searchParams, setSearchParams] = useSearchParams()
   searchParams.get('name')   // searchParams为map数据 
   ```

4. 路由loader函数,可用于初始数据获取,权限拦截, 异步执行

   ```jsx
   const data = useLoaderData()
   ```

5. 获取路由元信息meta

   ```jsx
   const location= useLocation()
   const match = matchRoutes(allRoutes,location)  // 最后一项为当前路由组件数据
   ```

   









