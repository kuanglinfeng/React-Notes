---
typora-copy-images-to: ./img
typora-root-url: ./img
---

# React-Notes
React Family Meals.



## React 核心概念



### JSX



#### JSX是什么？

- Fackbook起草的JS扩展语法
- 本质是一个JS对象，会被babel编译，最终会被转换为`React.createElement`
- 每个JSX表达式，有且仅有一个根节点
  - React.Fragment（空节点相当于`<></>`）
- 每个JSX元素必须结束（XML规范）



```jsx
<div className="App">
  <h1>hello</h1>
  <img />
</div>
```



#### 在JSX中嵌入表达式

- 将表达式作为内容的一部分
  - false，null和undefined不会显示
  - 普通对象不可作为子元素
  - 可以放置React元素对象
- 将表达式作为元素属性
- 属性使用小驼峰命名法
- 防止注入攻击
  - 自动编码
  - `dangerouslySetInnerHTML`

使用：

```jsx
function Me() {
  const person = { name: 'Flinn', friends: ['Leon', 'Monica'] }
  return (
    <div className="App">
      <h1>hello</h1>
      name: {person.name}
      <br />
      friends:
      <ul>{ person.friends.map(friend => <li>{friend}</li>) }</ul>
    </div>
  )
}
```

通常情况下，为防止注入攻击，React使用innerText进行页面的渲染，如有特殊需求，可使用`dangerouslySetInnerHTML`改为innerHTML：

```jsx
function App() {
  const content = '<h1>dangerous!!!</h1>'
  return (
    <div className="App" dangerouslySetInnerHTML={{__html: content}}>
    </div>
  )
}
```





#### 元素的不可变性



```jsx
const person = { name: 'Flinn', age: 21 }
const Person = (
  <div>
    {person.name}
    {person.age}
  </div>
)

// 报错：jsx对象上的属性是只读的
// 原理：Object.freeze(Person)
// 哲学：不断创建新的东西然后重新渲染才能看见页面变化
Person.props.children[0] = 'Leon' // TypeError

ReactDOM.render(<Person />, document.getElementById('root'))
```



### 组件



#### 创建一个组件

1. 函数组件
   1. 函数必须返回一个React元素
2. 类组件
   1. 必须继承`React.Component`
   2. 必须提供`render`函数，用于渲染组件
   3. render函数必须返回一个React元素

**组件的名称首字母必须大写的原因：如果小写，React会把组件当成普通的React元素去解析，从而解析失败。**

函数组件：

```jsx
const App = () => "hello"
ReactDOM.render(<App />, document.getElementById('root'))
```

类组件：

```jsx
import React from 'react'

class Person extends React.Component {

  render() {
    return (
      <div>
        hello
      </div>        
    )
  }
}
```



#### 组件的属性

1. 函数组件用参数props传递属性

```jsx
const Person = props => {
  // {name: 'Flinn', age: 21}
  console.log(props)
  return (
    <>
      name: { props.name }
      age: { props.age  }
    </>
  )
}

ReactDOM.render(<Person name="Flinn" age={21}/>, document.getElementById('root'))
```

2. 类组件用构造函数的参数传递属性

```jsx
class Person extends React.Component {

  constructor(props) {
    // {name: 'Flinn', age: 21}
    console.log(props)
    super(props) // 会执行：this.props = props 
    //  {name: 'Flinn', age: 21} {name: 'Flinn', age: 21} true
    console.log(this.props, props, this.props === props)
  }

  render() {
    return (
      <>
        name: {this.props.name}
        age: {this.props.age}
      </>
    )
  }
}
```

**注意：组件无法改变自身的属性，因为组件也是React元素**

React中的哲学：数据属于谁，谁才有权利改动。数据是自顶而下流动的。



#### 组件的状态

组件状态：组件可以自行维护的数据。

类组件中，状态（state）本质上是类组件的一个属性。

**状态初始化（必须）**

```jsx
constructor(props) {
  super(props)
  // 初始化
  this.state = {}
}
```

**状态的改变**：必须使用`this.setState()`，一旦调用了`this.setState()`会导致组件重新渲染

#### 深入理解setState()

```jsx
// 点击button一次，输出如下:
// render（初始化render）
// 0 (为啥是0不是1??，这说明改变state的过程很可能是异步的)
// render (state改变重新render)
class Counter extends React.Component {
  state = { count: 0 }

  handleClick = () => {
    this.setState({ count: this.state.count + 1 })
    console.log(this.state.count)
  }

  render() {
    console.log('render')
    return (
      <div>
        <span>{ this.state.count }</span>
        <button onClick={ this.handleClick }>+</button>
      </div>
    )
  }
}
```

结论：setState()，它对状态的改变，**可能**是异步的。

- 如果改变状态的代码处于某个HTML元素的事件中，则是异步的。否则是同步的。

- 实际开发中，我们只需要始终把setState当成是异步的就好了。(使用setState的回调函数，也就是其第二个参数)



**`this.setState`的参数**

1. `this.setState(stateObject)`

   参数stateObject是一个对象，这个对象会和之前的`this.state`对象进行混合（`Object.assign()`，只把相同属性覆盖）后重新赋值给`this.state`，从而改变`this.state`。

2. `this.setState(stateObject, callbackFunction)`

   这个callbackFunction是一个函数，它会在状态改变完成(render)之后执行。

3. `this.setState(fn, callbackFunction)`

   还可以接受第一个参数为函数，这个函数fn有一个参数为prevState（这个参数是可以信任的），fn的返回结果会混合到之前的state中。而且函数fn也是异步执行的。

   ```jsx
   this.setState(prevState => {
     ...
     return newState
   }, () => { 
     // 一些后续操作 
   })
   ```

   

演示1:

```jsx
// 倒计时组件
class Tick extends React.Component {
  constructor(props) {
    super(props)
    this.state = { left: this.props.number }
    this.timer = setInterval(() => {
      // 这种情况下的setState是同步的
      this.setState({ left: this.state.left - 1 })
      if (this.state.left === 0) {
        clearInterval(this.timer)
        this.timer = null
      }
    }, 1000)
  }

  render() {
    return <h1>倒计时剩余时间：{ this.state.left }</h1>
  }
}
```

演示2:

```jsx
// 点击button一次（预期输出：3），输出如下:
// 1
class Counter extends React.Component {
  state = { count: 0 }

  handleClick = () => {
    // setState异步执行 this.state.count + 1
    this.setState({ count: this.state.count + 1 })
    // 由于异步，执行时下面这行时，this.state.count 还是0
    this.setState({ count: this.state.count + 1 })
    // 由于异步，执行时下面这行时，this.state.count 还是0
    this.setState({ count: this.state.count + 1 })
    // 最终只会执行一次 this.state.count + 1
  }

  render() {
    return (
      <div>
        <span>{ this.state.count }</span>
        <button onClick={ this.handleClick }>+</button>
      </div>
    )
  }
}
```

使用函数参数改进实现演示2预期的效果：

```jsx
  handleClick = () => {
    this.setState(prevState => ({ count: prevState.count + 1 }))
    this.setState(prevState => ({ count: prevState.count + 1 }))
    this.setState(prevState => ({ count: prevState.count + 1 }))
    // 而且最终只会render一次
  }
```

React会对**异步的setState**进行优化，将多次setState进行合并（将多次状态改变完成后，再统一对state进行改变，然后触发render）

**setState最佳实践：**

1. 把所有的setState当做是异步的
2. 永远不要信任setState调用之后的状态
3. 如果要使用改变之后的状态，需要使用回调函数（setState的第二个参数）
4. 如果新的状态依赖之前的状态进行计算，使用函数的方式改变状态（setState的第一个参数）

#### 组件的数据

1. props: 该数据是由组件的使用者传递的数据，所有权不属于组件自身，因此组件无法改变该数据。
2. state：该数据是由组件自身创建的，所有权属于组件自身，因此组件有权改变该数据。



### 事件

在React中，事件的本质就是一个属性。

```jsx
const App = () => <button onClick={() => console.log('clicked')}>点击</button>
```

自定义React组件不会自带`onClick，onMouseMove等事件`，只用React自带的组件才有。

**如果没有特殊处理，在事件处理函数中，this指向undefined**

解决方式：

1. 使用bind函数，来绑定函数的this

   1. 在构造函数里面绑定（会将函数放到实例对象上）

      `this.xxx = this.xxx.bind(this)`

   2. 使用的时候绑定（函数在原型上）

      `onClick={ this.xxx.bind(this) } `

2. 使用箭头函数

   1. 声明函数的时候使用箭头函数

      `handleXXX = () => {}`

   2. 使用的时候使用箭头函数

      `onClick={ () => this.hanleXXX() }`

可以通过向子组件传递事件(函数)，使子组件调用父组件方法来改变父组件的数据。

```jsx
// 倒计时组件的父组件App
class App extends React.Component {
  
  constructor(props) {
    super(props)
    this.state = { isOver: false }
    // 将函数handleOver的this绑定为这个类的实例对象
    this.hanleOver = this.hanleOver.bind(this)
  }

  hanleOver() {
    this.setState({ isOver:true })
  }

  render() {
    return (
      <div>
        <Tick onOver={ this.hanleOver } />
        <h1>{ this.state.isOver ? '倒计时完成'  : '正在倒计时···' }</h1>
      </div>
    )
  }
}

// 倒计时组件
class Tick extends React.Component {
  constructor(props) {
    super(props)
    this.state = { left: this.props.number || 5, isOver: false }
    this.timer = setInterval(() => {
      this.setState({ left: this.state.left - 1 })
      if (this.state.left === 0) {
        clearInterval(this.timer)
        this.timer = null
        // 倒计时完成 通知父组件改变数据
        this.props.onOver()
      } 
    }, 1000)
  }

  render() {
    return (
      <div className="tick">
        <h1>倒计时剩余时间：{ this.state.left }</h1>
      </div>
    )
  }
}
```



#### 事件扩展

这里的事件：React内置的DOM组件中的事件

1. 给document注册事件
2. 几乎所有的元素的事件处理，都在document的事件中处理
   1. 一些不冒泡的事件，是直接在元素上监听的（如：input元素的onFocus事件是不冒泡的）
   2. 一些document上面没有的事件，直接在元素上监听
3. 在document的事件处理中，React会根据虚拟DOM树完成事件函数的调用
4. React的事件参数，并非真实的DOM事件参数，是React合成的一个对象，该对象类似于真实的DOM的事件参数
   1. `e.stopPropagation()`，可以阻止事件在虚拟DOM树中冒泡
   2. `e.nativeEvent`，可以得到真实的DOM的事件参数
   3. `e.nativeEvent.stopPropagation()`，可以阻止真实的DOM事件冒泡，不过这个函数没啥用，因为最终监听到事件的真实DOM都是document
   4. `e.nativeEvent.stopImmediatePropagation()`，可以阻止剩余的真实的DOM事件处理程序运行
   5. 为了提高执行效率，React使用事件对象池（多个e都是相同的引用）来处理事件对象

**注意事项**

1. 如果给真实的DOM注册事件，阻止了事件冒泡，则会导致React的相应事件无法触发
2. 如果给真实的DOM注册事件，事件会先于React事件运行
3. 通过React的事件中阻止事件冒泡，无法阻止真实的DOM事件冒泡
4. 可以通过`e.nativeEvent.stopImmediatePropagation()`，阻止document上剩余事件的执行
5. 在事件处理程序中，不要异步的使用事件对象e，因为e会被重用（事件对象池机制）

### 生命周期

生命周期：组件从诞生到销毁会经历一系列的过程，该过程就叫做生命周期。React在组件的生命周期中提供了一系列钩子函数，可以让开发者在函数中注入代码，这些代码就会在适当的时候运行。

**生命周期仅存在于类组件中，函数组件每次调用都是重新运行函数，旧的组件即刻被销毁**

#### 旧版生命周期函数

这里的旧版指的是React在16.0.0之前的版本。

![pre-lifecycle](./img/pre-lifecycle-9092468.png)

1. constructor
   1. 同一个组件对象只会调用一次constructor
   2. 不能在第一次挂载到页面之前，调用setState，为了避免bug，严禁在constructor中使用setState 
2. componentWillMount
   1. 正常情况下，和构造函数一样，它只会调用一次
   2. 可以使用setState，但是为了避免bug，不允许使用，因为在某些特殊情况下，该函数可能被调用多次（比如SSR，服务端调用一次，客户端也会调用一次）
3. **render**
   1. 返回一个虚拟DOM，会被挂载到虚拟DOM树中，最终渲染到页面的真实DOM中
   2. render可能不止运行一次，只要需要重新渲染，就会重新运行
   3. 严禁使用setState，因为可能导致无限递归渲染
4. **componentDidMount**
   1. 只会调用一次
   2. 可以使用setState
   3. 通常情况下，会将网络请求、启动计时器等一开始需要的操作，书写到该函数中
5. componentWillReceiveProps(newProps)
   1. 即将接收新的属性值时调用
   2. 参数为新的属性对象
   3. 该函数可能会导致一些bug，不推荐使用
6. **shouldComponentUpdate(newProps, newState)**
   1. 指示React是否要重新渲染该组件，通过返回true和false来指定
   2. 不显示声明该函数时，该函数默认返回true
   3. 该函数是一个性能优化点
7.  componentWillUpdate
8. componentDidUpdate(prevProps, prevState)
   1. 往往在该函数中使用DOM操作，改变元素
9. **componentWillUnmount**
   1. 通常在该函数中销毁一些组件依赖的资源，比如：计时器

#### 新版生命周期函数

这里的新版指的是React在16.0.0以及之后的版本。

![cur-lifecycle](./img/cur-lifecycle.png)

移除componentWillMount原因：有可能初始化会多次调用从而引发bug

移除componentWillReceiveProps原因：React官方认为，某个数据的来源必须是单一的，这个声明周期钩子很可能引发数据既受props影响又受state影响，这是一种反模式，很可能导致bug。而且还有很多开发者经常在这个钩子函数里面，使用this做一些骚操作，这样并不好，所以React官方干脆将替代函数`getDerivedStateFromProps`都设置为static的。

