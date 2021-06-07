### lazyMan

写出一个函数，能满足以下需求

- LazyMan('Hank')

打印出 你好我是Hank

- LazyMan('Hank').sleep(10).eat('lunch')

打印出 你好我是hank （沉默十秒）我刚睡了十秒 我在吃午餐

- LazyMan('Hank').eat('lunch').eat('supper')

打印出 你好我是hank 吃午餐 吃晚餐

- LazyMan('Hank').sleepFirst('5').eat('supper')

沉默五秒 打印出，你好我是hank， 吃晚饭

首先确定大致框架

```tsx
LazyMan = () =>{
	return {
		sleep:()=>{},
		eat:()=>{},
		sleepFirst:()=>{}
	}
}
```

队列+回调

```tsx
LazyMan = (name) =>{
var queue = []
const task = () => {console.log(`你好我是${name}`) next()}
queue.push(task)
const next = ()=>{
	const first = queue.shift()
	first?.()
}
const api = {
_x:queue
	sleep:(n)=>{
	const task = ()=> {
	setTimeout(()=>{
		console.log(`我刚睡了${n}秒钟`)
		next()
		},n*1000)	
	}
queue.push(task)
return api
},
		eat:(type)=>{
		const task = () => {console.log(`${type ==='lunch'?'吃午饭':'吃晚饭'}`);next()}
		queue.push(task)
		return api
	},
		sleepFirst:(n)=>{
				const task = ()=> {
				setTimeout(()=>{
					console.log(`我刚睡了${n}秒钟`)
					next()
			},n*1000)	
		}
		queue.unshift(task)
		return api
	}
}
	setTimeout(()=>{next()})
	return api
}
```
