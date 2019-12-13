# 单例模式

::: tip 定义
保证一个类仅有一个实例，并提供一个访问它的全局访问点。js 的开发中，举个例子：点击登录按钮的时候，页面中会出现一个登录浮窗，这个登录窗口是惟一的，不管点击多少次登录按钮，都只会被创建一次，那么这个登录窗口就适合使用单例模式来创建。
:::

## 实现单例模式

想法：用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下次获取该类的实例的时候直接返回之前创建的对象

代码实现：

```js
var Singleton = function(name) {
  this.name = name
  this.instance = null
}

Singleton.prototype.getName = function() {
  alert(this.name)
}

Singleton.getInstance = function(name) {
  if (!this.instance) {
    this.instance = new Singleton(name)
  }
  return this.instance
}

var a = Singleton.getInstance("111")
var b = Singleton.getInstance("222")
alert(a === b) // true
```

或者

```js
var Singleton = function(name) {
  this.name = name
}
Singleton.prototype.getName = function() {
  alert(this.name)
}
Singleton.getInstance = (function() {
  var instance = null
  return function(name) {
    if (!instance) {
      instance = new Singleton(name)
    }
    return instance
  }
})()
```

如上述代码所示，可以通过 `Singleton.getInstance` 来获取 `Singleton` 类的唯一对象，这种方式相对简单，但是增加了这个类的“不透明性”，这里不能通过 `new` 来获取对象，只能通过 `Singleton.getInstance` 来获取对象。

## 透明的单例模式

在下面的例子中，将使用 `CreateDiv` 在页面中创建唯一的 div 节点。

```js
var CreateDiv = (function() {
  var instance
  var CreateDiv = function(html) {
    if (instance) {
      return instance
    }
    this.html = html
    this.init()
    return instance == this
  }
  CreateDiv.prototype.init = function() {
    var dic = document.createElement("div")
    div.innerHTML = this.html
    document.body.appendChild(div)
  }
  return CreateDiv
})()
var a = new CreateDiv("111")
var b = new CreateDiv("222")
alter(a == b) // true
```

如此便完成了一个透明的单例类的编写，但是它有一些缺点：

- 为了把 instance 封装起来，使用了自执行的匿名函数和闭包，并且让这个匿名函数返回真正的 Singleton 构造方法，增加了程序的复杂度。
- 假设某天需要这个类，在页面中创建千千万万的 div，必须改写 CreateDiv 的构造函数，把控制创建唯一对象的那段去掉。

## 用代理实现单例模式

通过引入代理类的方式，解决上面的问题。

首先在 CreateDiv 构造函数中，把负责管理单例的代码移除，使它成为一个普通的创建 div 的类

```js
var CreateDiv = function(html) {
  this.html = html
  this.init()
}
CreateDiv.prototype.init = function() {
  var div = document.createElement("div")
  div.innerHTML = this.html
  document.body.appendChild(div)
}
```

接下来，引入代理类 proxySingletonCreateDiv

```js
var ProxySingletonCreateDiv = (function() {
  var instence
  return function(html) {
    if (!instance) {
      instance = new CreateDiv(html)
    }
    return instance
  }
})()

var a = new ProxySingletonCreateDiv("111")
var b = new ProxySingletonCreateDiv("222")

alert(a === b) // true
```

通过引入代理类的方式，完成了一个单例模式的编写，现在将负责管理单例的逻辑移到了代理类 ProxySingletonCreateDiv 中，这样一来，CreateDiv 就变成了一个普通的类，跟 ProxySingletonCreateDiv 组合即可达到单例模式的效果。

本例是缓存代理的应用之一。

## JavaScript 中的单例模式

前面提到的几种单例模式的实现，更多的是接近传统面向对象语言中的实现，单例对象从“类”中创建而来。

但是 JavaScript 是一门无类语言，因此，生搬单例模式的概念并无什么意义。单例模式的核心是 **确保只有一个实例，并提供全局访问**。

