title: 如何创建自己的webpack loader
date: 2016-04-15 13:16:46
tags: ['webpack', 'loader']
---

在目前的开源市场，前端架构中最火热的项目非webpack莫属了。在使用webpack的过程中，我们会用到各式各样的loader，毫无疑问，因为loader机制的存在让webpack拥有了无限的可能性，让webpack几乎可以容纳一切前端需要的资源。同时合理得利用loader也有助于我们在架构项目的时候省去很多重复工作，今天我们就来讲讲如何创建一个webpack的loader

> 在最开始想到要写loader的时候，其实我是拒绝的，因为webpack主要的功能是处理依赖以及编译，一提到编译我就头疼，各种字符串处理能让我上天。然而进一步了解之后我发现我想多了，大部分的时候编译的工作并不需要你来做，不多讲，看代码。

**首先**
你需要知道如何调试你本地的loader，幸运的是，不管是在`webpack.config.js`中写相对路径还是直接`require('./loader-name!<file path>')`webpack都是可以访问到我们的本地loader的，所以这点无需担心

**其次**
一个loader就是一个方法，这个方法接受一个`source`参数包括指定文件的内容，`this`包含了很多webpack的方法和属性供调用，该方法需要将你处理之后的内容返回，如果有sourcemap，也可以一并将sourcemap返回，这个时候需要调用`this.callback(null, source, map)`，第一个`null`代表没有错误，如果有错误的话就是一个`Error`对象

**所以**
一个loader大致长成这样
```
module.exports = function (source) {
    if (cacheable) this.cacheable()
    
    // do something about the source
    
    return dealedSource // 返回处理过的source
    // this.callback(null, dealedSource, map) // 如果有sourcemap
    
}
```
记住`cacheable`那一步必须要执行，一方面他可以提高webpack除第一次之外的编译熟读，再次如果有`cacheable`官方推荐是必须cacheable的，实践情况也是不执行的话会有奇葩错误，这点上因为webpack了解不深，同时也没有相关文档，所以不是很了解清楚（你知道webpack的源码多大么!!!!!）

**然后**
其实该讲的就已经讲完了。。。因为loader里面的处理逻辑是根据你的实际情况来的，这没什么好说的，比如less-loader里面就是调用了less把source处理一下然后return出去，所以想到什么的朋友应该已经可以动手写自己的loader了

**好吧，再说点什么**
在我遇到情况中，我需要在vue-loader之前做一些特定操作（通过demo生成文档），所以我先去研究了一下vue-loader的源码。vue-loader的操作逻辑我会重新起一片文章讲，到时候我再贴过来，我只想抱怨一下vue-loader真是一个大坑，因为vue-loader实际上调用两次文件的source，所以你在vue-loader之前对source做的任何操作都是**没什么卵用**，我了一整晚/(ㄒoㄒ)/~~

吐槽来了，[点这里](https://segmentfault.com/a/1190000004944322)

但在这种情况下，还是有办法处理，只是感觉有点hacker，并不是那么好，所以webpack得loader还是可以发挥你的很多想象力的。就酱，回家吃饭~~~


--------------- 4-15日补充 ----------------
发现一个重点，那就是如果你的loader处理的文件有依赖于别的文件，你必须在loader里面生命Dependency，不然的话很容易出现内容不更新等情况



