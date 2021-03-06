# 难度等级4，难点或深度知识

## js解析过程

对于静态语言(Java、C++、C等)来说，有一个“编译器”，可以将写好的代码编译成另一种代码(如机器码,字节码)，
然后通过其它方法(比如汇编器转换汇编代码)将这些机器码变为机器可执行命令，这就是大致的代码运行过程。
而对于Javascript这样一种动态语言而言，有一个“解释器”，
可以直接动态解析代码，并将代码运行，输出结果。
你知道JS代码解析运行的流程么，请描述出来?

```js
如果一个文档流中包含多个Script代码段(用Script标签分隔的js代码或引入的js文件),他们的执行顺序是:

步骤1:读入第一个代码段(JS执行引擎并非一行一行的执行程序,而是一段一段的分析)
步骤2:做语法分析,有错则报语法错误(比如括号不匹配),并调到步骤5
步骤3:对var变量和function做’预解析’(先进行函数声明，而后进行变量声明（如果已经有函数声明，则变量声明无效），变量统一的默认值是undefined,)
步骤4:执行代码段(赋值也是在这个阶段),有错则报错(比如调用了null.属性就会报错)
步骤5:如果还有下一个代码段,则读入下一个代码段,重复步骤2
步骤6:结束
```

## JS变量声明，形参，函数声明的顺序与优先级?

填充变量的顺序：

函数形参 -> 函数声明 -> 变量声明
当变量声明遇到VO中已有同名时，不会影响已经存在的属性

函数形参：
由名称和对应值组成的一个变量对象的属性被创建
没有传对应参数的话，那么由名称和undefined组成的变量对象的属性会被创建

函数声明：
由名称和对应值（函数对象(function-object)）组成一个变量对象的属性会被创建
如果变量中已经有这个相同名称的属性，则完全替换

变量声明：
由名称和对应值（undefined）组成的一个变量对象的属性被创建
如果变量名称与已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的



##  js的内存回收机制

1.标记清除
主流浏览器实现的垃圾清除机制，最常用的垃圾回收方式

当变量进入环境时，例如，在函数中声明一个变量，就将这个变量标记为“进入环境”。
从逻辑上讲，永远不能释放进入环境的变量所占用的内存，因为只要执行流进入相应的环境，就可能会用到它们。
而当变量离开环境时，则将其标记为“离开环境”。

垃圾回收器在运行的时候会给存储在内存中的所有变量都加上标记（当然，可以使用任何标记方式）。
然后，它会去掉环境中的变量以及被环境中的变量引用的变量的标记（闭包，也就是说在环境中的以及相关引用的变量会被去除标记）。
而在此之后再被加上标记的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。
最后，垃圾回收器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。

到目前为止，IE、Firefox、Opera、Chrome、Safari的js实现使用的都是标记清除的垃圾回收策略或类似的策略，只不过垃圾收集的时间间隔互不相同。

2.引用计数
需要考虑循环引用问题

引用计数的含义是跟踪记录每个值被引用的次数。
当声明了一个变量并将一个引用类型值赋给该变量时，则这个值的引用次数就是1。
如果同一个值又被赋给另一个变量，则该值的引用次数加1。

相反，如果包含对这个值引用的变量又取得了另外一个值，则这个值的引用次数减1。
当这个值的引用次数变成0时，则说明没有办法再访问这个值了，因而就可以将其占用的内存空间回收回来。
这样，当垃圾回收器下次再运行时，它就会释放那些引用次数为0的值所占用的内存。

但是需要考虑两个对象循环引用的问题，容易造成内存泄漏

Javascript引擎基础GC方案是（simple GC）：mark and sweep（标记清除），即：
1）遍历所有可访问的对象。
2）回收已不可访问的对象。

GC的缺陷
和其他语言一样，javascript的GC策略也无法避免一个问题：
GC时，停止响应其他操作，这是为了安全考虑。
而Javascript的GC在100ms甚至以上，对一般的应用还好，
但对于JS游戏，动画对连贯性要求比较高的应用，就麻烦了。
这就是新引擎需要优化的点：避免GC造成的长时间停止响应。