移除componentWillUpdate原因：这个钩子函数没啥用处

添加了`getDerivedStateFromProps(newProps, newState)`：

1. 当props或者state发生改变后调用
2. 通过参数可以获取新的属性和状态
3. 该函数是静态的
4. 该函数的返回值会覆盖掉组件状态
5. 这个钩子函数几乎没啥用，主要是为了替换掉getWillReceiveProps，减少骚操作

添加了`getSnapshotBeforeUpdate()`：

1. 真实的DOM构建完成，但还未实际渲染到页面中时调用
2. 在该函数中，通常用于实现一些绕过React的DOM操作
3. 该函数的返回值，会作为`componentDidUpdate`的第三个参数



### 传递元素内容

如果给自定义组件传递元素内容，则React会将元素内容作为children属性传递过去

```jsx
// props.children 就是组件Wrapper里包含的后代React元素
const Wrapper = props => (
  <div>
    <h1>Title</h1>
    {/* 下面一行显示的就是 <p>content</p> */}
    {props.children}
  </div>
)

export default class App extends React.Component {
  render() {
    return (
      <Wrapper>
        <p>content</p>
      </Wrapper>
    )
  }
}
```



### 表单

受控组件：组件的使用者，有能力完全控制该组件的行为和内容。通常情况下，受控组件往往没有自身的状态，其内容完全由收到的属性控制。函数组件往往就是一种受控组件。

非受控组件：组件的使用者，没有能力控制该组件的行为和内容，组件的行为和内容完全自行控制，往往是没有属性只有状态的组件

**表单组件，默认情况下是非受控组件，一旦设置了表单组件的value属性，则其变为受控组件**



## React 进阶



### 属性默认值和类型检查



#### 属性默认值

通过一个静态属性`defaultProps`告知React属性默认值

函数组件

```jsx
// 函数组件
const App = (props) => {
  return ( 
    <div>hello {props.name}</div>
  )
}
// 默认属性
App.defaultProps = {
  name: 'Flinn'
}
```

类组件

```jsx
class App extends Component {
  // 这样也可以
  static defaultProps = {
    name: 'Leon'
  }
	
	constructor(props) {
    // 在super之前props就已经完成了对默认props的混合
    super(props)
  }

  render() {
    return (
      <div>
        hello {this.props.name}
      </div>
    )
  }
}

// App.defaultProps = {
//   name: 'Flinn'
// }
```



#### 属性类型检查

使用库：`prop-types`

对属性使用静态属性`propTypes`告知React如何检查属性

propTypes支持的类型如下：

```jsx
PropTypes.any // 任意类型
PropTypes.array // 数组类型
PropTypes.bool // 布尔类型
PropTypes.func // 函数类型
PropTypes.number // 数字类型
PropTypes.object // 对象类型
PropTypes.string // 字符串类型
PropTypes.symbol // 符号类型

PropTypes.node // 任何可以被渲染的内容，如：字符串、数字、React元素
PropTypes.element // react元素
PropTypes.elementType, // 组件类型
PropTypes.instanceof (构造函数) // 必须是指定构造函数的实例
PropTypes.oneOf([xxx, xxx]) // 枚举
PropTypes.arrayOf(PropTypes.XXX) // 必须是某一类型组成的数组
PropTypes.objectOf(PropTypes.XXX) // 对象由某一类型的值组成

PropTypes.exact({...}) // 对象必须精确匹配传递的数据

// 自定义属性检查，如果有错误，返回错误对象即可
属性：function (props, propName, componentName) {
  // ...
}
```

演示：

```jsx
export default class ValidationComp extends React.Component {
	
  // 先混合属性
  static defaultProps = {
    count: 0
  }
  
  // 再对ValidationComp组件所要接受的props进行类型约束
  static propTypes = {
    count: PropTypes.number
    // 如果count是必选的:
    // count: PropTypes.number.isRequired
  }

  render() {
    return (
      <div>
        <span>{ this.props.count }</span>
        <button>+</button>
      </div>
    )
  }
}
```



### 高阶组件

HOF（Higher Order Funciton）: 高阶函数，以函数为参数，并且返回一个函数

HOC（Higher Order Component）: 高阶组件，以组件作为参数，并且返回一个组件

通常，可以利用HOC实现横切关注点

> 举例：20个组件，每个组件在创建组件和销毁组件时，需要做日志记录

> 举例：20个组件，它们需要显示一些内容，得到的数据结构完全一致

这样就可以写一个高阶组件将共同的逻辑抽离出来



写一个简单的记录日志的高阶组件：

```jsx
// 输出日志的高阶组件
export default function (Comp) {
  return class LogWrapper extends React.Component {

    componentDidMount() {
      console.log(`日志：组件${Comp.name}被创建了！${Date.now()}`)
    }

    componentWillUnmount() {
      console.log(`日志：组件${Comp.name}被销毁了！${Date.now()}`)
    }

    render() {
      return <Comp />
    }
  }
}

// 组件A
class A extends Component {
  render() {
    return <h1>A</h1>
  }
}

// 组件B
class B extends Component {
  render() {
    return <h1>B</h1>
  }
}

// 具有输出日志功能的组件A
const ALog = withLog(A)

// 具有输出日志功能的组件B
const BLog = withLog(B)
```

注意：

1. 不要在`render函数`里面使用高阶组件
2. 不要在高阶组件内部更改传入的组件



### ref

ref（reference）: 引用

场景：希望直接使用DOM元素中的某个方法，或者希望直接使用自定义组件中的某个方法

1. ref作用于内置的html组件，得到的将是真实的DOM对象
2. ref作用于类组件，得到的将是类的实例
3. ref不能作用于函数组件

**ref不再推荐使用字符串赋值（效率问题，且不够灵活），字符串赋值的方式将来可能会被移除**

**目前，ref推荐使用对象或者是函数**

字符串ref（不推荐）：

```jsx
class A extends Component {

  state = { isClicked: false }

  handleClick() {
    this.setState({ isClicked: true })
  }

  render() {
    return (
      <h2>
        { this.state.isClicked ? 'component A is clicked!!!' : null }
      </h2>
    )
  }
}

class B extends Component {

  click = () => {
    this.refs.inputBox.focus()
    this.refs.compA.handleClick()
  }

  render() {
    return (
      <div>
        <input type="text" ref={ 'inputBox' }/>
        <A ref={ 'compA' }/>
        <button onClick={ this.click }>点击B组件(相当于点击了A组件，同时对输入框聚焦)</button>
      </div>
    )
  }
}


class App extends Component {

  render() {
    return (
      <div>
        <A/>
        <B/>
      </div>
    )
  }
}
```



#### **对象**

通过`React.createRef`创建

演示：

```jsx
class App extends Component {

  constructor(props) {
    super(props)
    this.inputBox = React.createRef()
  }

  handleClick = () => {
    this.inputBox.current.focus()
  }

  render() {
    return (
      <div>
        <input type="text" ref={this.inputBox}/>
        <button onClick={this.handleClick}>聚焦</button>
      </div>
    )
  }
}
```



#### 函数

函数的调用时间：

1. `componentDidMount`的之前会调用该函数

   1. `componentDidMount`中可以使用ref了

2. 如果ref的值发生了变动（旧的函数被新的函数替代），分别调用旧的函数和新的函数，时间点出现在`componentDidUpdate`之前

   1. 旧的函数被调用时，传递null

   2. 新的函数被调用时，传递对象

   3. 示例：

      ```jsx
      class App extends Component {
      
        handleClick = () => {
          this.inputBox.focus()
        }
      
        render() {
          return (
            <div>
              {/*下面这行代码可能会调用多次，因为每次重新render的时候传进去的函数是不同的*/}
              <input type="text" ref={(element) => {this.inputBox = element}}/>
              <button onClick={this.handleClick}>聚焦</button>
            </div>
          )
        }
      }
      ```

      

3. **如果只是想保存一份引用可以如下操作**：

   ```jsx
   class App extends Component {
   
   
     handleClick = () => {
       this.inputBox.focus()
     }
   
     getRef = element => {
       this.inputBox = element
     }
   
     render() {
       return (
         <div>
           <input type="text" ref={ this.getRef }/>
           <button onClick={ this.handleClick }>聚焦</button>
         </div>
       )
     }
   }
   ```

4. 如果ref所在的组件被卸载，会调用该函数，传递null

**谨慎使用ref，ref其实是一种反模式，和React的理念是不符的。能够使用属性和状态进行控制，就不要使用ref**



#### ref转发

有时候，我们需要在函数组件的内部引用ref，这时就需要使用ref转发了(`React.forwardRef`)

`forwardRef方法`：

1. 参数，传递的是函数组件，不能是类组件，并且函数组件需要有第二个参数来得到ref
2. 返回值是一个新的组件

演示：

```jsx
function A(props, ref) {
  return (
    <h1 ref={ref}>
      组件A
      <span>{ props.words }</span>
    </h1>
  )
}

// 传递函数组件A 得到一个新组件NewA
const NewA = React.forwardRef(A)

class App extends Component {

  ARef = React.createRef()

  componentDidMount() {
    // {current: h1}
    console.log(this.ARef)
  }

  render() {
    return (
      <div>
        <NewA ref={ this.ARef } words={ 'i am a component' }/>
      </div>
    )
  }
}

ReactDOM.render(<App/>, document.getElementById('root'))
```

高阶组件中转发ref:

```jsx
// 输出日志记录的高阶组件
const withLog = Comp => {
  class LogWrapper extends Component {
    componentDidMount() {
      console.log(`日志：组件${ Comp.name }被创建了！${ Date.now() }`)
    }

    componentWillUnmount() {
      console.log(`日志：组件${ Comp.name }被销毁了！${ Date.now() }`)
    }

    render() {
      // rest 代表正常的属性
      // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
      const { forwardedRef, ...rest } = this.props
      return <Comp { ...rest } ref={ forwardedRef }/>
    }
  }

  // 注意 React.forwardRef 回调的第二个参数 “ref”。
  // 我们可以将其作为常规 prop 属性传递给 LogWrapper，例如 “forwardedRef”
  // 然后它就可以被挂载到被 LogWrapper 包裹的子组件上。
  return React.forwardRef((props, ref) => {
    return <LogWrapper {...props} forwardedRef={ref} />
  })
}

// 组件A 是一个类组件
class A extends Component {
  render() {
    return (
      <h1>
        组件A
        <span>{ this.props.words }</span>
      </h1>
    )
  }
}

// 高阶组件
const AHoc = withLog(A)

class App extends Component {
  myRef = React.createRef()

  componentDidMount() {
    // 这样就是{ current: A } 而不是{ current: AHoc } 
    console.log(this.myRef)
  }

  render() {
    return (
      <div>
        <AHoc words={ 'hello react' } ref={ this.myRef }/>
      </div>
    )
  }
}

ReactDOM.render(<App/>, document.getElementById('root'))
```



### context

上下文：context，表示做某一些事情的环境

React上下文特点：

1. 当某个组件创建了上下文后，上下文中的数据，会被所有后代组件共享
2. 如果某个组件依赖了上下文，会导致该组件不再纯粹（因为外部数据仅来自于props）
3. 一般情况下，用于第三方组件（通用组件）

![context](/context.png)

#### 旧的API

**创建上下文**

只有类组件才可以创建上下文

1. 给类组件书写静态属性`childContextTypes`，使用该属性对上下文中的数据类型进行约束
2. 添加实例方法`getChildContext`，该方法返回的对象，即为上下文中的数据，该数据必须满足类型约束，该方法会在每次render之后运行

**使用上下文中的数据**

要求：如果要使用上下中的数据，组件必须有一个静态属性`contextTypes`，该属性描述了需要获取的上下文中的数据类型

1. 可以在组件的构造函数中，通过第二个参数，获取上下文数据
2. **从组件的context属性中获取**
3. 在函数组件中，通过第二个参数，获取上下文数据

演示：

```jsx
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export default class OldContext extends Component{

  static childContextTypes = {
    count: PropTypes.number
  }

  getChildContext() {
    console.log('获取上下文中的数据')
    return {
      count: 99
    }
  }

  render() {
    return (
      <div>
        <ChildA />
      </div>
    )
  }
}


const ChildA = (props, context) => (
  <div>
    <h1>ChildA</h1>
    <p>ChildA Context Data: {context.count}</p>
    <ChildB />
  </div>
)

// 函数组件获取context 也要预先声明contextTypes为静态属性
ChildA.contextTypes = {
  count: PropTypes.number
}


class ChildB extends Component {

  // 声明需要使用那些上下文中的数据
  static contextTypes = {
    count: PropTypes.number
  }

  constructor(props, context) {
    // 将参数的上下文交给父类处理
    super(props, context)
    // { count: 99 }
    console.log(this.context)
  }

  render() {
    return (
      <div>
        <h1>ChildB</h1>
        ChildB Context Data: { this.context.count }
      </div>
    )
  }
}
```



**上下文的数据变化**

上下文的数据变化不可以直接变化，最终都是通过状态改变

在上下文中加入一函数，可以用于后代组件更改上下文中的数据

演示：

```jsx
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export default class OldContext extends Component {

  static childContextTypes = {
    count: PropTypes.number,
    changeCount: PropTypes.func
  }

  state = {
    count: 999
  }

  getChildContext() {
    console.log('获取上下文中的数据')
    return {
      count: this.state.count,
      changeCount: (newCount) => {
        this.setState({ count: newCount })
      }
    }
  }

  render() {
    return (
      <div>
        <ChildA/>
        {/*点击后 上下文中的数据将会加1*/ }
        <button onClick={ () => this.setState({ count: this.state.count + 1 }) }>+</button>
      </div>
    )
  }
}


const ChildA = (props, context) => (
  <div>
    <h1>ChildA</h1>
    <p>ChildA Context Data: { context.count }</p>
    <ChildB/>
  </div>
)

// 函数组件获取context 也要预先声明contextTypes为静态属性
ChildA.contextTypes = {
  count: PropTypes.number
}


class ChildB extends Component {

  // 声明需要使用那些上下文中的数据
  static contextTypes = {
    count: PropTypes.number,
    changeCount: PropTypes.func
  }

  constructor(props, context) {
    // 将参数的上下文交给父类处理
    super(props, context)
    // { count: 99 }
    console.log(this.context)
  }

  render() {
    return (
      <div>
        <h1>ChildB</h1>
        ChildB Context Data: { this.context.count }
        <button onClick={ () => this.context.changeCount(this.context.count + 2)}>子组件改变context count+2</button>
      </div>
    )
  }
}
```



