# JSX

## 目录

- [JSX](#jsx)
  - [目录](#目录)
  - [1. JSX](#1-jsx)
  - [2. JSX中使用js表达式](#2-jsx中使用js表达式)
  - [3. JSX列表渲染](#3-jsx列表渲染)
  - [4. JSX条件渲染](#4-jsx条件渲染)
  - [5. JSX样式处理](#5-jsx样式处理)
  - [6. JSX注意事项](#6-jsx注意事项)

## 1. JSX

概念：JSX是 JavaScript XML（HTML）的缩写，表示在 JS 代码中书写 HTML 结构

作用：在React中创建HTML结构（页面UI结构）

优势：

1. 采用类似于HTML的语法，降低学习成本，会HTML就会JSX
2. 充分利用JS自身的可编程能力创建HTML结构

注意：JSX 并不是标准的 JS 语法，是 JS 的语法扩展，浏览器默认是不识别的，脚手架中内置的 [@babel/plugin-transform-react-jsx](@babel/plugin-transform-react-jsx "@babel/plugin-transform-react-jsx") 包，用来解析该语法

## 2. JSX中使用js表达式

**语法**

`{ JS 表达式 }`

**可以使用的表达式**

1. 字符串、数值、布尔值、null、undefined、object（ \[] / {} ）
2. 1 + 2、'abc'.split('')、\['a', 'b'].join('-')
3. fn()

> **特别注意**：
>
> if 语句/ switch-case 语句/ 变量声明语句，这些叫做语句，不是表达式，不能出现在 `{}` 中！！

## 3. JSX列表渲染

> 页面的构建离不开重复的列表结构，比如歌曲列表，商品列表等，vue中用的是v-for，react如下👇

实现：使用数组的`map` 方法

```react&#x20;jsx
// 列表
const songs = [
  { id: 1, name: 'hello1' },
  { id: 2, name: 'hello2' },
  { id: 3, name: 'hello3' }
]

export default function App() {
  return (
    <div className="App">
      <ul>
        {
          songs.map(item => <li>{item.name}</li>)
        }
      </ul>
    </div>
  )
}

```

> 📌注意点：需要为遍历项添加 `key` 属性

1. key 在 HTML 结构中是看不到的，是 React 内部用来进行性能优化时使用
2. key 在当前列表中要唯一的字符串或者数值（String/Number）
3. 如果列表中有像 id 这种的唯一值，就用 id 来作为 key 值
4. 如果列表中没有像 id 这种的唯一值，就可以使用 index（下标）来作为 key 值

## 4. JSX条件渲染

作用：根据是否满足条件生成HTML结构，比如Loading效果

实现：可以使用 `三元运算符` 或 `逻辑与(&&)运算符`

## 5. JSX样式处理

能够在JSX中实现css样式处理

- 行内样式 - style&#x20;

    ```react&#x20;jsx
    <div style={{ color: 'red' }}>this is a div</div>

    //
    const styleObj = {
        color:red
    }
    ...
    <div style={ styleObj }>this is a div</div>
    ...

    ```

- &#x20;类名 - className —动态控制

    ```react&#x20;jsx
    .title {
      font-size: 30px;
      color: blue;
    }

    <div className={ showTitle ? 'title' : ''}>this is a div</div>

    ```

## 6. JSX注意事项

1. JSX必须有一个根节点，如果没有根节点，可以使用`<></>`（幽灵节点）替代
2. 所有标签必须形成**闭合**，成对闭合或者自闭合都可以
3. JSX中的语法更加贴近JS语法，属性名采用\*\*驼峰命名法 \*\*`class -> className` `for -> htmlFor`
4. JSX支持多行（换行），如果需要换行，需使用`()` 包裹，防止bug出现
