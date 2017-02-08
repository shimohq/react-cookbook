
React Cookbook
=====

*编写简洁漂亮，可维护的 React 应用*

## 目录
- [前言](#前言)
- [组件声明](#组件声明)
- [计算属性](#计算属性)
- [事件回调命名](#事件回调命名)
- [组件化优于多层 render](#组件化优于多层-render)
- [状态上移优于公共方法](#状态上移优于公共方法)
- [容器组件](#容器组件)
- [纯函数的 render](#纯函数的-render)
- [始终声明 PropTypes](#始终声明-proptypes)
- [Props 非空检测](#props-非空检测)
- [使用 Props 初始化](#使用-props-初始化)
- [classnames](#classnames)

---

## 前言

随着应用规模和维护人数的增加，光靠 React 本身灵活易用的 API 并不足以有效控制应用的复杂度。本指南旨在在 ESLint 之外，再建立一个我们团队内较为一致认可的约定，以增加代码一致性和可读性、降低维护成本。

_欢迎在 [Issues](https://github.com/shimohq/react-cookbook/issues) 进行相关讨论_

## 组件声明

全面使用 ES6 class 声明，可不严格遵守该属性声明次序，但如有 propTypes 则必须写在顶部， lifecycle events 必须写到一起。

* class
  * propTypes
  * defaultPropTypes
  * constructor
    * event handlers (如不使用[类属性](http://babeljs.io/docs/plugins/transform-class-properties/)语法可在此声明)
  * lifecycle events
  * event handlers
  * getters
  * render

```javascript
class Person extends React.Component {
  static propTypes = {
    firstName: PropTypes.string.isRequired,
    lastName: PropTypes.string.isRequired
  }
  constructor (props) {
    super(props)

    this.state = { smiling: false }

    /* 若不能使用 babel-plugin-transform-class-properties
    this.handleClick = () => {
      this.setState({smiling: !this.state.smiling})
    }
    */
  }

  componentWillMount () {}

  componentDidMount () {}

  // ...

  handleClick = () => {
    this.setState({smiling: !this.state.smiling})
  }

  get fullName () {
    return this.props.firstName + this.props.lastName
  }

  render () {
    return (
      <div onClick={this.handleClick}>
        {this.fullName} {this.state.smiling ? 'is smiling.' : ''}
      </div>
    )
  }
}
```

**[⬆ 回到目录](#目录)**

## 计算属性

使用 getters 封装 render 所需要的状态或条件的组合

对于返回 boolean 的 getter 使用 is- 前缀命名

```javascript
  // bad
  render () {
    return (
      <div>
        {
          this.state.age > 18
            && (this.props.school === 'A'
              || this.props.school === 'B')
            ? <VipComponent />
            : <NormalComponent />
        }
      </div>
    )
  }

  // good
  get isVIP() {
    return
      this.state.age > 18
        && (this.props.school === 'A'
          || this.props.school === 'B')
  }
  render() {
    return (
      <div>
        {this.isVIP ? <VipComponent /> : <NormalComponent />}
      </div>
    )
  }
```

**[⬆ 回到目录](#目录)**

## 事件回调命名

Handler 命名风格:

- 使用 `handle` 开头
- 以事件类型作为结尾 (如 `Click`, `Change`)
- 使用一般现在时

```javascript
// bad
closeAll = () => {},

render () {
  return <div onClick={this.closeAll} />
}
```

```javascript
// good
handleClick = () => {},

render () {
  return <div onClick={this.handleClick} />
}
```

如果你需要区分同样事件类型的 handler（如 `handleNameChange` 和 `handleEmailChange`）时，可能这就是一个拆分组件的信号

**[⬆ 回到目录](#目录)**

## 组件化优于多层 render

当组件的 jsx 只写在一个 render 方法显得太臃肿时，很可能更适合拆分出一个组件，视情况采用 class component 或 stateless component

```javascript
// bad
renderItem ({name}) {
  return (
    <li>
    	{name}
    	{/* ... */}
    </li>
  )
}

render () {
  return (
    <div className="menu">
      <ul>
        {this.props.items.map(item => this.renderItem(item))}
      </ul>
    </div>
  )
}
```

```javascript
// good
function Items ({name}) {
  return (
    <li>
    	{name}
    	{/* ... */}
    </li>
  )
}

render () {
  return (
    <div className="menu">
      <ul>
        {this.props.items.map(item => <Items {...item} />)}
      </ul>
    </div>
  )
}
```

**[⬆ 回到目录](#目录)**

## 状态上移优于公共方法

一般组件不应提供公共方法，这样会破坏数据流只有一个方向的原则。

再因为我们倾向于更细颗粒的组件化，状态应集中在远离渲染的地方处理（比如应用级别的状态就在 redux 的 store 里），也能使兄弟组件更方便地共享。

```javascript
//bad
class DropDownMenu extends Component {
  constructor (props) {
    super(props)
    this.state = {
      showMenu: false
    }
  }

  show () {
    this.setState({display: true})
  }

  hide () {
    this.setState({display: false})
  }

  render () {
    return this.state.display && (
      <div className="dropdown-menu">
        {/* ... */}
      </div>
    )
  }
}

class MyComponent extends Component {
  // ...
  showMenu () {
    this.refs.menu.show()
  }
  hideMenu () {
    this.refs.menu.hide()
  }
  render () {
    return <DropDownMenu ref="menu" />
  }
}

//good
class DropDownMenu extends Component {
  static propsType = {
    display: PropTypes.boolean.isRequired
  }

  render () {
    return this.props.display && (
      <div className="dropdown-menu">
        {/* ... */}
      </div>
    )
  }
}

class MyComponent extends Component {
  constructor (props) {
    super(props)
    this.state = {
      showMenu: false
    }
  }

  // ...

  showMenu () {
    this.setState({showMenu: true})
  }

  hideMenu () {
    this.setState({showMenu: false})
  }

  render () {
    return <DropDownMenu display={this.state.showMenu} />
  }
}
```

更多阅读: [lifting-state-up](https://facebook.github.io/react/docs/lifting-state-up.html)

## 容器组件

一个容器组件主要负责维护状态和数据的计算，本身并没有界面逻辑，只把结果通过 props 传递下去。

区分容器组件的目的就是可以把组件的状态和渲染解耦开来，改写界面时可不用关注数据的实现，顺便得到了可复用性。

```javascript
// bad
class MessageList extends Component {
  constructor (props) {
    super(props)
  	this.state = {
        onlyUnread: false,
        messages: []
  	}
  }

  componentDidMount () {
    $.ajax({
      url: "/api/messages",
    }).then(({messages}) => this.setState({messages}))
  }

  handleClick = () => this.setState({onlyUnread: !this.state.onlyUnread})

  render () {
    return (
      <div class="message">
        <ul>
          {
            this.state.messages
              .filter(msg => this.state.onlyUnread ? !msg.asRead : true)
              .map(({content, author}) => {
                return <li>{content}—{author}</li>
              })
          }
        </ul>
        <button onClick={this.handleClick}>toggle unread</button>
      </div>
    )
  }
}
```

```javascript
// good
class MessageContainer extends Component {
  constructor (props) {
    super(props)
  	this.state = {
        onlyUnread: false,
        messages: []
  	}
  }

  componentDidMount () {
    $.ajax({
      url: "/api/messages",
    }).then(({messages}) => this.setState({messages}))
  }

  handleClick = () => this.setState({onlyUnread: !this.state.onlyUnread})

  render () {
    return <MessageList
      messages={this.state.messages.filter(msg => this.state.onlyUnread ? !msg.asRead : true)}
      toggleUnread={this.handleClick}
    />
  }
}

function MessageList ({messages, toggleUnread}) {
  return (
    <div class="message">
      <ul>
        {
          messages
            .map(({content, author}) => {
              return <li>{content}—{author}</li>
            })
        }
      </ul>
      <button onClick={toggleUnread}>toggle unread</button>
    </div>
  )
}
MessageList.propTypes = {
  messages: propTypes.array.isRequired,
  toggleUnread: propTypes.func.isRequired
}
```

更多阅读:
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.sz7z538t6)
- [React AJAX Best Practices](http://andrewhfarmer.com/react-ajax-best-practices/)

**[⬆ 回到目录](#目录)**

## 纯函数的 render

render 函数应该是一个纯函数（stateless component 当然也是），不依赖 this.state、this.props 以外的变量，也不改变外部状态

```javascript
// bad
render () {
  return <div>{window.navigator.userAgent}</div>
}

// good
render () {
  return <div>{this.props.userAgent}</div>
}
```

更多阅读: [Return as soon as you know the answer](https://medium.com/@SimonRadionov/return-as-soon-as-you-know-the-answer-dec6369b9b67#.q67w8z60g)

**[⬆ 回到目录](#目录)**

## 始终声明 PropTypes

每一个组件都声明 PropTypes，非必须的 props 应提供默认值。

对于非常广为人知的 props 如 children, dispatch 也不应该忽略。因为如果一个组件没有声明 dispatch 的 props，那么一眼就可以知道该组件没有修改 store 了。

但如果在开发一系列会 dispatch 的组件时，可在这些组件的目录建立单独的 .eslintrc 来只忽略 dispatch。

更多阅读: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[⬆ 回到目录](#目录)**

## Props 非空检测

对于并非 `isRequired` 的 proptype，必须对应设置 defaultProps，避免再增加 if 分支带来的负担

```javascript
// bad
render () {
  if (this.props.person) {
    return <div>{this.props.person.firstName}</div>
  } else {
    return <div>Guest</div>
  }
}
```

```javascript
// good
class MyComponent extends Component {
  render() {
    return <div>{this.props.person.firstName}</div>
  }
}

MyComponent.defaultProps = {
  person: {
    firstName: 'Guest'
  }
}
```

如有必要，使用 PropTypes.shape 明确指定需要的属性

**[⬆ 回到目录](#目录)**

## 使用 Props 初始化

除非 props 的命名明确指出了意图，否则不该使用 props 来初始化 state

```javascript
// bad
constructor (props) {
  this.state = {
    items: props.items
  }
}
```

```javascript
// good
constructor (props) {
  this.state = {
    items: props.initialItems
  }
}
```

更多阅读: ["Props in getInitialState Is an Anti-Pattern"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[⬆ 回到目录](#目录)**

## classnames

使用 [classNames](https://www.npmjs.com/package/classnames) 来组合条件结果.

```javascript
// bad
render () {
  return <div className={'menu ' + this.props.display ? 'active' : ''} />
}
```

```javascript
// good
render () {
  let classes = {
    menu: true,
    active: this.props.display
  }

  return <div className={classnames(classes)} />
}
```

Read: [Class Name Manipulation](https://github.com/JedWatson/classnames/blob/master/README.md)

**[⬆ 回到目录](#目录)**
