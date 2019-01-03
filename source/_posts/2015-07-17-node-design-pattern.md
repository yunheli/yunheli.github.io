---
layout: post
title: 常用的Node.js设计模式
date: 2015-07-17 00:00
tags: [nodejs]
---

当我们谈到设计模式的时候，你很可能会想到单例模式、观察者模式、工厂模式。本文并不会仅仅局限于介绍这些在Node编程中常见的设计模式，而且还会涉及到依赖注入、中间件等功能的介绍。

<!--more-->

## 什么是设计模式

> A design pattern is a general, reusable solution to a commonly occurring problem.

### 单例模式

单例模式将“类”的实例的个数限制为一个。在Node.js中创建单例模式非常的简单，只需要用`require`即可。

	//area.js
	var PI = Math.PI;

	function circle (radius) {
	  return radius * radius * PI;
	}

	module.exports.circle = circle;

无论你在应用中require这个模块多少次，这个模块的实例只会有一份存在。

	var areaCalc = require('./area');
	console.log(areaCalc.circle(5));

正因为`require`的这种行为，单例模式很可能是NPM模块中最常见的Node.js设计模式。

### 观察者模式

一个对象维护着一个依赖/观察者列表，并在状态改变的时候自动的通知列表中的每个成员。要实现观察者模式，可以借助于`EventEmitter`。

	// MyFancyObservable.js
	var util = require('util');
	var EventEmitter = require('events').EventEmitter;

	function MyFancyObservable() {
	  EventEmitter.call(this);
	}

	util.inherits(MyFancyObservable, EventEmitter);

这样我们就创建了一个可被观察的对象。为了让它更有用，我们可以为它增加点功能：

	MyFancyObservable.prototype.hello = function (name) {
	  this.emit('hello', name);
	};

太好了，现在我们的观测者可以发出事件（`emit event`）了，让我们来试下：

	var MyFancyObservable = require('MyFancyObservable');
	var observable = new MyFancyObservable();

	observable.on('hello', function (name) {
	  console.log(name);
	});

	observable.hello('john');

### 工厂模式

工厂模式使我们不需要使用构造器，而是通过提供一个泛型（通用）接口来创建对象。这种模式在创建过程变得复杂时会非常有用。

	function MyClass (options) {
	  this.options = options;
	}

	function create(options) {
	  // modify the options here if you want
	  return new MyClass(options);
	}

	module.exports.create = create;

工厂也使得测试变得更加简单，因为你可以通过这种模式来注入模块的依赖。

### 依赖注入

> Dependency injection is a software design pattern in which one or more dependencies (or services) are injected, or passed by reference, into a dependent object.

在下面的例子中，我们将创建一个获取数据库依赖的`UserModel`类。

	function userModel (options) {
	  var db;

	  if (!options.db) {
	    throw new Error('Options.db is required');
	  }

	  db = options.db;

	  return {
	    create: function (done) {
	      db.query('INSERT ...', done);
	    }
	  }
	}

	module.exports = userModel;

然后，我们可以使用如下方法创建实例：

	var db = require('./db');

	var userModel = require('User')({
	  db: db
	});

为什么这很有用？因为这使得测试变得简单，当你写单元测试的时候，你可以很容易的注入一个假`db`对象到你的模型中。

### Middlewares/ pipelines

中间件的概念很简单但却非常强大：一个单元/函数的输出是下一个的输入。如果你曾用过`Express`或`Koa`那么你肯定接触过。

我们来看看在Koa中是怎么做的：

	app.use = function(fn){
	  this.middleware.push(fn);
	  return this;
	};

也就是说，当你使用一个中间件的时候，它会被push到`middleware`数组中，这非常的赞，但是当请求到达服务器的时候发生了什么呢？

	var i = middleware.length;
	while (i--) {
	  next = middleware[i].call(this, next);
	}

原来没什么神奇的地方，你的中间件只不过是依次被循环遍历的调用了而已。

### Streams

你可以将流想象成一个特殊的管道。它们更擅长处理大量的流动数据，即使是它们是字节而不是对象。

	process.stdin.on('readable', function () {
	    var buf = process.stdin.read(3);
	    console.dir(buf);
	    process.stdin.read(0);
	});

调用

	$ (echo abc; sleep 1; echo def; sleep 1; echo ghi) | node consume2.js
	<Buffer 61 62 63>
	<Buffer 0a 64 65>
	<Buffer 66 0a 67>
	<Buffer 68 69 0a>

有一本好书你可以参考一下：[NodeJS Stream Handbook](https://github.com/substack/stream-handbook)

### References

1. 英文原文 https://blog.risingstack.com/fundamental-node-js-design-patterns/