#### 新版API

旧版API存在严重的效率问题，并且容易导致滥用

新版API原理：

![newContext](/newContext.png)

**创建上下文**

上下文是一个独立于组件的对象，该对象通过`React.createContext()创建`

返回的是一个包含两个属性的对象

1. Provider属性：生产者，一个组件，该组件会创建一个上下文，该组件有一个value属性，通过该属性为其数据赋值
   1. 同一个Provider，不要用到多个组件中，如果需要在其他组件中使用该数据，应该考虑将数据提升到更高的层次
2. Consumer属性：消费者，一个组件，它的子节点是一个函数（它的props.children需要传递一个函数），函数参数为Provider中设置的属性value，返回值为渲染的内容



**使用上下文中的数据**

1. 在类组件中，可以直接使用this.context获取上下文数据
   1. 要求：必须拥有静态属性`contextType`，应赋值为创建的上下文对象

2. 在函数组件中，需要使用Consumer来获取上下文数据
   1. Consumer是一个组件
   2. 它的子节点是一个函数（它的props.children需要传递一个函数）

**注意细节**

如果，上下文提供者（Context.Provider）中的value属性发生变化（地址引用不一样，Object.is()比较），会导致该上下文提供的所有后代元素全部重新渲染，无论该子元素是否有优化（无论shouldComponentUpdate函数返回什么结果）

类组件中使用Provider：

```jsx
import React, { Component } from 'react'
import PropTypes from 'prop-types'

const ctx = React.createContext()

export default class NewContext extends React.Component {

  state = {
    count: 999,
    message: 'hello new context',
    changeCount: newCount => {
      this.setState({count: newCount})
    }
  }

  render() {
    const Provider = ctx.Provider
    return (
      <Provider value={ this.state }>
        <div>
          <ChildA />
        </div>
      </Provider>
    )
  }
}

class ChildA extends Component {

  static contextType = ctx

  render() {
    return (
      <div>
        <h1>ChildA</h1>
        <p>来自上下文的数据：count: {this.context.count}，message: {this.context.message}</p>
        <ChildB />
      </div>
    )
  }
}

class ChildB extends Component {

  static contextType = ctx

  render() {
    return (
      <div>
        <h1>ChildB</h1>
        <p>来自上下文的数据：count: {this.context.count}，message: {this.context.message}</p>
        <button onClick={() => this.context.changeCount(this.context.count + 1)}>count+1</button>
      </div>
    )
  }
}
```

函数组件中使用Consumer:

```jsx
import React, { Component } from 'react'
import PropTypes from 'prop-types'

const ctx = React.createContext()

export default class NewContext extends React.Component {

  state = {
    count: 999,
    message: 'hello new context',
    changeCount: newCount => {
      this.setState({ count: newCount })
    }
  }

  render() {
    const Provider = ctx.Provider
    return (
      <Provider value={ this.state }>
        <div>
          <ChildA/>
        </div>
      </Provider>
    )
  }
}

// 函数组件中使用Consumer
function ChildA(props) {
  return (
    <div>
      <h1>ChildA</h1>
      <div>
        <ctx.Consumer>
          {/*页面显示: 999, hello new context*/}
          { value => <>{ value.count }, { value.message }</> }
        </ctx.Consumer>
      </div>
      <ChildB/>
    </div>
  )
}


class ChildB extends Component {

  static contextType = ctx

  render() {
    return (
      <div>
        <h1>ChildB</h1>
        <p>来自上下文的数据：count: { this.context.count }，message: { this.context.message }</p>
        <button onClick={ () => this.context.changeCount(this.context.count + 1) }>count+1</button>
      </div>
    )
  }
}
```



### PureComponent

纯组件，用于避免不必要的渲染（运行render函数），主要对生命周期函数`shouldComponentUpdate`进行优化，从而提高效率

优化：如果一个组件的属性和状态，都没有发生变化，重新渲染该组件是没有必要的

PureComponent是一个组件，如果某个组件继承自该组件，则该组件的`shouldComponentUpdate`会进行优化，将当前的state和下一个state、当前的props和下一个props进行浅比较，如果相等则不会重新渲染

用法：

```jsx
import React, { PureComponent } from 'react'
import PropTypes from 'prop-types'

// 浅比较
function objectEqual(obj1, obj2) {
  for (let prop in obj1) {
    if (!Object.is(obj1[prop], obj2[prop])) {
      return false
    }
  }
  return true
}

// 继承PureComponent 相当于写了shouldComponentUpdate中的比较代码 可以大大的减少组件不必要的render次数
export default class Task extends PureComponent {

  static propTypes = {
    // 任务名称
    name: PropTypes.string.isRequired,
    // 任务是否完成
    isFinished: PropTypes.bool.isRequired
  }

  // shouldComponentUpdate(nextProps, nextState, nextContext) {
  //   if (objectEqual(this.props, nextProps) && objectEqual(this.state, nextState)) {
  //     return false
  //   }
  //   return true
  // }

  render() {
    console.log('Task Render')
    return (
      <li className={ this.props.isFinished ? 'finished' : '' }>
        { this.props.name }
      </li>
    )
  }
}
```

**注意**

1. PureComponent进行的是浅比较
   1. 为了效率，应该尽量使用PureComponent
   2. 要求不要改动之前的状态，永远是创建新的状态覆盖原来的状态（Immutable，不可变对象）
   3. 有一个第三方JS库，`Immutable.js`，它专门用于制作不可变对象
2. 如果是函数组件，使用`React.memo`函数制作纯组件

memo制作函数纯组件演示：

```jsx
import React, { PureComponent } from 'react'
import PropTypes from 'prop-types'
import './Task.css'
import { objectEqual } from '../utils/helper'


function Task(props) {
  console.log('Task Render')
  return (
    <li className={ props.isFinished ? 'finished' : '' }>
      { props.name }
    </li>
  )
}

Task.propTypes = {
  name: PropTypes.string.isRequired,
  isFinished: PropTypes.bool.isRequired
}

// Task变为纯组件了
export default React.memo(Task)

// 模拟实现React.memo
function memo(FunctionComponent) {
  return class Memo extends PureComponent {
    render() {
      return (
        <>
          { FunctionComponent(this.props) }
        </>
      )
    }
  }
}
// 效果是一样的
//export default memo(Task)
```



### render props

有时候，某些组件的各种功能及其处理逻辑完全一样，只是显示的部分界面不一样，建议下面的方式任选其一来解决重复代码的问题（横切关注点）

1. render props
   1. 某个组件，需要某个属性
   2. 该属性是一个函数，函数的返回值用于渲染
   3. 函数的参数会传递为需要的数据
   4. 注意纯组件的属性（尽量避免每次传递的render props 的地址不一致）
   5. 通常该属性的名字叫做`render`
2. HOC

render props演示：

```jsx
////////// MouseListener.js 文件
import React, { PureComponent } from 'react'
import './style.css'

/**
 * 该组件用于监听鼠标的变化
 */
class MouseListener extends PureComponent {
  state = {
    x: 0,
    y: 0
  }

  divRef = React.createRef()

  handleMouseMove = e => {
    const {left, top} = this.divRef.current.getBoundingClientRect()
    const x = e.clientX - left
    const y = e.clientY - top
    // 更新x、y的值   x、y是相对与panel左上定点的坐标
    this.setState({x, y})
  }

  render() {
    return (
      <div ref={this.divRef} className={ 'point' } onMouseMove={ this.handleMouseMove }>
        {/* 只对调用props.render()  可以渲染panel内部 有差异的UI */}
        { this.props.render ? this.props.render(this.state) : null }
      </div>
    )
  }
}

export default MouseListener

/////////// index.js文件

// 地址不改变，这样能保证是纯组件
function renderPoint(mouse) {
  return <>横坐标：{ mouse.x }, 纵坐标：{ mouse.y }</>
}

// 地址不改变，这样能保证是纯组件
function renderDiv(mouse) {
  return (
  	<>
     <div style={{ width: 100, height: 100, background: '#008c8c', position: 'absolute', left: mouse.x - 50, top: mouse.y - 50 }}/>
    </>
  )
}

const App = () => {
  return (
 	  <div>
      {/* 显示坐标的简单效果 */}
      <MouseListener render={ renderPoint } />
			{/* div跟随鼠标坐标变化而移动的效果 */}
      <MouseListener render={ renderDiv }/>
    </div>
  )
}
```



### Portals

Portals(插槽)：将一个React元素，渲染到指定的DOM容器中

`ReactDOM.createPortal(React元素，真实的DOM容器)`，该函数返回一个React元素

`ReactDOM.createPortal`用法演示：

```jsx
///////// index.html 结构
<body>
  <div id="root"></div>
  <div class="modal"></div>
</body>

///////// index.js 

const App = () => {
  return (
    <div>
      <ChildA/>
    </div>
  )
}

// 在真实的DOM渲染中，会将<div className={ 'child-a' }><ChildB/></div>渲染到
// 类名为'modal'的真实DOM中
// 但是，React组件结构不会发生改变 依旧是:
// <App>
//	 <ChildA>
//      <ChildB />
//   <ChildA/>
// </App>
const ChildA = () => ReactDOM.createPortal(
  <div className={ 'child-a' }><ChildB/></div>, document.querySelector('.modal')
)

const ChildB = () => (<div className={ 'child-b' }/>)

ReactDOM.render(<App/>, document.getElementById('root'))
```

真实DOM：

![s1](./img/s1.png)

组件结构：

![s2](./img/s2.png)

**注意事件冒泡**

1. React中的事件是包装过的
2. 它的事件冒泡是根据虚拟DOM树来冒泡的，与真实的DOM树无关



### 错误边界

默认情况下，若一个组件在**渲染期间**（render）发生错误，会导致整个组件树全部被卸载

错误边界：是一个组件，该组件会捕获到渲染期间（render）子组件发生的错误，并有能力阻止错误继续往根组件方向传播

**让某个组件捕获到错误**

1. 编写生命周期函数`getDerivedStateFromError`
   1. 静态函数
   2. 运行时间点：渲染子组件的过程中，发生错误之后，在更新页面之前
   3. **注意：只有子组件发生错误，才会运行该函数**
   4. 该函数返回一个对象，React会将该对象的属性覆盖掉当前组件的state
   5. 参数：错误对象
   6. 通常，该函数用于改变状态，可以根据状态做出相应的显示
2. 编写生命周期函数`componentDidCatch`
   1. 实例方法
   2. 运行时间点：渲染子组件的过程中，发生错误之后，更新页面之后
   3. **由于其运行时间点比较靠后，因此不太会在该函数中改变状态**
   4. 通常，该函数用于记录错误消息

**细节**

某些错误，错误边界组件无法捕捉

1. 组件自身的错误
2. 异步的错误
3. 事件中的错误

总结：仅处理渲染子组件期间的同步错误

演示：

```jsx
////////////////  App.js
import React from 'react'
import ErrorBound from './components/ErrorBound'

export default function App() {
  return (
    <div>
      {/* 这是一个错误边界组件 其子组件发生错误时 可进行相应的错误处理*/}
      <ErrorBound>
        <A />
      </ErrorBound>
      <B />
    </div>
  )
}

function A() {
  return (
    <div>
      <h1>A</h1>
      <AChild />
    </div>
  )
}

function B() {
  return <h1>B</h1>
}

function AChild() {
  // 这里抛出一个错误 如果不处理将会导致整个组件树被销毁
  throw new Error('error')
  return <h2>A's Child</h2>
}


/////////////////  ErrorBound.js 错误边界组件
import React, { PureComponent } from 'react'

class ErrorBound extends PureComponent {

  state = {
    hasError: false
  }

  // 运行时间点：渲染子组件的过程中，发生错误之后，在更新页面之前
  static getDerivedStateFromError(error) {
    console.log('发生了错误，错误对象', error)
    return {
      hasError: true
    }
  }

  // 运行时间点：渲染子组件的过程中，发生错误之后，在更新页面之后
  // componentDidCatch(error, errorInfo) {
  //   console.log(error, errorInfo)
  //   this.setState({
  //     hasError: true
  //   })
  // }

  render() {
    if (this.state.hasError) {
      return <h1>出现了错误</h1>
    } else {
      return this.props.children
    }
  }
}

export default ErrorBound
```



### 渲染原理

渲染：生成用于显示的对象，以及将这些对象形成真实的DOM对象

- React元素：React Element,通过`React.createElement`创建（语法糖：JSX ）
  - 例如：
  - ```<div><h1>hello react</h1></div>```
  - ```<App />```
- React节点：专门用于渲染到UI界面的对象，React会通过React元素，创建React节点，ReactDOM一定是通过React节点来渲染的
- React节点类型：
  - React DOM节点：由`React.createElement`创建，创建该节点的React元素类型(type)是一个字符串
  - React 组件节点：由`React.createElement`创建，创建该节点的React元素类型是一个函数或者是类
  - React 文本节点：由字符串、数字创建
  - React 空节点：由null、undefined、false、true创建
  - React 数组节点：由一个数组创建
- 真实DOM：通过document.createElement创建的dom元素

![xuanran](./img/xuanran.png)



#### 首次渲染（新节点渲染）

