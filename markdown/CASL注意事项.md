CASL注意事项

1. subject主体和它的类型彼此关联，就像对象实例和它的类彼此关联一样, 是因为检索了 `subject.constructor.name` 作为其主体类型

2. 规则的顺序很重要： `cannot` 声明应跟在 `can` 之后，否则将被 `can` 覆盖。

3. 传入的subject是泛类型时会跳过condition的检查

   ```ts
   can('update', 'Article', ['title', 'description'], { authorId: user.id });
   
   ability.can('update', ownArticle, 'title'); // 传入的是实体ownArticle true
   ability.can('update', anotherArticle, 'title'); //传入的是实体 anotherArticle false
   ability.can('update', 'Article', 'title'); //传入的是泛类型 'Article'   true!
   ```

4. 后端通常使用类来描述和封装领域逻辑作为数据, 而前端及node通常处理的是DTO对象（普通的 JS 对象）, 如果需要获得实体类就需要借助内置的 `subject` 辅助函数