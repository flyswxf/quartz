Button普通使用：
``` 
Button('Button_A',{ type: ButtonType.Capsule, stateEffect: false })
 .onClick(()=>{
	 ...
 })
```
- 第一个参数为Button显示的名字
- type控制button形状
- stateEffect为true时，可以往button中添加子组件

```
Button({ type: ButtonType.Capsule, stateEffect: true }){
     Text('情商智力')  
       .border({ width:{bottom:4},color: this.selectedOption === 'eq' ? 0x6db4ff : 0xffffff})  
       .animation({ duration: 200, curve: Curve.EaseInOut })  
   }  
     .backgroundColor(Color.White)
```
- color中设置了一个条件判断语句，这配合animation属性，可以产生动画效果