1. 通过参数的值创建节点
2. 根据不同的节点，做不同的事情
   1. 文本节点：通过`document.createTextNode`创建真实的文本节点
   2. 空节点：什么都不做
   3. 数组节点：遍历数组，将数组每一项递归创建节点（回到1进行反复操作，直到结束）
   4. DOM节点：通过`document.createElement`创建真实的DOM元素，设置该真实DOM元素的各种属性，然后遍历对应的React元素的children属性，递归操作（回到1进行反复操作，直到结束）
   5. 组件节点
      1. 函数组件：调用函数（该函数必须返回一个可以生成React节点的内容），将该函数的返回结果递归生成节点（回到1进行反复操作，直到结束）
      2. 类组件：
         1. 创建该类的实例
         2. 立即调用对象的生命周期方法：`static getDerivedStateFromProps`
         3. 运行该对象的render方法，拿到节点对象（将该节点递归操作，回到1进行反复操作）
         4. 将该组件的`componentDidMount`加入到执行队列（先进先执行），当整个虚拟DOM树全部构建完毕，并且将真实的DOM对象加入到真实的DOM容器中后，执行该队列
3. 生成出虚拟DOM树之后，将该树保存起来，方便后续使用
4. 将之前生成的真实的DOM对象，加入到容器中



演示：

```jsx
const App = (
	<div className='app'>
  	<h1>
    	标题
      { ['abc', null, <p>段落</p>] }
    </h1>
    <p>
    	{ undefined }
    </p>
  </div>
)
ReactDOM.render(App, document.getElementById('root'))
```

以上代码，生成的虚拟DOM树：

![tree](./img/tree.png)



```jsx
function App(props) {
  return (
    <div>
      <Comp1 n={5} />
    </div>
  )
}

function Comp1(props) {
  return <h1>Comp1, { props.n }</h1>
}

ReactDOM.render(<App />, document.getElementById('root'))
```

以上代码，生成的虚拟DOM树：

![func-comp-tree](./img/func-comp-tree.png)



```jsx
class Comp1 extends Component {

  render() {
    return (
      <h1>Comp1</h1>
    )
  }
}

class App extends Component {

  render() {
    return (
      <div>
        <Comp1 />
      </div>
    )
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```

以上代码，生成的虚拟DOM树：

![class-comp-tree](./img/class-comp-tree.png)



#### 更新节点

更新的场景：

1. 重新调用`ReactDOM.render`，完全重新生成节点树
   1. 触发根节点更新
2. 在类组件的实例对象中调用`setState`，会导致该实例所在的节点更新

##### 节点的更新

- 如果调用的是`ReactDOM.render`，进入根节点的**对比（diff）更新**
- 如果调用的是`setState`
  - 1. 运行生命周期函数，`static getDerivedStateFromProps`
  - 2. 运行`shouldComponentUpdate`，如果该函数返回false，终止当前流程
  - 3. 运行render，得到一个新的节点，进入该新的节点的**对比更新**
  - 4. 运行生命周期函数`getSnapshotBeforeUpdate`加入执行队列，以待将来执行
  - 5. 运行生命周期函数`componentDidUpate`加入执行队列，以待将来执行

后续步骤：

1. 完成真实的DOM更新
2. 依次调用执行队列中的`componentDidMount`
3. 依次调用执行队列中的`getSnapshotBeforeUpdate`
4. 依次调用执行队列中的`componentDidUpate`
5. 依次调用执行队列中的`componentWillUnmount`



##### 对比更新



将新产生的节点，对比之前虚拟DOM中的节点，发现差异，完成更新

问题：对比之前DOM树的哪个节点

React为了提高对比效率，做出以下假设：

1. 假设节点不会出现层级的移动（对比时，直接找到旧树中对应位置的节点进行对比）
2. 不同的节点类型会生成不同的结构
   1. 相同的节点类型：节点本身的类型相同，如果是由React元素生成，type值还必须一致
   2. 其它的，都属于不同的节点类型
3. 多个兄弟通过唯一标识（key）来确定对比的新节点（详细解释在下面）



###### 找到了对比的目标

判断节点类型是否一致

- **一致**

根据不同的节点类型，做不同的事情

**空节点**：不做任何事情

**DOM节点**：

1. 直接重用之前的真实DOM对象
2. 将其属性的变化记录下，以待将来统一完成更新 （现在不会真正的变化）
3.  遍历该新的React元素的子元素，**递归对比更新**

**文本节点**：

1. 直接重用之前的真实DOM对象
2. 将新的文本变化记录下来，将来统一完成更新

**组件节点**：

函数组件：重新调用函数，得到一个节点对象，进入**递归对比更新**

类组件：

1. 重用之前的实例
2. 调用生命周期方法`getDerivedStateFromProps`
3. 调用生命周期方法`shouldComponentUpdate`，若该方法返回false，终止
4. 运行render，得到新的节点对象，进入**递归对比更新**
5. 将该对象的`getSnapshotBeforeUpdate`加入队列
6. 将该对象的`componentDidUpdate`加入队列

**数组节点**：遍历数组进行**递归对比更新**



- **不一致**



###### 没有找到对比的目标



### 工具

#### 严格模式

StrictMode（`React.StrictMode`），本质是一个组件，该组件不进行UI渲染，它的作用是在渲染内部组件时，发现不合适的代码

如：

- 识别不安全的生命周期
- 关于使用过时字符串`ref API`的警告
- 关于使用废弃的`findDOMNode`方法的警告
- 检测意外的副作用（React要求副作用代码仅出现在`componentDidMount`、`componentDidUpdate`、`componentWillUnmout`中）
  - 副作用：一个函数中，做了一些会影响函数外部数据的事情
  - 副作用举例：异步操作、改变参数值、setState、本地存储、改变函数外部的变量
  - 没有副作用的函数：纯函数
  - 在严格模式下，虽然React不能监控到具体的副作用代码，但它会将不能具有副作用的函数调用两遍，以便发现问题（这种情况，仅在开发模式下有效）
- 检测过时的`context API`

#### Profiler

性能分析工具，`React Dev Tools扩展工具里提供的`

分析某一次或多次提交（更新），涉及到组件的渲染时间

火焰图：得到某一次提交，每个组件总的渲染时间以及自身的渲染时间

排序图：得到某一次提交，每个组件自身渲染时间的排序

组件图：某一个组件，在多次提交中，自身渲染花费的时间



## Hooks

Hooks是React16.8.0版本推出的api，用来解决函数组件中功能不足的问题

组件：无状态组件（函数组件）、类组件

类组件中的麻烦：

1. this指向问题
2. 繁琐的生命周期
3. 其它问题

Hook专门用于增强函数组件的功能(Hook在类组件中是不能使用的)，使之理论上可以成为类组件的替代品

官方强调：没有必要更改已经完成的类组件，目前没有计划取消类组件

Hook（钩子）本质上是一个函数（命名上总是以use开头），该函数可以挂载任何功能

Hook种类：

1. useState(解决函数组件中无法使用状态的问题)
2. useEffect(解决函数组件中无法使用生命周期的问题)
3. 其它...

注意：使用Hook的时候，如果没有严格按照Hook的规则进行，eslint的一个插件`eslint-plugin-react-hooks`会报出警告



### State Hook

#### 用法

State Hook是一个在函数组件中使用的函数（useState），用于在函数组件中使用状态

useState函数有一个参数，这个参数的值表示状态的默认值

- 函数有一个参数，这个参数的值表示状态的默认值
- 函数的返回值是一个数组，该数组一定包含两项
  - 第一项: 状态的值
  - 第二项: 一个函数，用来改变状态

基本用法1：

```jsx
import React, { useState } from 'react'

export default function App() {
  // 使用一个状态，该状态的默认值是0
  const [count, setCount] = useState(0)
  
  return (
    <div>
			<span>{ count }</span>
			<button onClick={ () => setCount(count + 1) }>+</button>
		</div>
  )
}
```

一个函数组件中可以有多个状态，这种做法有利于横向切分关注点

```jsx
import React, { useState } from 'react'

export default function App() {
  // 使用一个状态，该状态的默认值是0
  const [count, setCount] = useState(0)
  // 是否可见
  const [visible, setVisible] = useState(true)
  return (
    <div>
      <p style={{ display: visible ? "block" : "none" }}>
        <span>{ count }</span>
        <button onClick={() => setCount(count + 1)}>+</button>
      </p>
      <button onClick={() => setVisible(!visible)}>显示/隐藏</button>
    </div>
  )
}
```



#### 原理

![state-hook](./img/state-hook.png)

**注意的细节**

1. useState最好写到函数的起始位置，便于阅读，如果某些状态之间没什么必然的联系，应该分化为不同的状态（解耦）
2. **useState严禁出现在判断、循环的代码块中（这样会导致状态表对应不上）**
3. useState返回的函数，为节约内存空间，引用不会变化
4. **和类组件setState不同，useState中使用函数改变数据，若数据和之前的数据完全相等（使用Object.is比较），不会导致重新渲染，这样可以提升性能**
5. **和类组件setState不同，使用函数改变数据，传入的值不会和原来的数据进行合并，而是直接替换**
6. 和类组件一样，千万不要直接改变状态，而应该使用相应的api
7.  **和类组件一样，函数组件中改变状态可能是异步的（在DOM事件中），多个状态变化会合并，以提高性能。此时不能信任之前的状态，而应该使用回调函数的方式改变状态。**
8. 如果要实现强制刷新组件
   1. 类组件：使用forceUpdate函数（不会运行shouldComponentUpdate）
   2.  函数组件：可以使用空对象的useState 

异步问题：

```jsx
import React, { useState } from 'react'

export default function App() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <span>{count}</span>
      <button
        onClick={() => {
          // 异步 不会立即改变 事件运行完成之后一起改变
          setCount(count + 1)
          // 此时 count的值仍然是0 
          setCount(count + 1)
          // 点击事件执行完成后 发现多个setCount调用 合并多个setCount为一个 
          // 也就会导致后面的状态改变覆盖前面的 最终使得点击一次 count只会加1
        }}
      >+</button>
    </div>
  )
}
```

解决异步问题：

```jsx
import React, { useState } from 'react'

export default function App() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <span>{count}</span>
      <button
        onClick={() => {
          // 使用回调函数 解决异步问题
          // 这里调用两次setCount 但是最终只会render一次（调用setCount一次）
          // 原因还是会合并这两次状态 只不过隐式使用中间变量存储了每次改变count的结果
          // 最后进行一次setCount 最终使得点击一次 count只会加2
          setCount(prevCount => prevCount + 1)
          setCount(prevCount => prevCount + 1)
        }}
      >+</button>
    </div>
  )
}
```

强制刷新：

```jsx
import React, { useState } from 'react'

export default function App() {
  const [, forceUpdate] = useState({})
  return (
    <div>
      <button onClick={() => forceUpdate({})}>强制刷新</button>
    </div>
  )
}
```



### Effect Hook

Effect Hook: 用于在函数组件中处理副作用

副作用：

1. ajax请求
2. 计时器
3. 其它异步操作
4. 更改真实DOM对象
5. 本地存储
6. 其它会对外部产生影响的操作

函数useEffect，接收一个函数作为参数，接收的函数就是需要进行副作用操作的函数。

```jsx
import React, { useState, useEffect } from 'react'

export default function App() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    // 操作页面标题 有副作用
    document.title = count
  })

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => { setCount(count + 1)}}>
        +
      </button>
    </div>
  )
}
```



**注意的细节**：

1. 副作用函数的运行时间点，是在页面完成真实的UI渲染之后。因此它的执行是异步的，不会阻塞浏览器。
   1. 与类组件中`componentDidMount`和`componentDidUpdate`的区别
      1. `componentDidMount`和`componentDidUpdate`更改了真实DOM，但是用户还没有看到UI更新，同步的
      2. useEffect中的副作用函数，更改了真实DOM，并且用户已经看到了UI更新，异步的
2. 每个函数组件中，可以多次使用useEffect，但是不要放入判断、循环等代码块中
3. useEffect中副作用函数，可以有返回值，返回值必须是一个函数，该函数叫做清理函数
   1. **首次渲染组件不会运行**
   2. **清理函数运行时间点，每次在运行副作用函数之前**
   3. **组件被销毁时一定会运行**
4. useEffect函数，可以传递第二个参数
   1. 第二个参数是一个数组
   2. 数组中记录该副作用的依赖数据
   3. **当组件重新渲染后，只有依赖数组中的数据与上一次不一样时，才会执行副作用**
   4. 所以，当传递了依赖数据之后，如果数据没有发生变化
      1. **副作用函数只在第一次页面渲染完成后运行**
      2. **清理函数只在卸载组件后运行**
5. useEffect函数，不传递第二个参数，则副作用函数默认每次render后都会运行
6. useEffect闭包
7. 副作用函数在每次注册时，会覆盖掉之前的副作用函数，因此，尽量保证副作用函数稳定。否则控制起来会比较复杂

useEffect的返回值是一个清理函数：

```jsx
import React, { useState, useEffect } from 'react'

// 点击两次button 输出如下：
// render
// 我是副作用函数
// render
// 我是清理函数
// 我是副作用函数
// render
// 我是清理函数
// 我是副作用函数
export default function App() {
  const [count, setCount] = useState(0)
  console.log('render')
  useEffect(() => {
    console.log('我是副作用函数')
    // 操作页面标题 有副作用
    document.title = count
    return () => {
      console.log('我是清理函数')
    }
  })

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => { setCount(count + 1)}}>
        +
      </button>
    </div>
  )
}
```

useEffect传递第二个参数（依赖数组为空时可以实现副作用函数只执行一次的效果）：

```jsx
import React, { useState, useEffect } from 'react'

// 点击两次button 输出如下：
// render
// 我是副作用函数
// render
// render
export default function App() {
  const [count, setCount] = useState(0)
  console.log('render')
  useEffect(() => {
    console.log('我是副作用函数')
    // 操作页面标题 有副作用
    document.title = count
    return () => {
      console.log('我是清理函数')
    }
  }, []) // 依赖数组为空 

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => { setCount(count + 1)}}>
        +
      </button>
    </div>
  )
}
```

useEffect形成闭包：

