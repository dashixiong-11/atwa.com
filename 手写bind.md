# atwa.com

```jsx
var slice = Array.prototype.slice
function bind(asThis){
  vat args = slice.call(arguments,1) // 获取除this 以外的剩余参数
  vat fn = this // 这里的this 指的是调用bind 的函数
  if(typeof fn !== 'function'){
    throw new Error("bind必须调用在函数上")
  }
  return function (){ //这个是 函数调用bind后返回的函数
    var args2 = slice.call(arguments,0) //拿到 this
    return fn.apply(asThis,args.concat(args2)) 
  }
}
```
#### 支持使用 new

##### new 的实际作用
```jsx
function P (){}
const p = new P()
//使用new 实际运行了以下代码
	this = {}
	this.__proto__ = P.prototype
	P.call(this)
	return this 
  //世界上 这边的 this 进行了两次改造， 第一次是使 this.__proto__ = P.prototype ,第二次是 把this 传入到 构造函数P中 让它继承 P的属性 ，然后再把改造后的 this return出去

// 任何东西 = new X() 都会让这个东西.__proto__ = X.prototype
```

```jsx
//为了便于理解我们用简易版 bind ，原理跟上面的一样
const bind2 = function(asThis,..args){
  var fn = this //谁调用bind 这里的this 就是谁, 而asThis 是需要绑定的this
  return function (...args2){
    return fn.call(asThis,...args,...args2)
  }
}

const fn = function(p1,p2){
  this.p1 = p1
  this.p2 = p2
}
const fn2 = fn.bind2(undefined,'x','y')
const ps =new fn2() //期待 ps.p1 = 'x' ps.p2 = 'y'

// 分析代码
//fn2 实际上是:
fn2 = function (){
  return fn.call(undefined,'x','y')
}
// 由于我们自己写了 return  所以 new 自己的return 是没用的 本来new return的是自己构造的this 结果却是 fn.call(undefined,'x','y') ,因为 this 是undefined 所以 this指向了window
```

```jsx
const bind2 = function(asThis,..args){
  var fn = this //谁调用bind 这里的this 就是谁, 而asThis 是需要绑定的this
  return function (...args2){
  // 需要判断是或否用 new 的this 还是 asThis 
    return fn.call(asThis,...args,...args2)
  }
}

```
#### 如何判断一个函数是被一般调用 还是被new 调用
```jsx

function P (){}
const p = new P()
//使用new 实际运行了以下代码
	this = {}
	this.__proto__ = P.prototype
	P.call(this)
	return this 
  
  //根据这个原理 我们只需要判断 this.__proto__ === P.prototype 是否成立就能知道是否使用了new
  
  const bind2 = function(asThis,..args){
  var fn = this //谁调用bind 这里的this 就是谁, 而asThis 是需要绑定的this
  return function resultFn(...args2){
    returun fn.call( this.__proto__ === resultFn.prototype ? this:asThis,...args,...args2) // 手动吧this传入 fn 中继承fn的属性 它自己也会执行 resultFn.call(this,...args,...args2) 因为resultFn 并没有构造this 这句代码无效 所以 this就继承了fn上的属性
   }
 }
 
 const fn = function(p1,p2){
  this.p1 = p1
  this.p2 = p2
}
const fn2 = fn.bind2(undefined,'x','y')
const ps =new fn2() // ps.p1 = 'x' ps.p2 = 'y'
 // 目前这个代码可以暂时实现我们的需求 
```
#### 使 fn.prototype = sayHi = ()=>{ console.log('hi')} 也能被继承

```jsx
  const bind2 = function(asThis,..args){
  var fn = this //谁调用bind 这里的this 就是谁, 而asThis 是需要绑定的this
   function resultFn(...args2){
    returun fn.call( this.__proto__ === resultFn.prototype ? this:asThis,...args,...args2) // 手动吧this传入 fn 中继承fn的属性 它自己也会执行       resultFn.call(this,...args,...args2) 因为resultFn 并没有构造this 这句代码无效 所以 this就继承了fn上的属性
   }
   resultFn.prototype = fn.prototype //因为new会让 this.__proto__ = resultFn.prototype 所以就把fn.prototype 赋值给 resultFn.prototype
   return resultFn
 }
```

#### 最终版
```jsx
var slice = Array.prototype.slice;
function bind(asThis) {
  // this 就是函数
  var args = slice.call(arguments, 1);
  var fn = this;
  if (typeof fn !== "function") {
    throw new Error("bind 必须调用在函数身上");
  }
  function resultFn() {
    var args2 = slice.call(arguments, 0);
    return fn.apply(
      resultFn.prototype.isPrototypeOf(this) ? this : asThis,
      args.concat(args2)
    );
  }
  resultFn.prototype = fn.prototype;
  return resultFn;
}

function _bind(asThis, ...args) {
  // this 就是函数
  var fn = this;
  function resultFn(...args2) {
    // resultFn.prototype.isPrototypeOf(this);
    return fn.call(this instanceof resultFn ? this : asThis, ...args, ...args2);
  }
  resultFn.prototype = fn.prototype;
  return resultFn;
}

module.exports = _bind;

if (!Function.prototype.bind) {
  Function.prototype.bind = _bind;
}
```
