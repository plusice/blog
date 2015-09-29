title: es7-decorators
date: 2015-09-25 14:30:55
tags:
---

>近期一直在做react的项目，看了很多开源项目的代码，其中有一个`redux-react-router-async-example`的项目给我印象深刻，因为。。。代码看不懂啊！！！
首先，用react开发项目用webpack打包已经是业界惯例（当然也有用bowserify的），然后用了webpack基本你就会用babel-loader来编译你的jsx代码，
然后用了babel之后这年头的大神们怎么可能忍得住不用es6甚至是es7
的新特性？所以这些项目中的代码语法基本就是怎么新怎么来，苦了我这个逗逼码农，算了，还是研究研究es6和es7的语法吧~

关于es6，es7新增了哪些语法，大家可以参考babel的[LearnES2015](http://babeljs.io/docs/learn-es2015/)，
类似`template string`和`Let + Const`这些仅仅是加强了一下js原先一些不合理设计的没什么好说的，需要深入理解的可能就是：

* Promise
* Generators
* Class
* Map + Set + WeakMap + WeakSet
* decorators
* ......

一边写一边看，发现es7的新语法还是很多的，这要全列出来可能得很长一条，先把一些较为常用的列出来看看，反正我们今天讲的也就是*decorators* (￣_,￣ )

废话不多说，进入正题。

>decorators：因为整好看到`google developers`上有一篇很好的文章描述，我就不通篇写我自己的理解了，前文就直接翻译过来了，当然语言是会做一些自己的表达方式的。
先附上[原文链接](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)

## 探索ES2016装饰器 
js的装饰器不得不说是来源于python的，在python里面，装饰器是一个包含其他方法的方法，这个方法在不修改他包含的方法前提下扩展他，以此来达到装饰的作用。

在python中一个装饰器长这样

```python
@mydecorator
def myfunc():
      pass
```
(`@mydecorator`)就是一个装饰器，这看上去跟es7的装饰器并没有太大的区别。

`@`标识我们这边写的代码是需要编译成一个使用`mydecorator`作为装饰的装饰器方法，我们的装饰器把需要装饰的方法作为一个参数传入，并把他加工之后再返回。
装饰器有很多的应用场景，比如：记录，实施权限控制和认证，进行监控，记录日志等等。

## 在es5和es6中的装饰器
在es5中因为并没有原生的类的支持所以使用装饰器并没有那么重要。但是到了es6，因为有了类的支持，我们就需要一些在多个类之间分配一些工具方法的方式。

Yehuda（我并不知道他是谁）的装饰器希望能够通过注解改变Javascript的类，属性以及字面量。

## es7中的装饰器
es7中的装饰器就是返回一个函数的表达式，他接受三个参数：目标对象，属性名，以及一个属性描述符。
你可以通过把他放在你想要修饰的函数上面，并在最前面加上`@`符号来使用他。
装饰器可以设计用来修饰类或者属性。

## 装饰一个属性
我们先声明一个类
```javascript
class Cat {
      meow() {return `${this.name} says Meow!`;}
}
```
假如我们想要将`meow`方法放入到`Cat.prototype`上，大致上就像下面这样：
```javascript
Object.defineProperty(Cat.prototype, 'meow', {
   value: specifiedFunction,
   enumerable: false,
   configurable: true,
   writable:true   
});
```
想象一下如果我们想要让一个属性或者方法名称不能修改，我们需要声明一个装饰器放在这个属性或者方法声明前。所以我们声明了`@readonly`装饰器，如下：
```javascript
function readonly(target, key, descriptor) {
      descriptor.writable = false;
      return descriptor;
}
```
然后我们把它放在我们的`meow`方法上面
```javascript
class Cat {
      @readonly
      meow() {return `${this.name} says Meow!`;}
}
```
一个装饰器只是一个返回一个方法的表达式，所以`@readonly`和`@something(params)`都可以工作。
（这里解释一下，作者的意思是`readonly`表达式表示一个方法，而`something`也表示一个方法，
在`@something(params)`其实这个装饰器是调用了`something`这个方法，真正的装饰器是调用`something`之后返回的函数）

把这个装饰器挂到`meow`方法上之后，实际是执行了以下的代码：
```javascript
let descriptor = {
   value: specifiedFunction,
   enumerable: false,
   configurable: true,
   writable:true 
};

descriptor = readonly(Cat.prototype, 'meow', descriptor) || descriptor;
Object.defineProperty(Cat.prototypr, 'meow', descriptor);
```
现在`meow`方法现在是只读的了。我们可以通过以下代码来验证：
```javascript
var garfield = new Cat();
garfield.meow = function() {
      console.log('I want lasagne');
}

// Exception:attempted to assign to readonly property
```

小菜一碟？那么我们一起来看一下类的装饰器（中间略过了一段介绍一个装饰器库的内容，有兴趣的可以看原文）

## 装饰一个类
下一步我们一起来看一下如何装饰一个类。根据es7的建议标准，类的装饰器把类的构造方法作为`target`。
举个例子，我们定义一个`MySuperHero`类，我们在定义一个简单的装饰器`@superhero`：
```javascript
function subperhero(target) {
      target.isSuperhero = true;
      target.power = 'flight';
}

@superhero
class MySuperHero() {}

console.log(MysuperHero.isSuperhero); // true
```
我们还可以继续扩展，我们可以通过定义我们的装饰器是一个工厂来让我们的装饰器可以接受参数
```javascript
function subperhero(isSuperhero) {
      return function(target) {
            target.isSuperhero = isSuperhero;
      };
}

@superhero(true)
class MySuperheroClass() {}
console.log(MysuperHero.isSuperhero); // true

@superhero(false)
class MySuperheroClass() {}
console.log(MysuperHero.isSuperhero); // false
```
es7的装饰器可以作用在属性描述符和类之上。
他们自动获得属性名和目标对象，同时进行覆盖。调用描述符允许装饰器完成一些类似让一个属性变成一个`getter`方法，
启用一些用其他方式实现会显得很笨重的方法，比如第一次调用一个属性时自动绑定方法到当前对象。

## es7装饰器和mixins
我（作者）非常喜欢阅读*Reg Braithwaite’s*的那篇
[ES2016 Decorators as mixins](http://raganwald.com/2015/06/26/decorators-in-es7.html),
和
[Functional Mixins](http://raganwald.com/2015/06/17/functional-mixins.html)
Reg建议提出了一个把行为加入到目标（类属性或者独立对象）的帮助方法，如下：
```javascript
function mixin(behaviour, sharedBehaviour = {}) {
      const instanceKeys = Reflect.ownKeys(behaviour);
      const sharedkeys = Reflect.ownKeys(sharedBehaviour);
      const typeTag = Symbol('isa')  // 需要native实现
      
      function _mixin(clazz) {
            for (let property of instanceKeys) {
                  Object.defineProperty(clazz.property, property, {value: behaviour[property]});
            }
            Object.defineProperty(clazz.prototype, typeTag, {value: true});
            return clazz;
      }
      for (let property of sharedKeys) {
            Object.defineProperty(_mixin, property, {
                  value: sharedBehaviour[property],
                  enumerable:sharedBehaviour.propertyIsEnumerable(property)
            });
      }
      Obecj.defineProperty(_mixin, Symbol.hasInstance, {
            value:(i) => !!i[typeTag]
      });
      return _mixin;
}
```
非常好，现在我们可以定义一些混合方法并尝试去修饰一个类。想象一下我们有一个简单的`ComicBookCharacter`类
```javascript
class ComicBookCharacter {
      constructor(first, last) {
            this.firstName = first;
            this.lastName = last;
      }
      realName() {
            return this.firstName + ' ' + this.lastName;
      }
}
```
这可能是这世界上最无聊的字符了，但是现在我们定义一些混合方法来为这个类提供`SuperPowers`和`UtilityBelt`行为
```javascript
const SuperPowers = mixin({
      addPower(name) {
            this.powers().push(name);
            return this;
      },
      powers() {
            return this._powers_processed || (this._powers_pocessed = []);
      }
});

const UtilityBelt = mixin({
      addToBelt(name) {
            this.utilities().push(name);
            return this;
      },
      utilities() {
            return this._utility_items || (this._utility_items = []);
      }
});
```
现在我们可以用`@`来把这些装饰器用在我们的`ComicBookCharacter`类上面，
注意我们现在在一个类上用了两个装饰器
```javascript
@SuperPowers
@UtilityBelt
class ComicBookCharacter {
      ...
}
```
现在我们用我们刚才定义的来打造一个*蝙蝠侠*角色
```javascript
const batman = new ComicBookCharacter('Bruce', 'Wayne');
console.log(batman.realName());
// Bruce Wayne

batman.addToBelt('batarang').addToBelt('cape');

console.log(batman.utilites());
// ['batarang', 'cape']

batman.addPower('detective').addPower('voice sounds like Gollum has asthma');

console.log(batman.powers());
// ['detective', 'voice sounds like Gollum has asthma']
```

## 在Babel中使用装饰器
可以参考原文，这边说一下在webpack中babel使用装饰器的配置
```
{
      loader: 'babel-loader?stage=1'
}
```

## 后续
文章的后续介绍了一个有趣的实验以及为什么要去尝试使用装饰器，因为跟装饰器的理解没有太大的关系，所以就不翻译了。

## 我的理解
看到这里其实我对类的装饰器和方法的装饰器存在一些疑虑的。因为方法的装饰器返回的是一个描述符，
然后通过definePorperty把描述符挂载到方法上面。而类的装饰器貌似都是直接操作这个类了。
所以我又去看了一下babel对于装饰器的实现[链接在这里](https://github.com/wycats/javascript-decorators)
看一下对于类和方法装饰器不同的实现：

类：
```
@F("color")
@G
class Foo {
}

// 实现
var Foo = (function () {
  class Foo {
  }

  Foo = F("color")(Foo = G(Foo) || Foo) || Foo;
  return Foo;
})();
```

方法：
```
class Foo {
  @F("color")
  @G
  bar() { }
}

// 实现
var Foo = (function () {
  class Foo {
    bar() { }
  }

  var _temp;
  _temp = F("color")(Foo.prototype, "bar",
    _temp = G(Foo.prototype, "bar",
      _temp = Object.getOwnPropertyDescriptor(Foo.prototype, "bar")) || _temp) || _temp;
  if (_temp) Object.defineProperty(Foo.prototype, "bar", _temp);
  return Foo;
})();
```
好吧。。。大概的意思就是类的装饰器就真的只是把方法传入然后加工了一下，而方法的装饰器是用属性描述符来装饰。

至于为什么是这样。。。大概就是这么定义的吧O(∩_∩)O~，需要深入的时候再研究吧。