```jsx
import React, { useState, useEffect } from 'react'

// 程序运行 快速点击button5下 输出如下：
// 0
// 1
// 2
// 3
// 4
// 5
export default function App() {
  // 使用一个状态，该状态的默认值是0
  const [count, setCount] = useState(0)
  useEffect(() => {
    setTimeout(() => {
      console.log(count)
    }, 5000)
  })

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  )
}
```

写一个倒计时组件：

```jsx
import React, { useState, useEffect } from 'react'

// props接收一个beginTime属性
function Countdown(props) {
  const [count, setCount] = useState(props.beginTime)
  useEffect(() => {
    if (count === 0)  return
    setTimeout(() => {
      setCount(count - 1)
    }, 1000)
  }, [count])

  return <span>{count}</span>
}

export default function App() {
  return <Countdown beginTime={10}/>
}
```



### 自定义Hook

自定义Hook: 将一些常用的、跨越多个组件的Hook功能，抽离出去形成一个函数，该函数就是自定义Hook。

自定义Hook，由于其内部需要使用Hook功能，所以它本身也需要按照Hook的规则实现：

1. 函数名必须以use开头
2.  调用自定义Hook函数时，应该放到顶层



例1：很多组件都需要在第一次加载完成后，获取某个api下的列表数据

```jsx
//// useAllStudents.js
import { useEffect, useState } from 'react'

async function getAllStudents() {
  return await fetch('http://mock-api.com/DmgvxyKQ.mock/list')
    .then(response => response.json()).then(response => response.data.list)
}

// 自定义Hook
export default function useAllStudents() {
  const [students, setStudents] = useState([])
  // 第一次运行useEffect，先注册异步函数，等到页面渲染完成后再运行
  // 当页面渲染完成后，运行useEffect里注册的异步函数，拿到了数据，重新设置状态
  // 状态发生改变，函数重新运行，第二次运行useEffect时发现依赖项数组为空，所以不再执行异步函数
  // 这时候最后返回从后端请求回来的数据
  useEffect(() => {
    (async () => {
      const students = await getAllStudents()
      setStudents(students)
    })()
  }, [])
  // 第一次返回空数组
  // 第二次才返回从后端拿到的数据
  return students
}


//// App.js 使用自定义Hook 展示数据
import useAllStudents from './useAllStudents'
import React from 'react'

export default function App() {
  const students = useAllStudents()
  const list = students.map(item => <li key={item}>{item}</li>)
  return <ul>
    {list}
  </ul>
}
```

当然，使用高阶组件，也可实现相同的封装。但是，你会发现：

1. 没有Hook方式来得优雅
2. 高阶组件会导致组件层次嵌套变深

高阶组件版：

```jsx
import React from 'react'

async function getAllStudents() {
  return await fetch('http://mock-api.com/DmgvxyKQ.mock/list')
    .then(response => response.json()).then(response => response.data.list)
}

function withAllStudents(Comp) {
  return class AllStudentsWrapper extends React.Component {

    state = {
      students: []
    }

    async componentDidMount() {
      const students = await getAllStudents()
      this.setState({students})
    }

    render() {
      return <Comp {...this.props} {...this.state.students} />
    }
  }
}
```

例2：很多组件都需要在第一次加载完成后，启动一个计时器，然后在组件销毁时卸载

```jsx
//// userTimer.js
import { useEffect } from 'react'

/* eslint "react-hooks/exhaustive-deps": "off" */

/**
 * 组件首次渲染后，启动一个interval计时器
 * 组件卸载后，清除该计时器
 * @param fn
 * @param duration
 */
export default (fn, duration) => {
  useEffect(() => {
    const timer = setInterval(fn, duration)
    return () => {
      clearInterval(timer)
    }
  }, [])
}


//// Test.js
import React from 'react'
import useTimer from './useTimer'

export default function Test() {
  useTimer(() => {
    console.log('Test组件的一些副作用操作')
  }, 1000)
  return <h1>Test组件</h1>
}
```



### Reducer Hook

Flux：Facebook出品的一个数据流框架

1. 规定了数据是单向流动的
2. 数据存储在数据仓库中（目前，可认为state就是一个存储数据的仓库）
3. action是改变数据的唯一原因（本质上就是一个对象，action有两个属性）
   1. type：字符串，动作的类型
   2. payload：任意类型，动作发生后的附加信息
4. 具体改变数据的是个函数，该函数叫做reducer
   1. 该函数接收两个参数
      1. state：表示当前数据仓库中的数据
      2. action：描述了如何去改变数据，以及改变数据的一些附加信息
   2. 该函数必须有一个返回结果，用于表示数据仓库变化之后的数据
      1. Flux要求，对象是不可变的。如果返回对象，必须创建新的对象
5. 如果要触发reducer，不可直接调用，而是应该调用一个辅助函数dispatch
   1. 该函数仅接受一个参数：action
   2. 该函数会间接去调用reducer，以达到改变数据的目的



自己手写这个数据流：

```jsx
import React, { useState } from 'react'

function reducer(state, action) {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}

export default function HookCounter() {

  const [count, setCount] = useState(0)

  function dispatch(action) {
    const newCount = reducer(count, action)
    console.log(`日志：n的值  ${count} => ${newCount}`)
    setCount(newCount)
  }

  return (
    <div>
      <button onClick={() => dispatch({type: 'increase'})}>+</button>
      <span>{count}</span>
      <button onClick={() => dispatch({type: 'decrease'})}>-</button>
    </div>
  )
}
```

使用自定义Hook优化：

```jsx
import React, { useState } from 'react'

function reducer(state, action) {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}

function useMyReducer() {
  const [count, setCount] = useState(0)
  function dispatch(action) {
    const newCount = reducer(count, action)
    console.log(`日志：n的值  ${count} => ${newCount}`)
    setCount(newCount)
  }
  return [count, dispatch]
}

export default function HookCounter() {

  const [count, dispatch] = useMyReducer()

  return (
    <div>
      <button onClick={() => dispatch({type: 'increase'})}>+</button>
      <span>{count}</span>
      <button onClick={() => dispatch({type: 'decrease'})}>-</button>
    </div>
  )
}
```

通用化自定义reducer：

```jsx
import React, { useState } from 'react'

function reducer(state, action) {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}

/**
 * 通用的useMyReducer函数
 * @param {function} reducer reducer函数，标准格式的
 * @param {any} initialState 初始状态
 */
function useMyReducer(reducer, initialState) {
  const [state, setState] = useState(initialState)

  function dispatch(action) {
    const newState = reducer(state, action)
    console.log(`日志：  ${state} => ${newState}`)
    setState(newState)
  }
  return [state, dispatch]
}

export default function HookCounter() {

  const [count, dispatch] = useMyReducer(reducer, 0)

  return (
    <div>
      <button onClick={() => dispatch({type: 'increase'})}>+</button>
      <span>{count}</span>
      <button onClick={() => dispatch({type: 'decrease'})}>-</button>
    </div>
  )
}
```

**使用Reducer Hook**

```jsx
import React, { useState, useReducer } from 'react'

function reducer(state, action) {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}

export default function HookCounter() {

  // useReducer 第三个参数是可选的函数 函数返回数用来替代第二个参数(初始状态)
  const [count, dispatch] = useReducer(reducer, 7, (secondParam) => {
    // 传过来的参数是 第二个参数 -> 7
    console.log(secondParam)
    // 现在初始值不是7 而是100了
    return 100
  })
  
  // 第三个参数通常没啥用 它的本意是通过对第二个参数进行一大堆运算 再重新得到一个初始值

  return (
    <div>
      <button onClick={() => dispatch({type: 'increase'})}>+</button>
      <span>{count}</span>
      <button onClick={() => dispatch({type: 'decrease'})}>-</button>
    </div>
  )
}
```



### Context Hook

用于获取上下文数据

使用context：

```jsx
import React from 'react'

const context = React.createContext()

const Provider = context.Provider
const Consumer = context.Consumer

function Test() {
  return (
    <Consumer>
      { value => <h1>Test, 上下文的值：{ value }</h1> }
    </Consumer>
  )
}

export default function () {
  return (
    <div>
      <Provider value="hello context hook">
        <Test />
      </Provider>
    </div>
  )
}
```

使用Context Hook：

```jsx
import React, { useContext } from 'react'

const context = React.createContext()

const Provider = context.Provider

function Test() {
  // 直接获取
  const value = useContext(context)
  return <h1>Test, 上下文的值：{ value }</h1>
}

export default function () {
  return (
    <div>
      <Provider value="hello context hook">
        <Test />
      </Provider>
    </div>
  )
}
```



### Callback Hook

函数名：useCallback

用于得到一个固定引用值的函数，通常用它进行性能优化

useCallback：

该函数有两个参数：

1. 函数，useCallback会固定该函数的引用，只要依赖项没有发生变化，则始终返回之前函数的地址
2. 数组，记录依赖项

```jsx
import React, { useCallback, useState } from 'react'

class Test extends React.PureComponent {
  render() {
    console.log('text render')
    return (
      <div>
        <h1>{ this.props.text }</h1>
        <button onClick={ this.props.onClick }>改变文本</button>
      </div>
    )
  }
}

function Parent() {
  console.log('parent render')
  const [text, setText] = useState('123')
  const [count, setCount] = useState(0)

  const handleClick = useCallback(() => {
    setText(Math.random())
  }, [])

  return (
    <div>
      {/* 函数的地址每次渲染都发生了变化 导致子组件跟着重新渲染
          若子组件是经过优化之后的组件 则可能导致优化失效
      */ }
      {/*<Test text={ text } onClick={ () => setText(Math.random) } />*/ }

      {/* 使用useCallback优化 这样当input输入值发生变化 不会去影响Test组件重新渲染了*/ }
      <Test text={ text } onClick={ handleClick } />
      <input type="number" value={ count } onChange={ e => setCount(e.target.value) } />
    </div>
  )
}

const App = () => <><Parent /></>

export default App
```



### Memo Hook

用于保持一些比较稳定的数据，通常用于性能优化

useMemo和useCallback使用场景类似，但是useMemo相较于useCallback用法更加强大

```jsx
import React, { useMemo, useState } from 'react'

const Item = (props) => {
  console.log('Item Render')
  return <li>{ props.value }</li>
}


// useMemo和useCallback功能类似
// 区别：useCallback是对函数进行缓存，useMemo可以对任何类型进行缓存
// useMemo()的第一个参数是一个函数，函数返回值为要缓存的对象，第二个参数是缓存依赖项，
// 只要依赖项没有发生改变，缓存就不会发生改变。在进行多个DOM渲染时，往往可以极大的提高性能。
const App = () => {

  const [range, setRange] = useState({ min: 1, max: 1000 })
  const [inputValue, setInputValue] = useState(0)

  const list = useMemo(() => {
    const list = []
    for (let i = range.min; i <= range.max; i++) {
      list.push(<Item key={ i } value={ i } />)
    }
    return list
  }, [range.min, range.max])

  return (
    <div>
      <ul>
        { list }
      </ul>
      {/* input框内容发生改变 逻辑上不应该影响list 所以对list进行缓存 list是否重新渲染
          只和range的取值范围相关
      */}
      <input type="number" value={ inputValue } onChange={ e => setInputValue(+e.target.value) } />
    </div>
  )
}

export default App
```



### Ref Hook

Ref Hook用于对ref的缓存，通常用于对某个函数组件节点中某个局部引用进行缓存

```jsx
import React, { useRef, useState } from 'react'

window.arr = []

const App = () => {
  // 点击刷新按钮n次 你会发现window.arr中的存储的ref引用都是不同的
  // const inputRef = React.createRef()
  
  // 点击刷新按钮n次 你会发现window.arr中的存储的ref引用都是同一个 这就是useRef的缓存机制
  const inputRef = useRef()
  window.arr.push(inputRef)
  const [refresh, setRefresh] = useState({})
  return (
    <div>
      <input type="text" ref={inputRef} />
      <button onClick={() => console.log(inputRef.current.value)}>得到input的值</button>
      <button onClick={() => setRefresh({})}>刷新</button>
    </div>
  )
}

export default App
```



### ImperativeHandle Hook

函数：useImperativeHandle()

```jsx
import React, { useImperativeHandle, useRef } from 'react'

function Test(props, ref) {

  // 第一个参数为ref 第二个参数是个函数 第三个参数是依赖项数组
  // 如果使用了依赖项，则第一次调用后，会进行缓存，只有依赖项发生变化时才会重新调用函数
  useImperativeHandle(ref, () => {
    // 该函数第一次加载组件后调用
    // 返回值相当于 ref.current = { method(){} }
    return {
      method() {
        console.log('i am a Test Component')
      }
    }
  }, [])

  return <h1 ref={ref}>Test Component</h1>
}

const TestWrapper = React.forwardRef(Test)

const App = () => {

  const testRef = useRef()

  return (
    <div>
      <TestWrapper ref={testRef}/>
      <button onClick={() => testRef.current.method()}>点击调用Test组件的method方法</button>
    </div>
  )
}

export default App
```



### LayoutEffect Hook

浏览器的页面大致渲染过程：

对DOM进行了改动（同步代码） => 浏览器下一次渲染时间点到达 => 对比差异，进行渲染（异步）=> 用户看到新的效果

useEffect: 浏览器渲染完成后，用户看到新的效果后

useLayoutEffect：完成了DOM改动，但还没有呈现给用户。和useEffect的用法没有任何区别，

唯一的区别就是运行的时间点不同。

**应该尽量使用useEffect，因为它不会导致渲染阻塞，如果出现了问题，再考虑useLayoutEffect**

![cur-lifecycle](./img/useLayoutEffect.png)

```jsx
import React, { useEffect, useLayoutEffect, useRef, useState } from 'react'

const App = () => {

  const [count, setCount] = useState(0)
  const h1Ref = useRef()

  // 使用useEffect时 在内部用ref操作元素DOM时 有可能导致页面一进来会先闪现 0 再 变为 random值
  // useEffect(() => {
  //   h1Ref.current.innerText = Math.random().toFixed(2)
  // })
  useLayoutEffect(() => {
    h1Ref.current.innerText = Math.random().toFixed(2)
  })

  return (
    <div>
      <h1 ref={h1Ref}>{count}</h1>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  )
}

export default App
```



