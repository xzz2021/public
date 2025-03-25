#### RenderProps用途及常用示例

1. 实现鼠标位置追踪

   ```jsx
   import React, { useState } from 'react';
   // 定义一个 Render Props 组件
   const MouseTracker = ({ render }) => {
     const [position, setPosition] = useState({ x: 0, y: 0 });
   
     const handleMouseMove = (event) => {
       setPosition({ x: event.clientX, y: event.clientY });
     };
     return (
       <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
         {render(position)}
       </div>
     );
   };
   // 使用该组件
   const App = () => {
     return (
       <MouseTracker
         render={({ x, y }) => (
           <h1>
             鼠标位置：({x}, {y})
           </h1>
         )}
       />
     );
   };
   export default App;
   ```

2. 封装异步数据加载

   ```jsx
   import React, { useState, useEffect } from 'react';
   const DataFetcher = ({ url, render }) => {
     const [data, setData] = useState(null);
     const [loading, setLoading] = useState(true);
   
     useEffect(() => {
       setLoading(true);
       fetch(url)
         .then((res) => res.json())
         .then((data) => {
           setData(data);
           setLoading(false);
         });
     }, [url]);
   
     return render({ data, loading });
   };
   // 使用该组件
   const App = () => {
     return (
       <DataFetcher
         url="https://jsonplaceholder.typicode.com/posts/1"
         render={({ data, loading }) =>
           loading ? <p>加载中...</p> : <div>标题: {data.title}</div>
         }
       />
     );
   };
   export default App;
   ```

3. 共享表单控件的输入逻辑

   ```jsx
   import React, { useState } from 'react';
   const InputHandler = ({ render }) => {
     const [value, setValue] = useState('');
     const handleChange = (e) => {
       setValue(e.target.value);
     };
     return render({ value, onChange: handleChange });
   };
   // 使用该组件
   const App = () => {
     return (
       <InputHandler
         render={({ value, onChange }) => (
           <div>
             <input type="text" value={value} onChange={onChange} />
             <p>输入内容: {value}</p>
           </div>
         )}
       />
     );
   };
   export default App;
   ```

   

