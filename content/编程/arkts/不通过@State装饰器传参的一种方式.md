使用后端异步请求获取变量, 已知可以通过@State接受变量, 再在子组件中通过@Prop,@Link接受变量, 实现动态渲染. 

本文介绍另一种可以在不适用@Prop接受参数的传参方式, 即,这种传参只需要
- 在子组件中申明参数(**但不用加@Prop**)
- 通过常规方式, ChildComponent(arg: T)传递参数
**这种方式尤其适用于Foreach渲染组件**, 因为Foreach中使用的**item不是@State修饰**, 无法通过常规方式传参

```ts
@State key:number = 0

func(){
	key=key+1;
	key=key-1;
}

build(){
	Foreach(func, (item:T)=>{
		ChildComponent(arg:T)
	})
}

```

解析:
- key是使用后端异步函数的返回值
- 当build进行第一次渲染时, 异步函数还没有返回值, 子组件使用在子组件内部定义的arg进行渲染
- 当异步函数返回时, 根据@State的特性, 对build中所有key相关的渲染进行更新.
- 更新会识别到Foreach中的func, 因此会重新做func, 再刷新Foreach(包括其中的子组件)

Q: **为什么不能把func放在aboutToAppear中**
A: aboutToAppear中的函数只在页面打开前运行, 此时异步函数还没有返回值, 使用func只会对空值就行操作. 后续异步函数回调函数触发后, key被更新, 但@State也不会对aboutToAppear中的内容进行更新渲染