### DebugValue Hook

useDebugValue：用于将自定义Hook的关联数据显示到调试栏

如果创建的自定义Hook通用性比较高，可以选择使用useDebugValue方便调试

```jsx
import React, { useDebugValue, useEffect, useLayoutEffect, useRef, useState } from 'react'

function useTest() {
  const [students, setStudents] = useState([])
  // 使用React DEV Tools 可以看到这个自定义hook信息
  useDebugValue("学生集合")
  return students
}

const App = () => {

  useState(0)
  useState('abc')

  useEffect(() => {
    console.log('effect')
  }, [])

  useTest()

  return (
    <div>
    </div>
  )
}

export default App
```



## React Router

### 概述

站点：无论是使用Vue还是React，开发的单页应用程序，可能只是该站点的一部分（某一个功能块）

一个单页应用里，可能会划分为"多个页面"（几乎完全不同的页面效果）

如果要在单页应用中，完成组件的切换，需要实现下面两个功能：

1. 根据不同的页面地址，展示不同的组件（核心）
2. 完成无刷新的地址切换

我们将实现了以上两个功能的插件，称之为路由

**React Router**

1. react-router：路由核心库，包含诸多和路由功能相关的核心代码
2. react-router-dom：利用路由核心库，结合实际的页面，实现跟页面路由密切相关的功能

如果是在页面中实现路由，需要安装`react-router-dom`，一般安装它的时候附带着就安装了`react-router`



### 两种模式

页面：根据不同的页面地址，展示不同的组件

url地址组成

例：https://www.react.com:443/news/1-2-1.html?a=1&b=2#abcdefg

1. 协议名(schema)：https
2. 主机名(host)：www.react.com
   1. ip地址
   2. 预设值：localhost
   3. 域名
   4. 局域网中的电脑名称
3. 端口号(port)：443
   1. 如果协议是http，端口号是80，则可以省略端口号
   2. 如果协议是https，端口号是443，则可以省略端口号
4. 路径(path)：/news/1-2-1.html
5. 地址参数(search、query)：?a=1&b=2
   1. 附带的数据
   2. 格式：属性名=属性值&属性名=属性值...
6. 哈希(hash、锚点)：#abcdefg
   1. 附带的数据 - abcdefg

#### Hash Router

哈希路由，根据url地址中的哈希值来确定显示的组件

原理：hash的变化，不会导致页面刷新

这种模式的兼容性最好



#### Borswer History Router 

浏览器历史记录路由，HTML5出现后，新增了History Api，从此以后，浏览器拥有了改变路径而不刷新页面的方式

History表示浏览器的历史记录，它使用栈的方式存储

window.history中的api：

1. history.length：获取栈中的数据量
2. history.pushState：向当前历史记录栈中加入一条新的记录
   1. 参数1：附加的数据，自定义的数据，可以是任何类型
   2. 参数2：页面标题，目前大部分浏览器不支持
   3. 参数3：新的地址
3. history.replaceState：将当前指针指向的历史记录，替换为某个记录
   1. 参数1：附加的数据，自定义的数据，可以是任何类型
   2. 参数2：页面标题，目前大部分浏览器不支持
   3. 参数3：替换的新地址

根据页面路径决定渲染哪个组件，这种模式更贴近用户的使用



### 路由组件

react-router 为我们提供了两个重要组件

#### Router组件

它本身不做任何展示，仅提供路由模式配置，另外该组件为产生一个上下文，上下文会提供一些实用

的对象和方法，供其它相关组件使用

1. HashRouter：该组件使用hash模式匹配
2. BrowserRouter：该组件使用BrowserHistory模式匹配

通常情况下，Router组件只有一个，该组件包裹整个页面（即页面的根组件）



#### Route组件

根据不同的地址，展示不同的组件。Route组件可以写到任意的地方，只要保证是Router组件的后代元素

重要属性：

1. path：匹配的路径
   1. 默认情况下，不区分大小写，可以设置sensitive属性为true，来区分大小写
   2. 默认情况下，只匹配初始目录，可以设置exact属性为true，来精确匹配
   3. 如果不写path，则会匹配任意路径
2. component：匹配成功后要显示的组件
3. children：
   1. 传递React元素，无论是否匹配，一定会显示children，此时会忽略掉component属性
   2. 传递一个函数，该函数有多个参数，这些参数来自于上下文，其返回值为React元素，则渲染该React元素，而且忽略掉component属性

```jsx
import React from 'react'
import { BrowserRouter as Router, Route } from 'react-router-dom'

function A() {
  return <h1>组件A</h1>
}

function B() {
  return <h1>组件B</h1>
}

function C() {
  return <h1>组件C</h1>
}

const App = () => {

  return (
    <Router>
      <Route path={ '/' } component={ () => <h1>首页</h1> } exact />
      <Route path={ '/a' } component={ A } exact />
      <Route path={ '/a/b' } component={ B } />
      <Route path={ '/a/c' } component={ C } />
    </Router>
  )
}

export default App
```



#### Switch组件

写到Switch组件中的Route组件，当匹配到第一个Route后，会立即停止匹配

由于Switch组件会循环所有子元素，然后让每个子元素去完成匹配，若匹配到则渲染组件，然后停止循环，

因此，不可在Switch的子元素中使用除Route外的其它组件

使用Switch可以实现404页面：

```jsx
import React from 'react'
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'

function A() {
  return <h1>组件A</h1>
}

function B() {
  return <h1>组件B</h1>
}

function C() {
  return <h1>组件C</h1>
}

const App = () => {

  return (
    <Router>
      <Switch>
        <Route path={ '/a/b' } component={ B } />
        <Route path={ '/a/c' } component={ C } />
        <Route path={ '/a' } component={ A }>
          {
            (...args) => {
              // { history:{...}, location:{...}, match:{...}, ... }
              console.log(...args)
              return <em>一定会看到的内容</em>
            }
          }
        </Route>
        <Route path={ '/' } component={ () => <h1>首页</h1> } exact />
        <Route component={ () => <h1>找不到页面</h1> } />
      </Switch>
    </Router>
  )
}

export default App
```



### 路由信息

Router组件会创建一个上下文，并且向上下文中注入一些信息

该上下文对开发者是隐藏的，Route组件若匹配到了地址，则会将这些上下文的信息**作为属性**传入对应的组件



#### 非路由组件获取路由信息

某些组件，并没有直接放到Route中，而是嵌套在其它普通组件中，因此它的props中没有路由信息，如果这些组件需要获取到路由信息，可以使用下面两种方式：

1. 将路由信息从父组件一层一层传递到子组件（基本不用）
2. 使用react-router提供的高阶组件`withRouter`，包装要使用的组件，该高阶组件会返回一个新组件，新组件可以使用路由信息

```jsx
import React from 'react'
import { BrowserRouter as Router, withRouter } from 'react-router-dom'

const B = (props) => {
  // 输出路由信息
  console.log(props)
  return <h1>组件B</h1>
}

const NewB = withRouter(B)

export default function () {
  return (
    <Router>
      <NewB />
    </Router>
  )
}
```





#### history

它并不是window.history对象，我们利用该对象无刷新跳转地址

**为什么没有直接使用history对象？**

1. React-Router中有两种模式：Hash、History模式，不直接使用是为了能够适配不同的模式
2. 当使用window.history.pushState方法时，没有办法收到任何通知，将导致React无法知晓地址发生了变化，导致无法重新渲染组件

方法：

- push：将某个新的地址入栈（历史记录）
  - 参数1：新的地址
  - 参数2：可选，附带的状态数据，可以在props.history.location.state里拿到附加的数据。这个参数不常用，因为我们一般会选择在地址里面传一些数据
- replace：将某个新的地址替换掉当前栈中指向的地址
- go：用法与window.history一致
- forward：用法与window.history一致
- back：用法与window.history一致



#### location

首先明确一点：props.history.location === props.location

即：与history.locaiton完全一致，是同一个对象，但是与window.history不同

location对象中记录了当前地址的相关信息

属性：

- pathname：路径
- search：查询参数
- hash：哈希
- state：附带的状态数据

由于上述属性的内容都是字符串的形式，这样往往不利于我们提取其中的数据

因此，我们通常使用第三方库`query-string`，用于解析地址栏中的数据



#### match

该对象保存了**路由匹配**的相关信息

属性：

- isExact：从事实上出发，当前的路径和路由配置的路径是否是精确匹配的
- path：代码中匹配的路径规则，如`/news/:year/:month/:day`
- url：地址栏的url，如`/news/2020/2/3`
- params：路径附加的数据信息，如`{year: "2020", month: "2", day: "3"}`



#### 路由传递数据

**向某个页面传递数据的方式**

1. 使用state：在push页面时，给第二参数为你想指定的数据，不常用
2. **利用search：把数据填写在地址栏中的`?`后，比较常用**
3. 利用hash：把数据填写到hash后，更少用
4. **利用params：把数据填写到路径中，常用**

parms方式的补充说明：

实际上，在书写Route组件的path属性时，可以书写一个`string pattern`（字符串正则），如`/news:year/:month/:day`

- 如果传递的day是可选的，那么可以`/news:year/:month/:day?`
- 如果要约束传递的必须是数字，那么可以`/news/:year(\d+)/:month(\d+)/:day(\d+)`
- 如果接着后面一定要匹配任意路径，但是得有，那么可以`/news/:year(\d+)/:month(\d+)/:day(\d+)/*`

react-router使用了第三方库`path-to-regexp`来解析路径字符串正则为真正的正则表达式

---

利用路由传递数据的两个经典例子：

1. search，即查询参数方式传递数据：

```jsx
import React from 'react'
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'
import queryString from 'query-string'

// 获取到了search中的数据
const News = (props) => {
  const search = queryString.parse(props.location.search)
  return <p>显示{ search.year }年的新闻</p>
}

// 使用search传递数据
const A = (props) => {
  return (
    <div>
      <h1>组件A</h1>
      <button onClick={
        () => props.history.push(`/news?year=${ new Date().getFullYear() }&`) }
      >
        跳转到news
      </button>
    </div>
  )
}

export default function () {
  return (
    <Router>
      <Switch>
        <Route path={ '/a' } component={ A } />
        <Route path={ '/news' } component={ News } />
      </Switch>
    </Router>
  )
}
```

2. params，即路径方式传递数据

```jsx
import React from 'react'
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'

// 获取到了params中的数据
const News = (props) => {
  const { year, month, day } = props.match.params
  return <p>显示{ year }年{ month }月{ day }日的新闻</p>
}

// 使用params传递数据
const A = (props) => {
  const date = new Date()
  const year = date.getFullYear()
  const month = date.getMonth()
  const day = date.getDay()
  return (
    <div>
      <h1>组件A</h1>
      <button onClick={
        () => props.history.push(`/news/${year}/${month}/${day}`) }
      >
        跳转到news
      </button>
    </div>
  )
}

export default function () {
  return (
    <Router>
      <Switch>
        <Route path={ '/a' } component={ A } />
        <Route path={ '/news/:year/:month/:day' } component={ News } />
      </Switch>
    </Router>
  )
}
```



### 其它组件

#### Link

生成一个无刷新跳转的a元素

属性：

- to
  - 字符串形式：跳转的目标地址
  - 对象形式：
    - pathname：url路径
    - search：查询参数
    - hash：哈希
    - state：附加的状态信息
- replace：bool值，表示是否替换当前地址，默认是false，即push方式
- innerRef：可以将内部的a元素的ref附着在传递的对象或函数参数上
  - 函数
  - ref对象

自己写一个简化版的Link：

```jsx
import React from 'react'
import { withRouter } from 'react-router-dom'

function Link(props) {
  return (
    <a href=""
       onClick={ e => {
         e.preventDefault()
         props.history.push(props.to)
       } }
    >
      { props.children }
    </a>
  )
}

export default withRouter(Link)
```



#### NavLink

是一种特殊的Link，Link组件具备的功能，它都有

它具备的额外功能是：根据当前地址和链接地址，来决定该链接的样式（多了个默认样式类名：`active`，可以通过activeClassName修改为别的名称）

属性：

- activeClassName：匹配成功时使用的类名

- activeStyle：匹配成功时的内联样式

- exact：是否精确匹配

- sensitive：匹配时是否区分大小写

- strict：是否严格匹配最后一个斜杠

  

#### Redirect

重定向组件，当加载到该组件时，会自动跳转到另外一个地址

属性：

- to：跳转的地址
  - 字符串
  - 对象
- push：默认为false，表示跳转使用replace的方式。true，为push方式
- from：当匹配到from地址规则时才进行跳转，可以是正则
- exact：是否精确匹配from
- sensitive：from匹配时是否区分大小写
- strict：from是否严格匹配最后一个斜杠







## Redux核心概念

### MVC

它是一个UI的解决方案，用于降低UI，以及UI关联的数据的复杂度

**传统的服务器端的MVC**

环境：

1. 服务端需要响应一个完整的HTML
2. 该HTML中包含页面中需要的数据
3. 浏览器仅承担渲染页面的作用

以上的这种方式叫做**服务端渲染**，即服务器端将完整的页面组装好后，一起发送给客户端

服务器端需要处理UI中要用到的数据，并且要将数据嵌入到页面中，最终生成一个完整的HTML页面响应

![serverRender](./img/serverRender.png)



![cRender](./img/cRender.png)

为了降低处理这个过程的复杂度，出现了MVC模式

**Controller**：控制器，处理请求，组装这次请求需要的数据

**Model**：需要用于渲染UI的数据模型

**View**：视图，用于将模型组装到界面中

![serverMVC](./img/serverMVC.png)



**前端MVC模式的困难**

React解决了 数据 -> 视图 的问题