GC优化策略
1.分代回收（Generation GC）
目的是通过区分“临时”与“持久”对象；
多回收“临时对象”区（young generation），
少回收“持久对象”区（tenured generation），减少每次需遍历的对象，从而减少每次GC的耗时。

2.增量GC
这个方案的思想很简单，就是“每次处理一点，下次再处理一点，如此类推”。
这种方案，虽然耗时短，但中断较多，带来了上下文切换频繁的问题。

因为每种方案都其适用场景和缺点，因此在实际应用中，会根据实际情况选择方案。

比如：低 (对象/s) 比率时，中断执行GC的频率，simple GC更低些；如果大量对象都是长期“存活”，则分代处理优势也不大。

像node v8引擎就是采用的分代回收（和java一样，作者是java虚拟机作者。）

## 最常见的两种攻击（XSS 和 CSRF）了解？

xss：
跨站脚本
说到底实际上是一个HTML注入问题
由于前端没有过滤输入，导致攻击者可以执行一些危险脚本
譬如输入文本实际为脚本执行代码，盗取cookie等敏感信息

主要的防御是：对输入进行限制


CSRF:
跨域伪造请求（XSS是csrf的诸多途径之一）
即冒充用户在站内操作
一般方法是通过 XSS 或链接欺骗等途径，让用户在本机(即拥有身份 cookie 的浏览器端)发起用户所不知道的请求。

要完成一次CSRF攻击，受害者必须依次完成两个步骤：
1.登录受信任网站A，并在本地生成Cookie。
2.在不登出A的情况下，访问危险网站B。

你也许会说：“如果我不满足以上两个条件中的一个，我就不会受到CSRF的攻击”。是的，确实如此，但你不能保证以下情况不会发生：
1.你不能保证你登录了一个网站后，不再打开一个tab页面并访问另外的网站。
2.你不能保证你关闭浏览器了后，你本地的Cookie立刻过期，你上次的会话已经结束。
(事实上，关闭浏览器不能结束一个会话，但大多数人都会错误的认为关闭浏览器就等于退出登录/结束会话了……)
3.所谓的攻击网站，可能是一个存在其他漏洞的可信任的经常被人访问的网站。


现在网站一般都会加上referer验证，可以防止一部分伪造
服务端一般三种策略：
1.验证HTTP Referer字段
2.在请求地址中添加token并验证
（譬如post中，以参数的形式加入一个随机产生的token）
3.在HTTP头中自定义属性并验证

## JS中var声明变量和绑定到window上的区别？

```js
var a = 1;

window.a = 1;
```

这两者有区别么？

**有区别**

虽然说乍看之下没区别，都能正常使用，但实际上从概念上来说要区分，
为了简单，以下只介绍`VO(globalContext)`

1. `var a = 1;`

    - 如果有这个语句，那么在进入这个执行上下文时，`VO(globalContext)`就已经有了属性`a`了
    
    - 就算是`let`也一样，有let声明，VO(globalContext)就会有这个属性，只不过直到`let`声明语句时这个属性才可用（所以let不存在hoist）
    
2. `window.a = 1;`

    - 如果有这个语句，那么在进入这个执行上下文时，`VO(globalContext)`**不会有属性`a`**
    
    - 直到执行到`window.a = 1;`时，才会在global中加入这个属性
    
而在函数作用域中，差别更甚

而之所以在全局作用域中，这两个看起来相等，是因为`VO(globalContext) === global`

## JS中VO和AO的区别

变量对象VO是与执行上下文相关的特殊对象,用来存储上下文的函数声明，函数形参和变量。

在global全局上下文中，变量对象也是全局对象自身，在函数上下文中，变量对象被表示为活动对象AO。

```js
VO(globalContext) === global
```

在函数执行上下文中，**VO是不能直接访问的**，此时由活动对象(activation object,缩写为AO)扮演VO的角色。