全局变量不是单例模式，但是在 JavaScript 中经常会把全局变量当成单例来使用。只需要全局定义一个唯一的变量即可。但是使用这种方法存在很多问题，很容易造成命名空间污染。在大项目中，如果不限制，可能会存在很多这样的变量，而且很容易不小心被覆盖。作为开发者，要尽量减少全局变量的使用，即使需要，也要把它的污染降到最低。

相对降低全局变量带来的命名污染：

1. 使用命名空间

   适当的使用命名空间，不会杜绝全局变量，但可以减少全局变量的数量。

   最简单的方法是使用对象字面量的方式：

   ```js
   var namespace1 = {
     a: function() {
       alert(1)
     },
     b: function() {
       alert(2)
     }
   }
   ```

   将 a 和 b 都定义成 namespace1 的属性，可以减少变量和全局作用域打交道的机会。还可以动态的创建命名空间：

   ```js
   var MyApp = {}
   MyApp.namespace = function(name) {
     var parts = name.split(".")
     var current = MyApp
     for (var i in parts) {
       if (!current[parts[i]]) {
         current[parts[i]] = {}
       }
       current = current[parts[i]]
     }
   }
   MyApp.namespace("event")
   MyApp.namespace("dom.style")

   // 上述代码等价于
   var MyApp = {
     event: {},
     dom: {
       style: {}
     }
   }
   ```

2. 使用闭包封装私有变量

```js
var user = (function() {
  var __name = "niuniu"
  var __age = 18
  return {
    getUserInfo: function() {
      return __name + "-" + __age
    }
  }
})()
```

## 惰性单例

惰性单例指的是在需要的时候才创建对象实例。是单例模式的重点。第一节 instance 实例对象总是在调用 Singleton.getInstance 的时候才被创建，而不是在页面加载好的时候就被创建。

```js
Singleton.getInstance = (function() {
  var instance = null
  return function(name) {
    if (!instance) {
      instance = new Singleton(name)
    }
    return instance
  }
})()
```

但是这是基于“类”的单例模式，基于“类”的单例模式在 JavaScript 中并不适用。以 WebQQ 的登录浮窗为例，介绍与全局变量结合实现惰性的单例。

第一种解决方案：在页面加载完成的时候便创建好这个 div 浮窗，这个浮窗一开始肯定是隐藏状态的，当用户点击登录按钮的时候，它才开始显示:

```html
<html>
  <body>
    <button id="loginBtn">登录</button>
  </body>
  <script>
    var loginLayer = (function() {
      var div = document.createElement("div")
      div.innerHTML = "我是登录浮窗"
      div.style.display = "none"
      document.body.appendChild(div)
      return div
    })()
    document.getElementById("loginBtn").onclick = function() {
      loginLayer.style.display = "block"
    }
  </script>
</html>
```

这种方案有一个问题，可能打开的时候不需要登录，因为登录浮窗是一开始就创建好的，因此有可能浪费一些 DOM 节点。那么，用户点击的时候才开始创建该浮窗：

```html
<html>
  <body>
    <button id="loginBtn">登录</button>
  </body>
  <script>
    var createLoginLayer = function() {
      var div = document.createElement("div")
      div.innerHTML = "我是登录浮窗"
      div.style.display = "none"
      document.body.appendChild(div)
      return div
    }
    document.getElementById("loginBtn").onclick = function() {
      var loginLayer = createLoginLayer()
      loginLayer.style.display = "block"
    }
  </script>
</html>
```

那么，现在达到了惰性的目的，但是却失去了单例的效果。每次点击的时候创建的是一个新的浮窗。可以在关闭的时候将这个浮窗从页面中删除，但是这样频繁的创建和删除节点是不合理的，也是不必要的。

_可以使用一个变量来判断是否已经创建过登录浮窗_

```js
var createLoginLayer = (function() {
  var div
  return function() {
    if (!div) {
      div = document.createElement("div")
      div.innerHTML = "我是登录浮窗"
      div.style.display = "none"
      document.body.appendChild(div)
    }
    return div
  }
})()
document.getElementById("loginBtn").onclick = function() {
  var loginLayer = createLoginLayer()
  loginLayer.style.display = "block"
}
```

## 通用的惰性单例

上面的惰性单例还存在如下问题：
