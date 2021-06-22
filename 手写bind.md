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