1. 前端的controller要比服务器复杂很多，因为前端中的controller处理的是用户的操作，而用户的操作场景是复杂的
2. 对于那些组件化的框架，比如Vue，React它们使用的是单向数据流。若需要共享数据，则必须将数据提升到顶层组件，然后数据再一层一层传递，极其繁琐
   1. 虽然可以使用上下文来提供共享数据，但对数据的操作难以监控，容易导致调试错误的困难，以及数据还原的困难
   2. 若开发一个大中型项目，共享的数据很多，会导致上下文中的数据非常复杂

因此，前端需要一个独立的数据解决方案

**Flux**

Fackbook提出的数据解决方案，它的最大历史意义在于，它引入了action的概念

action是一个普通的对象，用于描述要干什么

store表示数据仓库，用于存储共享数据。还可以根据不同的action更改仓库中的数据

![flux](./img/flux.png)

示例：

```js
var loginAction = {
  type: 'login',
  payload: {
    loginId: 'admin',
    loginPwd: '123123'
  }
}

var deleteAction = {
  type: 'delete',
  payload: 1 // 用户ID
}
```



**Redux**

在Flux基础上，引入了reducer的概念

reducer：处理器，用于根据action来处理数据，处理后的数据会被仓库重新保存

redux的流程：

![redux](./img/redux.png)

代码示例：

```jsx
import { createStore } from 'redux'

// 假设仓库中仅存放了一个数字，该数字的变化可能是加1或者减1
// 约定action的格式：{type: "操作类型", payload: 附加数据}

// 初始状态
const initState = 0

/**
 * reducer 本质上就是一个普通函数
 * @param state 之前仓库中的数据
 * @param action 描述要做什么的对象
 * @return number 返回一个新的状态
 */
function reducer(state = initState, action) {
  if (action.type === 'increase') {
    return state + 1
  } else if (action.type === 'decrease') {
    return state - 1
  } else {
    return state
  }
}

const store = createStore(reducer, initState)

const increaseAction = {
  type: 'increase'
}

// 得到仓库中当前的数据
// 输出 0
console.log(store.getState())

// 向仓库分发action
store.dispatch(increaseAction)

// 得到仓库中当前的数据
// 输出 1
console.log(store.getState())
```



### action

1. action是一个plain-object（平面对象）
   1. 它的`__proto__`指向Object.prototype
2. 通常，使用payload属性表示附加数据（不是强制要求）
3. action中必须有type属性，该属性用于描述操作的类型
   1. 但是，没有对type类型做出要求
4. 在大型项目中，由于操作类型非常多，为了避免硬编码（hard code），会将action的类型存放到一个或一些单独的文件中
5. 为了方便传递action，通常会使用action创建函数来创建action
   1. action创建函数应为无副作用的纯函数
   2. 纯函数：不能以任何形式改动参数、不可以有异步、不能对外部的数据造成影响

```js
function actionCreator(newNum) {
  return {
    type: CHANGE,
    payload: newNum
  }
}
```

6. 为了方便利用action创建函数来分发action，redux提供了一个函数`bindActionCreators`，该函数用于增强action创建函数的功能，使它不仅可以创建并且创建后可以自动完成分发



### reducer

reducer是用于改变数据的函数

1. 一个数据仓库，有且仅有一个reducer，并且通常情况下，一个工程只有一个仓库
2. 为了方便管理，通常会将reducer放到单独的文件中
3. reducer被调用的时机
   1. 通过store.dispatch，分发了一个action
   2. 当创建一个store的时候（调用`createStore`时的初始化操作），会调用一次reducer
      1. 可以利用这一点，用reducer初始化状态
      2. 创建仓库时，不传递任何默认状态
      3. 将reducer的参数state设置一个默认值
4. reducer内部通常用switch分支
5. **reducer必须是一个没有副作用的纯函数**
   1. 为什么要是纯函数？
      1. 纯函数有利于测试和调试
      2. 有利于还原数据
      3. 有利于将来和react结合时的优化
   2. 具体要求
      1. 不能改变参数，因此若要让状态变化，必须得到一个新的状态
      2. 不能有异步
      3. 不能对外部环境造成影响
6. 由于在大中型项目中，操作比较复杂，数据结构也比较复杂，因此，需要对reducer进行细分
   1. redux提供了一个方法`combineReducers`， 可以帮助我们更加方便的合并reducer
   2. `combineReducers`：合并reducer，得到一个新的reducer，将新的reducer管理一个对象，该对象中的每一个属性交给对应的reducer管理

```jsx
import loginUsers from './loginUser'
import users from './users'
import { combineReducers } from 'redux'

// 合并多个loginUsers 和 users这两个reducer
export default (state = {}, action) => {
  const newState = {
    loginUsers: loginUsers(state.loginUsers, action),
    users: users(state.users, action)
  }
  return newState
}

// 相当于以上代码
// export default combineReducers({
//   loginUsers,
//   users
// })
```



### store

store：用于保存数据

通过`createStore()`创建的对象

该对象的成员：

- dispatch：分发一个action
- getState：得到仓库中当前的状态
- replaceReducer：替换掉当前的reducer
- subscribe：注册一个监听器，监听器是一个无参函数，当分发一个action之后，会运行注册的监听器，该函数会返回一个函数，用于取消监听
- Symbol("observable")



### 手写`createStore`

```js
/**
 * 实现createStore的功能
 * @param {function} reducer reducer
 * @param {any} defaultState 默认的状态值
 */
export default function (reducer, defaultState) {

  // 当前使用的reducer
  let currentReducer = reducer
  
  // 当前仓库中的状态
  let currentState = defaultState

  // 记录所有的监听器（订阅者）
  const listeners = []

  function dispatch(action) {
    // 验证action
    if (!isPlainObject(action)) {
      throw new TypeError('action must be a plain object')
    }
    
    // 验证action的type属性是否存在
    if (action.type === undefined) {
      throw new TypeError('action must has a property of "type"')
    }
    
    currentState = currentReducer(currentState, action)
    
    // 运行所有的监听器（订阅者）
    listeners.forEach(listener => listener())
  }
  
  function getState() {
    return currentState
  }

  /**
   * 添加一个监听器
   * @param listener
   */
  function subscribe(listener) {
    // 将监听器加入到数组中
    listeners.push(listener)
    
    // 是否已经移除
    let isRemove = false
    
    // 返回一个取消监听的函数
    return () => {
      if (isRemove) return
      // 将listener从数组中移除
      const index = listeners.indexOf(listener)
      listeners.splice(index, 1)
      isRemove = true
    }
  }

  // 创建仓库时，需要分发一次初始的action
  dispatch({
    type: `@@redux/INIT${getRandomString(6)}`
  })

  return {
    dispatch,
    getState,
    subscribe
  }
}

/**
 * 判断某个对象是否是plain-object
 * @param o
 */
function isPlainObject(o) {
  if (typeof o !== 'object')  return false
  return Object.getPrototypeOf(o) === Object.prototype
}

/**
 * 得到一个指定长度的中间用点分隔的随机字符串
 * @param length
 */
function getRandomString(length) {
  return Math.random().toString(36).substr(2, length).split('').join('.')
}
```



### 手写`bindActionCreators`

回顾bindActionCreators的用法：

```jsx
import React from 'react'
import { createStore, bindActionCreators } from 'redux'

const reducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case 'decrease':
      return { count: state.count - 1 }
    case 'increase':
      return { count: state.count + 1 }
    default:
      return state
  }
}

const store = createStore(reducer)

// action创建函数
const createDecreaseAction = () => ({ type: 'decrease' })
const createIncreaseAction = () => ({ type: 'increase' })

// 放到对象里
const actionCreators = {
  decrease: createDecreaseAction,
  increase: createIncreaseAction
}

const bindedActionCreators = bindActionCreators(actionCreators, store.dispatch)

// 当然不放到对象里 单个actionCreator函数也是可以的
// const bindedDecreaseCreator = bindedActionCreators(createDecreaseAction, store.dispatch)

// 绑定好了action就可以帮你省略掉dispatch这一步 直接触发就好了
bindedActionCreators.decrease()
console.log(store.getState())  // -1

bindedActionCreators.increase()
console.log(store.getState())  // 0
```

自己实现`bindActionCreators`：

```js
export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return getAutoDispatchActionCreator(actionCreators, dispatch)
  } else if (typeof actionCreators === 'object') {
    const result = {}
    for (const key in actionCreators) {
      if (actionCreators.hasOwnProperty(key)) {
        const actionCreator = actionCreators[key]
        if (typeof actionCreator === 'function') {
          result[key] = getAutoDispatchActionCreator(actionCreator, dispatch)
        }
      }
    }
    return result
  } else {
    throw new TypeError('actionCreators must be an object or function which means action creator')
  }
}

/**
 * 得到一个自动分发的action创建函数
 * @param actionCreator
 * @param dispatch
 */
function getAutoDispatchActionCreator(actionCreator, dispatch) {
  return function (...args) {
    const action = actionCreator(...args)
    dispatch(action)
  }
}
```



### 手写`combineReducers`

回顾`combineReducers`的用法：

```jsx
import React from 'react'
import { createStore, combineReducers } from 'redux'

const counterReducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case 'decrease':
      return { count: state.count - 1 }
    case 'increase':
      return { count: state.count + 1 }
    default:
      return state
  }
}

const welcomeReducer = (state = { greeting: 'Hello Sir!' }, action) => {
  if (action.type === 'madam') {
    return { greeting: 'Hi Madam!' }
  } else {
    return state
  }
}

const reducer = combineReducers({counterReducer, welcomeReducer})

const store = createStore(reducer)

// {counterReducer: {...}, welcomeReducer: {...}}
console.log(store.getState())
```

combineReducers功能：组装多个reducer，返回一个reducer，数据使用一个对象表示，对象的属性名与传递的参数对象保持一致

自己实现combineReducers：

```js
export default function combineReducers(reducers) {
  // 1. 验证
  validateReducers(reducers)

  /**
   * 返回的是一个reducer函数
   */
  return function (state = {}, action) {
    // 要返回的新状态
    const newState = {}
    for (const key in reducers) {
      if (reducers.hasOwnProperty(key)) {
        const reducer = reducers[key]
        newState[key] = reducer(state[key], action)
        // 上面一行代码等价于下面
        // newState[key] = reducer(undefined, action)
      }
    }
    // 返回状态
    return newState
  }
}

function validateReducers(reducers) {
  if (typeof reducers !== 'object') {
    throw new TypeError('reducers must be an object')
  }
  if (!isPlainObject(reducers)) {
    throw new TypeError('reducers must be an plain object')
  }
  // 验证reducer的返回结果是不是undefined 这里会验证两次
  for (const key in reducers) {
    if (reducers.hasOwnProperty(key)) {
      const reducer = reducers[key]
      // 传递一个特殊的type值
      let state = reducer(undefined, {
        type: getActionTypes().init()
      })
      if (state === undefined) {
        throw new TypeError('reducers must not return undefined')
      }
      // 第二次验证
      state = reducer(undefined, {
        type: getActionTypes().unknown()
      })
      if (state === undefined) {
        throw new TypeError('reducers must not return undefined')
      }
    }
  }
}

/**
 * 判断某个对象是否是plain-object
 * @param o
 */
function isPlainObject(o) {
  if (typeof o !== 'object')  return false
  return Object.getPrototypeOf(o) === Object.prototype
}

/**
 * 得到一个指定长度的中间用点分隔的随机字符串
 * @param length
 */
function getRandomString(length) {
  return Math.random().toString(36).substr(2, length).split('').join('.')
}

/**
 * 两个用于验证的 action type
 * @return {string|{init(): string, unknown(): string}}
 */
function getActionTypes() {
  return {
    init() {
      return `@@redux/INIT${getRandomString(6)}`
    },
    unknown() {
      return `@@redux/PROBE_UNKNOWN_ACTION${getRandomString(6)}`
    }
  }
}
```



## Redux中间件

中间件，类似于插件，可以在不影响原本功能、并且不改动原本代码的基础上，对其功能进行增强。在redux中，中间件主要用于增强dispatch函数

实现redux中间件的基本原理是，更改仓库中的dispatch函数

![middleware](./img/middleware.png)



redux中间件书写：

- 中间件本身是一个函数，该函数接收一个store参数，表示创建的仓库，该仓库并非完整的仓库对象，仅包含getState()和dispatch()。该函数运行的时间是在仓库创建之后运行
- 由于创建仓库后需要自动运行设置的中间件函数，因此，需要在仓库创建时，告诉仓库有哪些中间件
- 需要调用`applyMiddleware`函数，将函数的返回结果作为createStore的第二或第三个参数
- 中间件函数必须返回一个dispatch创建函数
  - 返回的函数需要有一个参数dispatch

应用中间件的两种方式：

```js
// 方式1
const store = createStore(reducer, applyMiddleware(logger1, logger2))
// 方式2
const store = applyMiddleware(logger1, logger2)(createStore)(reducer)
```



## dva

官方网站：https://dvajs.com/，dva是阿里巴巴出品的一个第三库，后来阿里还做了一个脚手架umijs，特别好用

dva不仅仅是一个第三方库，更是一个框架，它主要整合了redux的相关内容，让我们处理数据更加容易，实际上，dva依赖了很多：react、react-router、redux、redux-saga、react-redux、conneced-react-router等

![dva](./img/dva.png)



### 使用

1. dva默认导出了一个函数，通过调用该函数，可以得到一个dva对象

2. dva对象.router：路由方法，传入一个函数，该函数返回一个React节点，将来应用程序启动后，会自动渲染该节点

3. dva对象.start：该方法用于启动dva应用程序，可以认为就是启动react程序，该函数传入一个选择器，用于选中页面中的某个DOM元素，react会将内容渲染到改元素内部

4. dva对象.model：该方法用于定义一个模型，该模型可以理解为redux的action、reducer、redux-sage副作用处理的整合，整合成一个对象，将该对象传入model方法即可

