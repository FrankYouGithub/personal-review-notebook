# Proxy

## 概述

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

## 参数

_target_
要使用 Proxy 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。

_handler_
一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 p 的行为。

## 语法

```
const p = new Proxy(target, handler)
```

```
var obj =newProxy({},{
  get:function(target, propKey, receiver){
    console.log(`getting ${propKey}!`);
    returnReflect.get(target, propKey, receiver);
  },
  set:function(target, propKey, value, receiver){
    console.log(`setting ${propKey}!`);
    returnReflect.set(target, propKey, value, receiver);
  }
});

obj.count =1
//  setting count!
++obj.count
//  getting count!
//  setting count!
//  2
```

上面代码对一个空对象架设了一层拦截，重定义了属性的读取（get）和设置（set）行为。

下面是另一个拦截读取属性行为的例子。

```
var proxy =newProxy({},{
  get:function(target, propKey){
    return35;
  }
});
proxy.time // 35
proxy.name // 35
proxy.title // 35

```

上面代码中，配置对象有一个 get 方法，用来拦截对目标对象属性的访问请求。get 方法的两个参数分别是目标对象和所要访问的属性。可以看到，由于拦截函数总是返回 35，所以访问任何属性都得到 35。

如果 handler 没有设置任何拦截，那就等同于直接通向原对象。

```
var target ={};
var handler ={};
var proxy =newProxy(target, handler);
proxy.a ='b';
target.a // "b"
```

同一个拦截器函数，可以设置拦截多个操作。

```
var handler ={
  get:function(target, name){
    if(name ==='prototype'){
      returnObject.prototype;
    }
    return'Hello, '+ name;
  },
  apply:function(target, thisBinding, args){
    return args[0];
  },
  construct:function(target, args){
    return{value: args[1]};
  }
};
var fproxy =newProxy(function(x, y){
  return x + y;
}, handler);
fproxy(1,2)// 1
new fproxy(1,2)// {value: 2}
fproxy.prototype ===Object.prototype // true
fproxy.foo ==="Hello, foo"// true
```

对于可以设置、但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果。

## 下面是 Proxy 支持的拦截操作一览，一共 13 种。

- get(target, propKey, receiver)：拦截对象属性的读取，比如 proxy.foo 和 proxy['foo']。
- set(target, propKey, value, receiver)：拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v，返回一个布尔值。
- has(target, propKey)：拦截 propKey in proxy 的操作，返回一个布尔值。
- deleteProperty(target, propKey)：拦截 delete proxy[propKey]的操作，返回一个布尔值。
- ownKeys(target)：拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
- getOwnPropertyDescriptor(target, propKey)：拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
- defineProperty(target, propKey, propDesc)：拦截 Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
- preventExtensions(target)：拦截 Object.preventExtensions(proxy)，返回一个布尔值。
- getPrototypeOf(target)：拦截 Object.getPrototypeOf(proxy)，返回一个对象。
- isExtensible(target)：拦截 Object.isExtensible(proxy)，返回一个布尔值。
- setPrototypeOf(target, proto)：拦截 Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
- construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)。

## 双向绑定

既然用 Proxy 能监控一个变量的读写情况，那么我们就很容易实现一个双向绑定了。

具体代码[看这里](https://jsbin.com/liwequkice/1/edit?html,js,console,output)。
