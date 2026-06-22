有两个数组, arr是临时变量,任务需要将**arr并到list上**
```ts
class myClass{
	list:Array<number> = []
	updateList(arr:Array<number>){
	//arr:Array<number> = [1,2,3]
		this.list = this.list.concat(arr)
	}
}
```

- concat方法虽然是array本身的方法, 但是它**不会对list和arr进行改变**
- concat的**结果会作为返回值**, 因此需要令list=list.concat接收