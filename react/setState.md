## setState

react中的状态state通过 setState() 方法来改变促进页面的重新渲染，但是在使用的时候每个人都会遇到各种不同的问题，我在网上看到很多人说setState的坑，吐槽的人也比较多，类似于下面这种问题，在我刚开始的时候也遇到过：

### 1. setState不会立即改变数据

```
// name is ''
this.setState({
     name: 'myName'
})
console.log('name is', this.state.name) // 输出 ？？？
```

这里当然是期望打印出改变后的state值，但是却并不能得到期望的值，为什么会这样呢？因为 setState 是一个异步执行的函数，所以这里输出的依然是改变之前的 state 值，其实 setState 提供给我们两个参数，第二个参数是一个回调函数，所以我们可以通过回调函数来获取到正确的值。

```
this.setState({
    name: 'myName'
}, () => {
    console.log(`name is ${this.state.name}`)
})
```
### 2. setState多次，Re-render 一次
以前刚接触react的时候，我一度认为每次 setState 都会造成一次 re-render ，其实并不是这样：
```
componentDidMount() {
this.setState((prevState, props) => ({count: this.state.count + 1})) // 1
this.setState((prevState, props) => ({count: this.state.count + 1})) // 2
this.setState((prevState, props) => ({count: this.state.count + 1})) // 3
this.setState({name: "xiaohesong"}) // 4
}
render() {
    console.log('render')
    return(
	// ...
    )
}
```
可以发现，这里打印出 ‘render’ 仅为两次，并不是 4+1 次，为什么？
我们之前说 `setState` 是一个异步执行的方法，其实是说当我们调用 `setState` 的时候，`setState` 是放在一个队列中异步处理的，也就是说他会把我们这四个 `setState`操作放到一个队列中，然后batch处理。
```
  this.setState((prevState, props) => ({count: this.state.count + 1})) // 1
  this.setState((prevState, props) => ({count: this.state.count + 1})) // 2
  this.setState((prevState, props) => ({count: this.state.count + 1})) // 3
  this.setState({name: "xiaohesong"}) // 4
```
如何批量操作的？？？setState方法是将传入的参数对象或函数返回的对象与现有的state对象进行合并，非常类似于使用Object.assign(prevState, newState)的效果
```
Object.assign(state,{count: this.state.count + 1},{count: this.state.count + 1}...{name: "xiaohesong"})
```
### 3. setState 造成没必要的渲染
第二点也告诉我们 setState 每次都会造成页面的重新渲染，但是很多时候，有些渲染不是必要的，不必要的渲染有以下几个原因：

+ 新的 state 其实和之前的是一样的。这个问题通常可以通过 shouldComponentUpdate 来解决。
+ 通常发生改变的 state 是和渲染有关的，但是也有例外。比如，有些数据是根据某些状态来显示的。
+ 第三，有些 state 和渲染一点关系都没有。有一些 state 可能是和事件、 timer ID 有关的。

#### 总结 ：
1. setState操作,默认情况下是每次调用, 都会re-render一次,除非你手动shouldComponentUpdate为false. react为了减少rerender的次数,会进行一个浅合并.将多次re-render减少到一次re-render.

2. setState之后,无法立即获取到this.state的值,是因为在setState的时候,他只会把操作放到队列里.
3. 和渲染无关的状态尽量不要放在 state 中来管理。
