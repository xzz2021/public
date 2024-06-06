### js高级语法学习笔记

1. `reflect` 弥补早期语言设计时的缺陷,obj对象被设计成了构造函数,估计希望未来明确区分,将obj逐渐转移到reflect上,从而将reflect明确作为真正的对象; 同时呼应proxy的设计,因为早期监听数据变动是曲线使用defineProperty, 而之后可以直接使用专门的proxy代理进行监听!
2. 