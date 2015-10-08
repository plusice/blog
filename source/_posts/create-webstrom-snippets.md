title: 在webstrom中使用snippets
date: 2015-10-08 10:55:01
tags:
---
>snippets是一个提高我们开发效率非常有用的工具，一个snippets是一个有固定格式的代码片段，其中会包括一些需要用户修改的参数或者变量的地方。
这种效果非常类似于emmet（html开发非常有效率的工具），我最先知道snippets是在sublime当中，当我输入`func`并敲击键盘上的tab键之后，我的编辑器中就会显示如下代码：

```
function (params) {
	/* function code here */
}
```

>并且params会处于选中状态，我可以修改参数的名字，然后我再敲击tab，就会选中方法中的注释，这个时候我就可以写方法的具体内容了。
这就是snippets的使用方法，合理得利用好snippets可以非常有效得提高开发效率。

**今天我们要讲的则是如何创建一个snipperts，以及如何在webstrom中创建**

### webstrom创建snippets
首先我们打开setting，在搜索框中输入`live template`（webstrom中snippets叫这个名字），然后点击右侧的添加按钮

![创建一个snippets](http://7xn3gy.com1.z0.glb.clouddn.com/snippetscreate.png)

创建好之后，我们需要输入这个snippets的名字和描述，名字就是用来触发这个snippets的关键字

![snippets名字和描述](http://7xn3gy.com1.z0.glb.clouddn.com/snippetsname-and-desc.png)

然后我们在把代码写入编辑区

![snippets代码](http://7xn3gy.com1.z0.glb.clouddn.com/snippetscode-here.png)

### 创建snippets
我们来看一个snippets代码的样例：
```
for (var $INDEX$ = 0; $INDEX$ < $ARRAY$.length; $INDEX$++) {
  var $VAR$ = $ARRAY$[$INDEX$];
  $END$
}
```
其中这些用`$...$`包裹起来的字符串就是我们需要自定义的变量了，
上面的例子中，我们在使用这个snippets的时候首先光标会选中`$INDEX`区域，然后我们可以编辑这个`INDEX`的名字，一般我会命名为i，同时所有`$INDEX`区域都会变成我输入的新的变量名称，
然后我们敲击`tab`，光标会自动选中下一个变量名称，在这里就是`$ARRAY$`，依次下推直到最后一个。

到这里，基本我们已经知道如何去创建一个snippets了，赶紧去创建一个吧。

### snippets应用场景
snippets能够非常有效得提高编码效率，但如何有效得利用起来却是需要考虑一下的，至少我知道这东西两年了还是没有很好得利用起来，直到现在我才下定决心培养这个习惯。
下面我们看一下哪些场景我们可以创建一些snippets来帮助我们：

##### 常用代码片段
比如上面看到的`for`循环就是我们经常会用到的代码片段，这种片段经常会有，比如我用`react`的时候经常需要
```
let Name = React.createClass({
	
	render() {
		return ();
	}
	
});
```
那么这个代码片段我就可以创建一个snippets。

##### 项目规范代码
我们编码最终是为了项目服务，如果我们在开发一个项目，我们肯定会有我们的一套规范，比如我们的一个ajax请求需要传哪些参数，
跟上面不同，这种情况下更可能是一大段类似的代码，因为这是跟本身项目的一个架构相关的，所以我们更加能找到相同点

##### 配置
现在前端开发有一大堆的工具可以帮助我们进行开发和部署代码，而这些工具无一例外的都需要进行配置，而很多配置都是很类似的，那么这个时候我们就可以把这些配置创建一个snippets。

>以上是关于snippets的一些内容，记录这个的目的更多也是为了我自己能提高一些工作效率。从工作到现在，我都秉承一个道理，那就是能怎偷懒怎么偷懒╮(╯-╰)╭，毕竟写代码也是体力活啊~

