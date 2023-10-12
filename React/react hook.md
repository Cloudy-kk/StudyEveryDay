# react hook

## 目录

- [react hook](#react-hook)
  - [目录](#目录)
  - [userState](#userstate)
    - [1. 基础使用](#1-基础使用)
    - [2. 状态的读取和修改](#2-状态的读取和修改)
    - [3. 组件的更新过程](#3-组件的更新过程)
    - [4. 使用规则](#4-使用规则)
    - [摘要](#摘要)
    - [批处理](#批处理)
      - [在下次渲染前多次更新同一个 state](#在下次渲染前多次更新同一个-state)
      - [命名惯例](#命名惯例)
    - [useState - 回调函数的参数](#usestate---回调函数的参数)
  - [useEffect](#useeffect)
    - [*useEffect* 依赖项](#useeffect-依赖项)
    - [一、不传依赖项](#一不传依赖项)
    - [二、依赖项为空数组](#二依赖项为空数组)
    - [三、依赖项为非空数组](#三依赖项为非空数组)
    - [四、清理副作用](#四清理副作用)
    - [五、useEffect - 发送网络请求](#五useeffect---发送网络请求)
  - [useRef](#useref)
    - [使用场景](#使用场景)
    - [使用步骤](#使用步骤)
    - [自定义一个hook](#自定义一个hook)
  - [useMemo/useCallback](#usememousecallback)
  - [useContext](#usecontext)

## userState

> 📌TODO：源码阅读

### 1. 基础使用

**作用**

useState为函数组件提供状态（state）

**使用步骤**

1. 导入 `useState` 函数
2. 调用 `useState` 函数，并传入状态的初始值
3. 从`useState`函数的返回值中，拿到状态和修改状态的方法
4. 在JSX中展示状态
5. 调用修改状态的方法更新状态

```react&#x20;tsx
import { useState } from 'react'

function App() {
  // 参数：状态初始值比如,传入 0 表示该状态的初始值为 0
  // 返回值：数组,包含两个值：1 状态值（state） 2 修改该状态的函数（setState）
  const [count, setCount] = useState(0)
  return (
    <button onClick={() => { setCount(count + 1) }}>{count}</button>
  )
}
export default App
```

### 2. 状态的读取和修改

**读取状态**

该方式提供的状态，是函数内部的局部变量，可以在函数内的任意位置使用

**修改状态**

1. setCount是一个函数，参数表示`最新的状态值`
2. 调用该函数后，将使用新值替换旧值
3. 修改状态后，由于状态发生变化，会引起视图变化

**注意事项**

1. 修改状态的时候，一定要使用新的状态替换旧的状态，不能直接修改旧的状态，尤其是引用类型

### 3. 组件的更新过程

`const [count, setCount] = useState(0);`

函数组件使用 **useState** hook 后的执行过程，以及状态值的变化

- 组件第一次渲染
    1. 从头开始执行该组件中的代码逻辑
    2. 调用 `useState(0)` 将传入的参数作为状态初始值，即：0
    3. 渲染组件，此时，获取到的状态 count 值为： 0
- 组件第二次渲染
    1. 点击按钮，调用 `setCount(count + 1)` 修改状态，因为状态发生改变，所以，该组件会重新渲染
    2. 组件重新渲染时，会再次执行该组件中的代码逻辑
    3. 再次调用 `useState(0)`，此时 **React 内部会拿到最新的状态值而非初始值**，比如，该案例中最新的状态值为 1
    4. 再次渲染组件，此时，获取到的状态 count 值为：1

注意：**useState 的初始值(参数)只会在组件第一次渲染时生效**。也就是说，以后的每次渲染，useState 获取到都是最新的状态值，React 组件会记住每次最新的状态值

### 4. 使用规则

1. `useState` 函数可以执行多次，每次执行互相独立，每调用一次为函数组件提供一个状态
2. `useState` 注意事项

    只能出现在函数组件或者其他hook函数中

    不能嵌套在if/for/其它函数中（react按照hooks的调用顺序识别每一个hook）

### 摘要

- 当一个组件需要在多次渲染间“记住”某些信息时使用 state 变量。
- State 变量是通过调用 `useState` Hook 来声明的。
- Hook 是以 `use` 开头的特殊函数。它们能让你 “hook” 到像 state 这样的 React 特性中。
- Hook 可能会让你想起 import：它们需要在非条件语句中调用。调用 Hook 时，包括 `useState`，仅在组件或另一个 Hook 的顶层被调用才有效。
- `useState` Hook 返回一对值：当前 state 和更新它的函数。
- 你可以拥有多个 state 变量。在内部，React 按顺序匹配它们。
- State 是组件私有的。如果你在两个地方渲染它，则每个副本都有独属于自己的 state。
- 在一个 React 应用中一次屏幕更新都会发生以下三个步骤：

    触发

    渲染

    提交

### 批处理

设置组件 state 会把一次重新渲染加入队列。

**React 会等到事件处理函数中的 *****所有***** 代码都运行完毕再处理你的 state 更新**

（餐厅里帮你点菜的服务员。服务员不会在你说第一道菜的时候就跑到厨房！相反，他们会让你把菜点完，让你修改菜品，甚至会帮桌上的其他人点菜。）

```react&#x20;tsx
setNumber(0 + 1);//1
setNumber(0 + 1);//1
setNumber(0 + 1);//1
```

#### 在下次渲染前多次更新同一个 state

但是如果你想在下次渲染之前多次更新同一个 state，你可以像 `setNumber(n => n + 1)` 这样传入一个**根据队列中的前一个 state 计算下一个 state 的 *****函数***，而不是像 `setNumber(number + 1)` 这样传入 *下一个 state 值*。这是一种告诉 React “用 state 值做某事”而不是仅仅**替换它**的方法。

```react&#x20;tsx
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

`n => n + 1` 被称为 **更新函数**

传递给 `setNumber` state 设置函数的内容：

- **一个更新函数**（例如：`n => n + 1`）会被添加到队列中。
- **任何其他的值**（例如：数字 `5`）会导致“替换为 `5`”被添加到队列中，已经在队列中的内容会被忽略。

事件处理函数执行完成后，React 将触发重新渲染。在重新渲染期间，React 将处理队列。更新函数会在渲染期间执行，因此 \*\*更新函数必须是 \*\*[**纯函数**](https://zh-hans.react.dev/learn/keeping-components-pure "纯函数") 并且只 *返回* 结果。不要尝试从它们内部设置 state 或者执行其他副作用。在严格模式下，React 会执行每个更新函数两次（但是丢弃第二个结果）以便帮助你发现错误。

#### 命名惯例

通常可以通过相应 state 变量的第一个字母来命名更新函数的参数：

```react&#x20;tsx
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

> 注意：setState的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。

<https://github.com/sisterAn/blog/issues/26>

### useState - 回调函数的参数

**使用场景**
参数只会在组件的初始渲染中起作用，后续渲染时会被忽略。如果初始 state 需要通过计算才能获得，则可以传入一个函数，在函数中计算并返回初始的 state，此函数只在初始渲染时被调用

```react&#x20;tsx
const [name, setName] = useState(()=>{   
  // 编写计算逻辑    return '计算之后的初始值'
})
```

**语法规则**

  1. 回调函数return出去的值将作为 `name` 的初始值
  2. 回调函数中的逻辑只会在组件初始化的时候执行一次

**语法选择**

1. 如果就是初始化一个普通的数据 直接使用 `useState(普通数据)` 即可
2. 如果要初始化的数据无法直接得到需要通过计算才能获取到，使用`useState(()=>{})`

```react&#x20;tsx
import { useState } from 'react'

function Counter(props) {
  const [count, setCount] = useState(() => {
    return props.count //初始值为调用组件的时候传入
  })
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
    </div>
  )
}

function App() {
  return (
    <>
      <Counter count={10} />
      <Counter count={20} />
    </>
  )
}

export default App
```

## useEffect

**什么是副作用**
副作用是相对于主作用来说的，一个函数除了主作用，其他的作用就是副作用。对于 React 组件来说，**主作用就是根据数据（state/props）渲染 UI**，除此之外都是副作用（比如，手动修改 DOM）

**常见的副作用**

1. 数据请求 ajax发送
2. 手动修改dom
3. localstorage操作

    useEffect函数的作用就是为react函数组件提供副作用处理的

`useEffect`函数会在浏览器完成布局和绘制之后，下一次重新渲染之前执行，保证不会阻塞浏览器对屏幕的更新

`useEffect` 的第一个参数是 effect 的回调，第二个参数是 deps 依赖项，可选，类型是数组，会根据依赖项，决定是否调用 EffectCallback。

### *useEffect* 依赖项

### 一、不传依赖项

**调用时机：** 在不传 deps 依赖项情况下，*useEffect* 会在**每次渲染后都执行**，即它在第一次渲染之后\_和\_每次更新之后都会执行。相当于在 `componentDidMount` 和 `componentDidUpdate` 两个类方法中执行。

适合于 effect 的回调里计算量小，不会触发渲染的情形。

### 二、依赖项为空数组

**调用时机：** 当不需要在每次渲染后都执行，希望**只在它第一次渲染之后执行**，我们可以把 *useEffect* 的依赖项设为空数组，相当于以前的 class 组件的 `componentDidMount`。

例子：

1. 设定 Interval 定时器。

    让 state count 每秒自加一。如果不传依赖项，就会每秒都 `setInterval`, 不符合需求。正确做法是，只在第一次渲染之后执行。

    ```react&#x20;tsx
    function Count(){
      const [index, setIndex] = useState(0)
      
      useEffect(()=>{
        const id = setInterval(() => {
          setIndex(index => index+1)
        }, 1000)
      }, [])
    }
    ```

2. 进行数据请求。

    ```react&#x20;tsx
    function Comment(){
      const [commentList, setCommentList] = setState([])
      
      useEffect(() => {
        getComList().then(res => {
          setCommentList(res.data)
        })
      },[])
    }
    ```

### 三、依赖项为非空数组

**调用时机：** 当依赖项为非空数组，则 **在每次渲染后，非空数组上的 state 有更新**时，就会执行 *EffectCallback*，即 *useEffect* 传入回调函数。

> 📌**这里的更新是指什么呢？是指对依赖项进行浅比较**，对数组中的每个state，进行上次渲染后的值，跟这次渲染后的值，进行**浅比较**，不相等时，就执行 *EffectCallback*。
>
> > 对于**值类型的浅比较是对比值是否相等**。
> >
> > 对于**引用类型的浅比较是对比对象的引用，对比对象指向的内存地址**。（当依赖项是引用类型时，React 会对比当前渲染下的依赖项和上次渲染下的依赖项的内存地址是否一致，如果一致，`effect` 不会执行，只有当对比结果不一致时，`effect` 才会执行。

### 四、清理副作用

如果想要清理副作用 可以在副作用函数中的末尾return一个新的函数，在新的函数中编写清理副作用的逻辑

注意执行时机为：

1. 组件卸载时自动执行

    ```react&#x20;tsx
    function Count(){
      const [index, setIndex] = useState(0)
      
      useEffect(()=>{
        const id = setInterval(() => {
          setIndex(index => index+1)
        }, 1000)
        return ()=> clearInterval(id)
      }, [])
    }
    ```

2. 组件更新时，下一个useEffect副作用函数执行之前自动执行

    ```react&#x20;tsx
    import { useEffect, useState } from "react"

    const App = () => {
      const [count, setCount] = useState(0)
      useEffect(() => {
        const timerId = setInterval(() => {
          setCount(count + 1)
        }, 1000)
        return () => {
          // 用来清理副作用的事情
          clearInterval(timerId)
        }
      }, [count])
      return (
        <div>
          {count}
        </div>
      )
    }

    export default App
    ```

### 五、useEffect - 发送网络请求

在useEffect中发送网络请求，并且封装同步 async await操作

```react&#x20;tsx
useEffect(()=>{   
    async function fetchData(){      
       const res = await axios.get('http://geek.itheima.net/v1_0/channels')                            console.log(res)   
    } 
},[])
```

**语法要求**
不可以直接在useEffect的回调函数外层直接包裹 await ，因为**异步会导致清理函数无法立即返回**

```react&#x20;tsx
错误的：
useEffect(async ()=>{    
    const res = await axios.get('http://geek.itheima.net/v1_0/channels')   
    console.log(res)
},[])
```

## useRef

### 使用场景

在函数组件中获取真实的dom元素对象或者是组件对象

useRef 有一个名为 current 的属性，用于在任何时候检索被引用对象的值，同时也接受一个初始值作为参数。你可以通过更新当前值来更改引用对象的值。

```react&#x20;tsx
import { useRef } from ‘react’

const myComponent = () => {
  const refObj = useRef(initialValue)
  
  return (
  //…
  )
}
```

```react&#x20;tsx
// 在函数内部
const handleRefUpdate = () => {
    // 访问被引用对象的值
    const value = refObj.current

    // 更新被引用对象的值
   refObj.current = newValue
}
```

- 被引用对象的值在重新渲染之间保持不变。
- 更新被引用对象的值不会触发重新渲染。

### 使用步骤

1. 导入 `useRef` 函数
2. 执行 `useRef` 函数并传入null，返回值为一个对象 内部有一个current属性存放拿到的dom对象（组件实例）
3. 通过ref 绑定 要获取的元素或者组件
4. **获取Dom**

    ```react&#x20;tsx
    import { useEffect, useRef } from 'react'
    function App() {  
        const h1Ref = useRef(null)  
        useEffect(() => {    
            console.log(h1Ref)  
        },[])  
        return (    
            <div>      
                <h1 ref={ h1Ref }>this is h1</h1>    
            </div>  
        )
    }
    export default App
    ```

5. **获取组件实例**
    > 函数组件由于没有实例，不能使用ref获取，如果想获取组件实例，必须是**类组件**

    ```react&#x20;tsx
    class Foo extends React.Component {  
        sayHi = () => {    
            console.log('say hi')  
        }  
        render(){    
            return <div>Foo</div>  
        }
    }
        
    export default Foo
    ```

    ```react&#x20;tsx
    import { useEffect, useRef } from 'react'
    import Foo from './Foo'
    function App() {  
        const h1Foo = useRef(null)  
        useEffect(() => {    
            console.log(h1Foo)  
        }, [])  
        return (    
            <div> <Foo ref={ h1Foo } /></div>  
        )
    }
    export default App
    ```

&#x20;

### 自定义一个hook

创建了一个自定义 hook，它接受一个引用对象作为 `ref` 和一个回调函数，然后我们执行一个事件监听器来检查鼠标何时被单击，如果单击不在当前的 `ref` 上，那么我们就触发回调函数。

```react&#x20;tsx
import React, { useEffect } from 'react'
 
export default function useClickAway(ref: any, callback: Function) {
  useEffect(() => {
    function handleClickAway(event: any) {
      if (ref.current && !ref.current.contains(event.target)) { //********/
        callback();
      }
    }
    document.addEventListener("mousedown", handleClickAway);
    return () => {
      document.removeEventListener("mousedown", handleClickAway);
    };
  }, [ref]);
 };
```

**需求描述**：自定义一个hook函数，实现获取滚动距离Y

```react&#x20;tsx
import { useState, useEffect } from "react"

export function useWindowScroll () {
  const [y, setY] = useState(0)
  useEffect(()=>{
     const scrollHandler = () => {
        const h = document.documentElement.scrollTop
        setY(h)
     })
     window.addEventListener('scroll', scrollHandler)
     return () => window.removeEventListener('scroll', scrollHandler)
  })
 
  return [y]
}
```

**需求描述：** 自定义hook函数，可以自动同步到本地LocalStorage

> `const [message, setMessage] = useLocalStorage(key，defaultValue)`
>
> 1. message可以通过自定义传入默认初始值
> 2. 每次修改message数据的时候 都会自动往本地同步一份

```react&#x20;tsx
import { useEffect, useState } from 'react'

export function useLocalStorage (key, defaultValue) {
  const [message, setMessage] = useState(defaultValue)
  // 每次只要message变化 就会自动同步到本地ls
  useEffect(() => {
    window.localStorage.setItem(key, message)
  }, [message, key])
  return [message, setMessage]
}
```

## useMemo/useCallback

`useMemo(()=>{},[])`

useMemo接收两个参数，分别是函数和一个数组（实际上是依赖），函数里return 函数，数组内存放依赖。

```react&#x20;tsx
const addM = useMemo(() => {
    return () => {
      setM({ m: m.m + 1 });
    };
  }, [m]); //表示监控m变化
```

useCallback

语法糖

```react&#x20;tsx
  const addM = useCallback(() => {
    setM({ m: m.m + 1 });
  }, [m]);
```

- 使用`memo`可以帮助我们优化性能，让`react`没必要执行不必要的函数
- 由于复杂数据类型的地址可能发生改变，于是传递给子组件的`props`也会发生变化，这样还是会执行不必要的函数，所以就用到了`useMemo`这个api
- `useCallback`是`useMemo`的语法糖

`useMemo` 和 `useCallback` 都是用来帮助我们优化 **重新渲染**

- 减少在一次渲染中需要完成的工作量。
- 减少一个组件需要重新渲染的次数。

<https://juejin.cn/post/7165338403465068552>

<http://docs.lanhanba.net/fe/@lhb%2Fhook?name=docs>

## useContext

**实现步骤**

1. 使用`createContext` 创建Context对象
2. 在顶层组件通过`Provider` 提供数据
3. 在底层组件通过`useContext`函数获取数据

```react&#x20;tsx
import { createContext, useContext } from 'react'
// 创建Context对象
const Context = createContext()

function Foo() {  
    return <div>Foo <Bar/></div>
}

function Bar() {  
    // 底层组件通过useContext函数获取数据  
    const name = useContext(Context)  
    return <div>Bar {name}</div>
}

function App() {  
    return (    
        // 顶层组件通过Provider 提供数据    
        <Context.Provider value={'this is name'}>     
            <div><Foo/></div>    
        </Context.Provider>  
    )
}

export default App
```

```react&#x20;tsx
interface paramsType {
//省id
provinceId: number ;
//城市id
cityId: number;
//区域id；
districtId: number；
}
```