5. model方法传入一个对象参数，对象的属性有：

   1. namespace：不能省略，命名空间，该属性是一个字符串，字符串的值，会被作为仓库中的属性保存

   2. state：不能省略，该模型的默认状态

   3. reducers：该属性配置为一个对象，对象中的每个方法就是一个reducer。方法的名字，就是匹配的action类型

   4. effects：处理副作用，底层使用redux-saga实现的，该属性配置为一个对象，对象中的每个方法均处理一个副作用，方法的名字，就是匹配的action类型

      1. 函数的参数1：action
      2. 函数的参数2：封装好的saga/effects对象

   5. subscriptions：配置为一个对象，该对象中可以写任意数量任意名称的属性，每个属性是一个函数，这些函数会在模型加入到仓库后立即运行

      ```js
      subscriptions: {
        // 模型加入仓库后立即执行
        // o为一个对象：{dispatch: fn, history: {}}
        init(o) {
          console.log('init', o)
        }
      }
      ```

      

演示1：

```jsx
import React from 'react'
import dva from 'dva'
import Counter from './Counter'

// 得到一个dva对象
const app = dva()

// 设置跟路由 即启动后，要运行的函数，函数返回结果会被渲染
app.router(() => <Counter />)

const counterModel = {
  namespace: 'counter',
  state: 0,
  reducers: {
    increase(state) {
      return state + 1
    },
    decrease(state) {
      return state - 1
    },
    // 第二个参数为附加的数据
    add(state, { payload }) {
      return state + payload
    }
  }
}

// 在启动之前 使用定义的模型
app.model(counterModel)

// 启动dva
app.start("#root")

////  Counter组件
import React, {useRef} from 'react'
import { connect } from 'dva'

function Counter(props) {
  const inputRef = useRef()
  return (
    <div>
      <button onClick={ props.onDecrease }>-</button>
      <span>{ props.count }</span>
      <button onClick={ props.onIncrease }>+</button>
      <input type="text" ref={inputRef} />
      <button onClick={() => {
        const n = parseInt(inputRef.current.value)
        props.onAdd(n)
      }}>add</button>
    </div>
  )
}

function mapStateToProps(state) {
  return {
    count: state.counter
  }
}

function mapDispatchToProps(dispatch) {
  return {
    onDecrease() {
      dispatch({
        type: 'counter/decrease'
      })
    },
    onIncrease() {
      dispatch({
        type: 'counter/increase'
      })
    },
    onAdd(n) {
      dispatch({
        type: 'counter/add',
        payload: n
      })
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

演示2 - 使用effects：

现在我们增加两个需求：异步加和异步减

```jsx
//// counterModel
export default {
  namespace: 'counter',
  state: 0,
  reducers: {
    increase(state) {
      return state + 1
    },
    decrease(state) {
      return state - 1
    },
    add(state, { payload }) {
      return state + payload
    }
  },
  // 一些副作用操作
  effects: {
    // 函数第二个参数为一个对象 里面有很多saga的操作副作用的方法
    // 异步增1 必须写成生成器函数
    * asyncIncrease(action, sagaEffects) {
      const { call, put } = sagaEffects
      yield call(delay, 2000)
      // 在模型内部可以不写namespace -> counter/
      yield put({ type: 'increase' })
    },
    // 异步减1 必须写成生成器函数
    * asyncDecrease(action, sagaEffects) {
      const { call, put } = sagaEffects
      yield call(delay, 2000)
      yield put({ type: 'decrease' })
    }
  }
}

function delay(duration) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve()
    }, duration)
  })
}

/////  Counter组件
import React, {useRef} from 'react'
import { connect } from 'dva'

function Counter(props) {
  const inputRef = useRef()
  return (
    <div>
      <button onClick={ props.onDecrease }>-</button>
      <span>{ props.count }</span>
      <button onClick={ props.onIncrease }>+</button>
      <button onClick={ props.onAsyncDecrease }>异步-</button>
      <button onClick={ props.onAsyncIncrease }>异步+</button>
      <input type="text" ref={inputRef} />
      <button onClick={() => {
        const n = parseInt(inputRef.current.value)
        props.onAdd(n)
      }}>add</button>
    </div>
  )
}

function mapStateToProps(state) {
  return {
    count: state.counter
  }
}

function mapDispatchToProps(dispatch) {
  return {
    onDecrease() {
      dispatch({
        type: 'counter/decrease'
      })
    },
    onIncrease() {
      dispatch({
        type: 'counter/increase'
      })
    },
    onAdd(n) {
      dispatch({
        type: 'counter/add',
        payload: n
      })
    },
    onAsyncDecrease() {
      dispatch({type: 'counter/asyncDecrease'})
    },
    onAsyncIncrease() {
      dispatch({type: 'counter/asyncIncrease'})
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```



### 路由

传统方式：

```jsx
import React from 'react'
import Counter from './Counter'
import { BrowserRouter as Router, Route, Switch, NavLink } from 'dva/router'

function Home() {
  return <h1>首页</h1>
}

export default function() {
  return (
    <Router>
      <div>
        <ul>
          <li><NavLink to="/">首页</NavLink></li>
          <li><NavLink to="/counter">计数器</NavLink></li>
        </ul>
        <div>
          <Switch>
            <Route path="/" exact component={Home} />
            <Route path="/counter" component={Counter} />
          </Switch>
        </div>
      </div>
    </Router>
  )
}
```

上面那种方式没有将路由和仓库的数据相结合，因此，在数据仓库中就无法知道路由的相关信息了

dva内部实现实际上就是用了connected-react-router这个库来实现数据和路由相关联

connected-react-router：

1. connectRouter函数：传入history对象，得到一个reducer
2. routerMiddleware函数，传入history对象，得到一个redux中间件
3. ConnectedRouter组件，传入history对象，提供路由上下文

**在dva中同步路由到仓库：**

1. 在调用dva函数时，配置history对象
2. 使用ConnectedRouter，提供路由上下文

将路由关联仓库的演示：

```jsx
/////  index.js
import routerConfig from './routerConfig'
import dva from 'dva'
import counterModel from './models/counter'
import { createBrowserHistory } from 'history'

// 得到一个dva对象
const app = dva({
  // 关联路由history
  history: createBrowserHistory()
})

// 设置跟路由 即启动后，要运行的函数，函数返回结果会被渲染
app.router(routerConfig)

// 在启动之前 使用定义的模型
app.model(counterModel)

// 启动dva
app.start("#root")


/////  routerConfig.js 文件
import React from 'react'
import Counter from './Counter'
import { routerRedux, Route, Switch, NavLink } from 'dva/router'

// 使用ConnectedRouter
const { ConnectedRouter } = routerRedux

function Home() {
  return <h1>首页</h1>
}

export default function(props) {
  return (
    // 注意：history一定要传
    <ConnectedRouter history={props.history}>
      <div>
        <ul>
          <li><NavLink to="/">首页</NavLink></li>
          <li><NavLink to="/counter">计数器</NavLink></li>
        </ul>
        <div>
          <Switch>
            <Route path="/" exact component={Home} />
            <Route path="/counter" component={Counter} />
          </Switch>
        </div>
      </div>
    </ConnectedRouter>
  )
}
```



### 配置

```js
const app = dva({
  // 配置
  history: createBrowserHistory(),
  initialState: ...
})
```

1. history：同步到仓库的history对象
2. initialState：创建redux仓库时，使用的默认状态，我们通常不会在这里配置默认状态
3. onError：当仓库的运行发生错误的时候，运行的函数
   1. 参数1：error对象
   2. 参数2：dispatch函数
4. onAction：当触发action时，运行的函数，一般可以用来配置redux中间件
   1. 传入一个中间件对象
   2. 传入一个中间件数组
5. onStateChange：当仓库中的状态发生变化时，运行的函数
6. onReducer：对模型中的reducer的进一步封装
   1. 参数为原来的reducer函数
   2. 返回值为一个新的reducer函数
7. onEffect：类似于对模型中的effect的进一步封装
8. extraReducers：用于配置额外的reducer，它是一个对象，对象的每一个属性是一个方法，每个方法就是一个需要合并的reducer，方法名即属性名
9. extraEnhancers：它是用于封装createStore函数的，dva会将原来的仓库创建函数作为参数传递，返回一个新的用于创建仓库的函数，函数必须放置到数组中



### 插件

通过`dva对象.use(插件)`，来使用插件，插件本质上就是一个对象，该对象与配置对象相同，dva会在启动时，将传递的插件对象混合到配置中

自己写一个简单的dva插件：

```jsx
const logger = store => next => action => {
  console.log("老状态", store.getState())
  console.log(action)
  next(action)
  console.log("新状态", store.getState())
  console.log(action)
}
const myDvaPlugin = {
  onAction: logger
}

// 得到一个dva对象
const app = dva({
  // 关联路由history
  history: createBrowserHistory()
})

// 使用插件
app.use(myDvaPlugin)
```

学会使用第三方dva插件，如：`dva-loading`

该插件会在仓库中加入一个状态，名称为loading，它是一个对象，有三个属性

1. global：全局是否正在处理副作用（加载），只要有任何一个模型在处理副作用，则该属性为true
2. models：一个对象，对象中的属性名以及属性的值，表示哪个对应的模型是否在处理副作用
3. effects：一个对象，对象中的属性名以及属性值，表示的是哪个action触发了副作用

```jsx
import createLoading from 'dva-loading'

// 使用dva-loading插件
app.use(createLoading())

// 后续可以根据仓库中的状态loading的变化来 设置loading效果了
```



## umi

特性：

-  插件化：umi 的整个生命周期都是插件化的，甚至其内部实现就是由大量插件组成，比如 pwa、按需加载、一键切换 preact、一键兼容 ie9 等等，都是由插件实现。
- 开箱即用：你只需一个 umi 依赖就可启动开发，无需安装 react、preact、webpack、react-router、babel、jest 等等。
- 约定式路由：不用再去配置路由了，pages目录下的js文件即路由

全局安装umi，会提供一个命令行工具，通过命令可以对umi工程进行操作

- umi dev：使用开发模式启动工程
- umi build：打包



### 路由

umi对路由的处理，主要通过两种方式：

1. 约定式：使用约定好的文件夹和文件，来代表页面，umi会根据开发者书写的页面，生成路由配置
2. 配置式：直接书写路由配置文件



#### 约定式路由

- umi约定，工程中的pages文件夹中存放的是页面。如果工程包含src目录，则src/pages是页面文件夹
- umi约定，页面的文件名，以及页面的文件路由，是该页面匹配的路由
- umi约定，如果页面的文件名是index，则可以省略文件名，即直接访问相应文件的目录就可访问到index
- umi约定，如果src/layout目录存在，则该目录中的index.js表示的是全局的通用布局，布局中的children则会添加具体的页面（嵌套路由）
- umi约定，如果pages文件夹中包含_layout.js，则layout.js所在的目录以及其所有的子目录中的页面，共用该布局（嵌套路由）
- umi约定，pages/404.js，表示404页面，如果页面无匹配，则会渲染该页面，该约定在开发模式中无效，部署后才能看到效果（404约定）
- umi约定，使用$名称，会产生动态路由。如pages/sub/$id.js，对应的路由就是/sub/:id（动态路由）

路由跳转

1. 声明式

```jsx
import Link from 'umi/link'; // 实际上就是react-router-dom 中的 Link，umi也可使用NavLink

export default () => (
  <Link to="/list">Go to list page</Link>
);
```

2. 命令式

```jsx
import router from 'umi/router';

function goToListPage() {
  router.push('/list');
}
```

路由信息

所有的页面、布局组件都会通过属性，收到下面的属性

- match：等同于react-router的match
- history：等同于react-router的history（history.lcation.query被封装成了一个对象，内部使用的就是query-string进行的封装）
- location：等同于react-router的location（lcation.query被封装成了一个对象，内部使用的就是query-string进行的封装）
- route：对应的是路由配置

如果需要在普通组件中获取路由信息，则需要使用withRouter封装，可以通过`umi/withRouter`导入





#### 配置式路由

当使用了路由配置后，约定式路由全部失效。

两种方式书写umi配置(二选一)：

1. 使用根目录下的文件`.umirc.js`
2. 使用根目录下的文件，`config/config.js`

进行路由配置时，每个皮遏制就是一个匹配规则，并且，每个配置就是一个对象，对象的某些属性，会直接形成Route组件的属性

配置时的注意点：

1. component配置项，需要填写页面组件的路径，路径相对与pages文件夹
2. 如果配置项没有exact，则会自动添加`exact:true`
3. 每一个路由配置，可以添加任何属性
4. Routes属性是一个数组，数组的每一项是一个组件路径，路径相对于项目根目录，当匹配到路由后，会转而渲染该属性指定的组件，并会将component组件作为children放到匹配的组件中

路由配置中的信息，同样可放到约定式路由中，方式是，为约定式路由添加第一个文档注释（主食的格式为YAML），需要将注释放到最开始的位置

YAML格式：

- 键值对，冒号后需要加上空格
- 如果某个属性有多个键或多个值，需要进行缩进（空格）

```jsx
/**
 * title: 首页
 */
import React from 'react'
export default function index() {
  return <h1>首页</h1>
}

/////  .umi/router.js
const routers = [
  ...
  {
    path: '/',
    component: require('../../layouts/index.js').default,
    // 加载到全局布局组件后 接着匹配组件作为其children
    routes: [
      {
        path: '/',
        exact: true,
        component: require('../index.js').default,
        // YAML格式注释 生成的
        title: '首页'
      }
    ]
  }
  ...
]
```



需求1：匹配成功某个路径成功后，接着往下匹配其子路由

```jsx
export default {
  routes: [
    {
      path: '/',
      component: '../layout/index.js',
      exact: false,
      routes: [
        {
          path: '/',
          component: './index.js',
          title: '首页'
        },
        {
          path: '/login',
          component: './login.js',
          title: '登录页'
        },
        {
          path: '/welcome',
          component: './welcome',
          title: '欢迎页'
        }
      ]
    },
  ]
}
```







## Ant Design

对于前端开发者：antd实际上就是一个UI库

网站 = 前台 + 后台

前台：给用户访问的页面，通常需要设计师参与制作

后台：给管理员（通常是公司内部员工）使用，通常设计师不参与制作后台页面