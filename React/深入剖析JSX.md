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
function createElement(type, config, children) {
  // 入参分析
  // type: 1.标签名字符串（如：span div 等）；2.组件（如：react类组件或者函数式组件，或者是fragment）
  // config：可选参数，标签属性或者组件的属性（如：span的className或者fragment的key等）
  // children：可选参数，子元素（或者新的React.createElement），多个子元素会返回数组

  let propName // 存储后续遍历的config属性
  const props = {} // 储存config属性键值对

  let key = null
  let ref = null
  let self = null
  let source = null

  // 标签属性不为空时：进行下列赋值操作
  if (config != null) {
    // ref校验赋值处理
    if (hasValidRef(config)) {
      ref = config.ref
      // __DEV__ 前端开发环境（分为development-- 开发模式，production-- 生产模式 ）
      // __DEV__ 为true时 处于development模式 反之处于production模式
      // 详细说明 见 -- https://overreacted.io/zh-hans/how-does-the-development-mode-work/
      if (__DEV__) { warnIfStringRefCannotBeAutoConverted(config) }
    }
    // key校验赋值处理
    if (hasValidKey(config)) {
      if (__DEV__) { checkKeyStringCoercion(config.key) }
      key = '' + config.key
    }
    // __self __source 此处资料解释较少，知道的朋友可以回复哈
    self = config.__self === undefined ? null : config.__self
    source = config.__source === undefined ? null : config.__source
    // props属性筛选
    for (propName in config) {
      if ( hasOwnProperty.call(config, propName) && !RESERVED_PROPS.hasOwnProperty(propName))
      { props[propName] = config[propName] }
    }
  }
  // childrenLength 代表当前元素子元素的个数 （减去的2为type和config）
  const childrenLength = arguments.length - 2
  // 单个子元素 props.children直接赋值传入的children
  if (childrenLength === 1) { props.children = children }
  // 多个子元素
  else if (childrenLength > 1) {
    const childArray = Array(childrenLength)
    for (let i = 0; i < childrenLength; i++) { childArray[i] = arguments[i + 2] }
  // 处于development模式下，冻结对象
    if (__DEV__) { if (Object.freeze) { Object.freeze(childArray) } }
    props.children = childArray
  }
  // 子组件设置默认值
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps
    for (propName in defaultProps) {
      // 父组件undefined 则把默认值赋值赋值给props当作props的属性
      if (props[propName] === undefined) { props[propName] = defaultProps[propName] }
    }
  }
  // 开发模式下
  if (__DEV__) {
    if (key || ref) {
      // type为组件执行前面，type为标签执行后面
      const displayName = typeof type === 'function' ? type.displayName || type.name || 'Unknown' : type
      if (key) { defineKeyPropWarningGetter(props, displayName) }
      if (ref) { defineRefPropWarningGetter(props, displayName) }
    }
  }
  // ReactElement源码解析如下
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
 
```

3.  **ReactElement** 源码解析 (ReactElement函数返回虚拟dom)


```javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  // 入参：type：1.标签 2.组件
  // key:react key
  // ref: dom 引用
  // self / source暂时不清楚
  // owner：
  // props：标签属性或者父组件传入的属性
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  if (__DEV__) {
    // The validation flag is currently mutative. We put it on
    // an external backing store so that we can freeze the whole object.
    // This can be replaced with a WeakMap once they are implemented in
    // commonly used development environments.
    element._store = {};

    // To make comparing ReactElements easier for testing purposes, we make
    // the validation flag non-enumerable (where possible, which should
    // include every environment we run tests in), so the test framework
    // ignores it.
    Object.defineProperty(element._store, 'validated', {
      configurable: false,
      enumerable: false,
      writable: true,
      value: false,
    });
    // self and source are DEV only properties.
    Object.defineProperty(element, '_self', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: self,
    });
    // Two elements created in two different places should be considered
    // equal for testing purposes and therefore we hide it from enumeration.
    Object.defineProperty(element, '_source', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: source,
    });
    if (Object.freeze) {
      Object.freeze(element.props);
      Object.freeze(element);
    }
  }
  return element; // 返回虚拟dom
};
```

