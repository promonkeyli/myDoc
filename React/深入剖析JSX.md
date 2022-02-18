### 深入剖析==jsx==

#### 概述

官网：javaScript的语法扩展，类似模板语法，同时具有javaScript的所有功能（通俗点就是html，css，js结构样式以及行为写在一块）

#### 使用

```javascript
const name = 'zs'
// 1.变量（或者js表达式）使用大括号包裹
const elementOne = <h1>Hello, {name}</h1>

// 2.class变成了className(jsx使用小驼峰定义属性,类名可以使用表达式，也可以是字符串)
const elementTwo = <h1 className='myStyle'>Hello, {name}</h1>

// 3.想要实现类似 vue中的v-if或者Angular中的*ngIf效果，代码如下
const state = true
const myDOm = state ? <h1>{name}</h1>: null // 更改状态state的值，实现dom的显隐控制
```

#### jsx底层原理

1. ==Babel== （`js代码编译器：将高版本的js转为向后兼容的js代码`）会把 JSX 转译成一个名为 `React.createElement()` 函数调用，jsx也就是React.createElement()函数调用的语法糖

```javascript
const element = ( <h1 className="greeting"> Hello, world! </h1> ) // jsx
const element = React.createElement( 'h1',{className: 'greeting'},'Hello, world!' ) // React.createElement函数调用
```

2. `React.createElement()` 源码解读

* 源码位置（GitHub clone react 源码）：react>packages>react>src>ReactElement.js>createElement()

```javascript
// 源码如下
function createElement(type, config, children) {
  // 入参分析
  // type: 1.标签名字符串（如：span div 等）；2.组件（如：react类组件或者函数式组件，或者是fragment）
  // config：可选参数，标签属性或者组件的属性（如：span的className或者fragment的key等）
  // children：可选参数，子元素（或者新的React.createElement），多个子元素会返回数组
  
  let propName
  
  const props = {}
  
  let key = null
  let ref = null
  let self = null
  let source = null
  
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref
      if (__DEV__) { warnIfStringRefCannotBeAutoConverted(config) }
    }
    if (hasValidKey(config)) {
      if (__DEV__) { checkKeyStringCoercion(config.key) }
      key = '' + config.key
    }
    self = config.__self === undefined ? null : config.__self
    source = config.__source === undefined ? null : config.__source
    for (propName in config) {
      if ( hasOwnProperty.call(config, propName) && !RESERVED_PROPS.hasOwnProperty(propName))
      { props[propName] = config[propName] }
    }
  }

  const childrenLength = arguments.length - 2
  if (childrenLength === 1) { props.children = children } else if (childrenLength > 1) {
    const childArray = Array(childrenLength)
    for (let i = 0; i < childrenLength; i++) { childArray[i] = arguments[i + 2] }
    if (__DEV__) { if (Object.freeze) { Object.freeze(childArray) } }
    props.children = childArray
  }
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps
    for (propName in defaultProps) {
      if (props[propName] === undefined) { props[propName] = defaultProps[propName] }
    }
  }
  if (__DEV__) {
    if (key || ref) {
      const displayName = typeof type === 'function' ? type.displayName || type.name || 'Unknown' : type
      if (key) { defineKeyPropWarningGetter(props, displayName) }
      if (ref) { defineRefPropWarningGetter(props, displayName) }
    }
  }
  // 出参分析：
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  )
```