```js
VO(functionContext) === AO;
```

所以，这两者完全可以理解是同一个东西，只不过由于某些细微功能区别，分出了两个概念而已

## this与执行上下文

this是执行上下文中的一个属性：（和VO类似）

```js
activeExecutionContext = {
  VO: {...},
  this: thisValue,
  scopeChain: ...（还有一个作用域链，这里省略）,
};
```

**this与上下文中可执行代码的类型有直接关系，this值在进入上下文时确定，并且在上下文运行期间永久不变。**

所以，在全局代码中，this始终是全局对象本身，这样就有可能间接的引用到它了。

## 从JS的词法分析到AO

从解释器分析词法，到AO的生成（以函数作用域为例）

关键步骤：

1. 先分析参数

2. 分析函数声明（优先级高于变量提升）

3. 分析变量声明

步骤：

1. 函数的在进入可执行上下文的瞬间，生成一个活动对象（Active Object)就是所谓的AO

2. 分析参数

    - 函数接收参数（型参），添加到AO的属性上面，值全部都是undefined,如AO.age=undefined
    
    - 接收实参，形成AO对应的属性值
    
3. 分析函数的声明，如果funcion foo(){}

    - 如果AO中已经有这个变量，会覆盖，因此AO.foo为这个函数
    
4. 分析变量声明，如var age,

    - 如果AO上还没有age属性，则添加AO 属性，值是undefined
    
    - 如果AO 上面已经有了age属性,则不做任何操作。
    
5. 接下来就是其它的正常语句执行，变量赋值

关于函数声明和变量声明的先后参考来源：在《你不知道的JavaScript（上卷）》一书的第40页中写到：**函数会首先被提升，然后才是变量。**

简单理解：函数提升优先级比变量提升要高，且不会被变量声明覆盖，但是会被变量赋值覆盖

不过，你会发现，假设你理解的是，先变量提升，再函数声明，由于函数声明会覆盖变量，所以可能从代码角度上无法直接检测到区别。
（这里就先依参考来源中的概念来构建了）

### 执行栈的简单解释

进一步展开（这部分包含执行上下文概念，不理解可以先跳过）

- JS有一个`执行栈`）

- 浏览器首次载入脚本，它将创建`全局执行上下文`，并压入执行栈栈顶（不可被弹出）

- 然后每进入其它作用域就创建对应的执行上下文并把它压入执行栈的顶部

- 一旦对应的上下文执行完毕，就从栈顶弹出，并将上下文控制权交给当前的栈。

- 这样依次执行（最终都会回到全局执行上下文）

## 浏览器构建DOM树的大致步骤

1. 转码（Bytes -> Characters）

    - 读取接收到的 HTML 二进制数据，按指定编码格式将字节转换为 HTML 字符串
   
2. Tokens 化（Characters -> Tokens）

    - 解析 HTML，将 HTML 字符串转换为结构清晰的 Tokens，每个 Token 都有特殊的含义同时有自己的一套规则

3. 构建 Nodes（Tokens -> Nodes）

    - 每个 Node 都添加特定的属性（或属性访问器），通过指针能够确定 Node 的父、子、兄弟关系和所属 treeScope
    （例如：iframe 的 treeScope 与外层页面的 treeScope 不同）
    
4. 构建 DOM 树（Nodes -> DOM Tree）

    - 最重要的工作是建立起每个结点的父子兄弟关系
    
在 Chrome 开发者工具下 Timeline 面板的 Parse HTML 阶段对应着 DOM 树的构建。

浏览器构建CSSOM树的大致步骤也和构建 DOM 树一样需要这几步，不过最终的 CSSOM 树其实是用户代理样式（浏览器本来样式）与页面所有样式的重新计算

然后DOM 树与 CSSOM 树融合成渲染树

注意：渲染树只包括渲染页面需要的节点

- 排除 <script> <meta> 等功能化、非视觉节点

- 排除 display: none 的节点

