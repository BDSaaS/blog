## 前言

js中有不少比较难以理解的概念，比如 **js原型** 和 **继承** 。我曾经很早的时候就看过js原型方面的知识，并在当时写了一篇 [博客](https://blog.csdn.net/dizuncainiao/article/details/79378700) 作为记录，很显然当时的我只是死记硬背。最近我利用空闲的时间将一些相对比较深入的js概念和用法重新学习，并新建了一个专栏 **深入javascript** 用于记录和分享。本篇来介绍  **如何实现在 js 中实现继承**：

## 概念

> 继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的属性和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

以上的概念还是非常好理解的，继承的好处就是实现代码的复用，简化程序的设计。对于 **js继承** 的学习我是在 [这个地址](https://zhuanlan.zhihu.com/p/37735247) 看的，关于 `ES5` 中如何实现继承这位大佬总结了5种方式，接下来我会结合其中一种谈谈我对于继承的理解。

## 举个🌰

如果你对于原型链理解的不够深，我墙裂推荐你先看完这一篇博文 [**深入 javascript 之 原型和原型链！！！**](https://blog.csdn.net/dizuncainiao/article/details/110123349) 再来看接下来的内容会好一点。

```javascript
function Animal(age) {
	this.age = age
}

Animal.prototype.eat = function () {
	console.log('每一种动物都会吃东西')
}

console.log(new Animal(10))
```

以上代码实现了一个最简单的 `动物类` ，它定义了每一种动物都有的一种属性 `age 年龄` 以及行为 `eat 吃` ，在谷歌浏览器打印代码看看👇：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205161108966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpenVuY2Fpbmlhbw==,size_16,color_FFFFFF,t_70#pic_center)
根据上图我们看到，`new Animal()` 可以调用 `Object构造函数` 的原型方法 `hasOwnProperty` ，而我们知道在 js 中获取对象的属性和方法时会在它的原型链上一层一层往上找，找得到则返回结果，找不到则返回 `undefined` 。实现基于原型链继承的关键就在于 —— <font color=red>**如何形成原型链?**</font>

## 如何形成原型链🤔

虽然 `__proto__` 属性已经被web标准废弃，但是对于我们初步理解 **基于原型链继承** 有很大的帮助。还是根据上面的图片来谈：

我们发现 `new Animal()` 有 `__proto__` 属性，它指向它的构造函数的原型 `Animal.prototype` ，`Animal.prototype` 也有一个 `__proto__` ，它也是指向它的构造函数的原型 `Object.prototype` 。

---

我们将 `Animal` 作为 **父类**，将 `Person` 作为 **子类** ，我们如何在 `Person.prototype` 中实现 `__proto__` 指向 Animal.prototype 。其实很简单，打印 `console.log(new Animal(18))` 👇：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020120516125326.png#pic_center)

`new Animal()` 拥有 `__proto__` 属性，我们尝试以下代码操作（<font color=#E6A23C>**Start**</font>）：

```javascript
function Animal(age) {
	this.age = age
}

Animal.prototype.eat = function () {
	console.log('每一种动物都会吃东西')
}

function Person () {}

Person.prototype = new Animal(18)
```

查看以下打印👇：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205161347686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpenVuY2Fpbmlhbw==,size_16,color_FFFFFF,t_70#pic_center)


（<font color=#E6A23C>**End**</font>）上图中 `new Person()` 基于原型链已经能够调用父类的 `eat` 方法了，但是这种直接的方式有很大的弊端：**无法基于子类每个的实例对象定制父类基本属性的值**、**父类中某个基本属性为引用类型时，子类实例调用该属性是共用一个内存地址** 等，还是看下例子吧：

```javascript
function Animal(age) {
    this.info = { name: 'foo' }
    this.age = age
}

Animal.prototype.eat = function () {
    console.log('每一种动物都会吃东西')
}

function Person() {}

Person.prototype = new Animal()
```

查看以下打印👇：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205161456123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpenVuY2Fpbmlhbw==,size_16,color_FFFFFF,t_70#pic_center)


可以清楚的看出，子类的实例对象修改 `info` 属性后导致其他实例的 `info` 属性一同被改了，这是因为它们从同一个内存地址取值的。如何解决这些弊端，实现 **js完美继承** ？接下来重点讲解：应该是 es6 之前所能实现的完美继承 —— <font color=red>**寄生组合式继承**</font>

（PS：请务必 **100%** 掌握 `call` 函数的用法，如果还不会请移步到我的这篇 [博客](https://blog.csdn.net/dizuncainiao/article/details/109990791)）

## 完美继承😎——寄生组合式继承

```javascript
// 父类 - 动物
function Animal(age) {
    this.age = age
    this.info = { description: '这是一只动物' }
}

Animal.prototype.eat = function () {
    console.log('每一种动物都会吃东西')
}

// 子类 - 人
function Person (name, sex) {
	this.name = name
	this.sex = sex
}
```

父类通常定义更通用的属性和方法来实现复用，子类则增加更具体的属性和方法来实现多样性。我们需要实现子类继承父类的属性和原型方法，并且子类生成实例对象时，可以定制父类的属性的值。一步一步来，先继承父类的实例属性：

```javascript
function Person (name, sex, age) {
	this.name = name
	this.sex = sex
    Animal.call(this, age)
}

const p1 = new Person('foo', '男', 18)
const p2 = new Person('bar', '女', 17)
```

构造函数 `Person` 内的 `this` 值为它的实例对象 `p1` ，通过 `call` 函数将 `Animal` 内的 `this` 替换成了 `p1` ，整个代码执行过程相当于：

```
p1.age = age
p1.info = { description: '这是一只动物' }
```

而由于 `this` 的值分别为 `p1` 和 `p2` ，所有就避免了 `info` 属性被污染的问题，验证一下👇：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020120516160822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpenVuY2Fpbmlhbw==,size_16,color_FFFFFF,t_70#pic_center)


借助于 `call` 函数的粗浅使用，我们实现了无污染的继承父类的实例属性。

---

下面我们来实现完美的继承父类的原型方法，首先看回到上面 **如何形成原型链** 代码中，就是 <font color=#E6A23C>**Start**</font> 到 <font color=#E6A23C>**End**</font> 这里，它存在以下弊端：

- `age` 等实例属性冗余
- 子类原型丢失 `constructor` 属性
- 两次调用 Animal()

很明显，这不符合我们的编程直觉，如何解决上面的弊端，让我们看下述代码：

```javascript
(function () {
    function Super() {}

    Super.prototype = Animal.prototype
    Person.prototype = new Super()
})()

Person.prototype.sayName = function () {
    console.log(`我的名字是：${this.name}`)
}

try {
    Object.defineProperty(Person.prototype, 'constructor', {value: Person})
} catch (err) {
    Person.prototype.constructor = Person
    console.warn(err.message);
}
```

以上代码将父类原型复制一份得到一个 `new Super()` ，它是一个纯净对象，没有父类的一些赋值语句，不掺杂父类的实例属性，并且它已经 **形成原型链** 。我们还给子类添加了原型方法，修复了子类原型的 `constructor` 属性。

## 完整代码

```javascript
// 父类 - 动物
function Animal(age) {
    this.age = age
    this.info = { description: '这是一只动物' }
}

Animal.prototype.eat = function () {
    console.log('每一种动物都会吃东西')
}

// 子类 - 人
function Person (name, sex, age) {
    this.name = name
    this.sex = sex
    Animal.call(this, age)
}

(function () {
    function Super() {}

    // 为什么不直接用 Person.prototype = new Animal()，因为这样会调用两次 Animal()，第一次在 Animal.call()
    // 通过将 父类的原型 复制出来，避免调用两次，摒除冗余属性
    Super.prototype = Animal.prototype
    Person.prototype = new Super()
})()

Person.prototype.sayName = function () {
    console.log(`我的名字是：${this.name}`)
}

// 添加 constructor 属性，使它指向其构造函数
try {
    Object.defineProperty(Person.prototype, 'constructor', {value: Person})
} catch (err) {
    Person.prototype.constructor = Person
    console.warn(err.message);
}
```

以上便是 **js寄生组合式继承** 代码的全部了，看下打印吧👇：

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-oKJ13Iwt-1607155551919)(E:\blog md\image-20201205154559324.png)\]](https://img-blog.csdnimg.cn/20201205161633507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpenVuY2Fpbmlhbw==,size_16,color_FFFFFF,t_70#pic_center)


## 总结

虽然现在的 **ES6** 中实现了更贴近于传统面向对象语言的 `class` 语法并能更简单的实现继承，但是在如今的框架层出不群、前端氛围略显浮躁的今天，理解传统的js继承方式以及更多的深入知识会让我们消除迷惘，掌握新技术也会更有底气。以上的博文绝大部分是我的个人理解，不能保证内容 100% 准确，欢迎讨论指正😝。

## 参考

[知乎 - js继承的几种方式](https://zhuanlan.zhihu.com/p/37735247)
