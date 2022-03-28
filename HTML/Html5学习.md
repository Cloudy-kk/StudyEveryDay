
# HTML5/CSS3

## 什么是BFC？BFC的布局规则是什么？如何创建BFC？BFC应用？

BFC 是 Block Formatting Context 的缩写，即块格式化上下文。B**FC是CSS布局的一个概念，是一个环境，里面的元素不会影响外面的元素。**
**布局规则**：Box是CSS布局的对象和基本单位，页面是由若干个Box组成的。元素的类型和display属性，决定了这个Box的类型。不同类型的Box会参与不同的Formatting Context。
**创建**：

- **根元素**或其它包含它的元素；
- **浮动** (元素的`float`不为`none`)；
- **绝对定位元素** (元素的`position`为`absolute`或`fixed`)；
- **行内块**`inline-blocks`(元素的 `display: inline-block`)；
- **表格单元格**(元素的`display: table-cell`，HTML表格单元格默认属性)；
- `overflow`的值不为`visible`的元素；
- **弹性盒 flex boxes** (元素的`display: flex`或`inline-flex`)；



## **清除浮动方法：**

- 父级div定义height
- 结尾处加空div标签clear:both
- 父级div定义伪类:after和zoom
- 父级div定义overflow:hidden
- 父级div也浮动，需要定义宽度
- 结尾处加br标签clear:both
- 比较好的是第3种方式



## CSS优先级算法如何计算？

优先级**就近原则，同权重情况下样式定义最近者为准**
载入样式以最后载入的定位为准（后载入的高于前面的，因为会被覆盖）
优先级为:** !important（规则最重要，大于其它规则） > id（1000） > class（10） > tag（1）**; !important 比 内联优先级高

## 请描述一下 cookies，sessionStorage 和 localStorage 的区别？

- cookie是网站为了标示用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密)
- cookie数据始终在同源的http请求中携带（即使不需要），记会在浏览器和服务器间来回传递
- sessionStorage（关闭浏览器自动删除）和localStorage（主动删除）不会自动把数据发给服务器，仅在本地保存
- cookie 设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭

## 简述一下src与href的区别？

- **src用于替换当前元素，href用于在当前文档和引用资源之间确立联系。**
- src是source的缩写，指向外部资源的位置，指向的内容将会嵌入到文档中当前标签所在位置；在请求src资源时会将其指向的资源下载并应用到文档内，例如js脚本，img图片和frame等元素

< script src ="js.js"> < /script> **当浏览器解析到该元素时，会暂停其他资源的下载和处理**，直到将该资源加载、编译、执行完毕，图片和框架等元素也如此，类似于将所指向资源嵌入当前标签内。这也是为什么将js脚本放在底部而不是头部

- href 是Hypertext Reference的缩写，**指向网络资源所在位置**，建立和当前元素（锚点）或当前文档（链接）之间的链接，如果我们在文档中添加< link href="common.css" rel="stylesheet"/> **那么浏览器会识别该文档为css文件，就会并行下载资源并且不会停止对当前文档的处理**。这也是为什么建议使用link方式来加载css，而不是使用@import方式



## H5有哪些新特性。移除了哪些元素：

HTML5 现在已经不是 SGML 的子集，主要是关于图像，位置，存储，多任务等功能的增加

- 绘画 **canvas**
- 用于媒介回放的**video** 和**audio** 元素
- 本地离线存储**localStorage**长期存储数据，浏览器关闭后数据不丢失
- **sessionStorage** 的数据在浏览器关闭后自动删除
- 语意化更好的内容元素：**article、footer、header、nav、section**
- 表单控件：**calendar、date、time、email、url、search**
- 新的技术：**webworker、 websocket、 Geolocation**

语义化的优点有:

- 代码结构清晰，易于阅读，利于开发和维护
- 方便其他设备解析（如屏幕阅读器）根据语义渲染网页。
- 有利于搜索引擎优化（SEO），搜索引擎爬虫会根据不同的标签来赋予不同的权重

## 浏览器内核：
- 主要分成两部分：**渲染引擎**(layout engineer或Rendering Engine)和**JS引擎**
	渲染引擎：**负责取得网页的内容（HTML、XML、图像等等）、整理讯息（例如加入CSS等），以及计算网页的显示方式，然后会输出至显示器或打印机**。浏览器的内核的不同对于网页的语法解释会有不同，所以渲染的效果也不相同。所有网页浏览器、电子邮件客户端以及其它需要编辑、显示网络内容的应用程序都需要内核
	JS引擎：解析和执行javascript来实现网页的动态效果



![](image/image.png "")





## 三角形

/*记忆口诀：盒子宽高均为零，三面边框皆透明。 除了top*/

```CSS
<style>
    #sjx {
      position: absolute;
      width: 0px;
      height: 0px;
      content: " ";
      border-right: 100px solid transparent;
      border-top: 100px solid #ff0;
      border-left: 100px solid transparent;
      border-bottom: 100px solid transparent;
    }
  </style>
```




## Flex布局：

```CSS
display:flex
```


采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。

它的所有**子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。**

flex-direction

```CSS
.box {
    flex-direction: row | row-reverse | column | column-reverse;
}

row 表示从左向右排列
row-reverse 表示从右向左排列
column 表示从上向下排列
column-reverse 表示从下向上排列

```


![](image/image_1.png "")

flex-wrap

```CSS
.box {
    flex-wrap: nowrap | wrap | wrap-reverse;
}

nowrap(缺省)：所有Flex项目单行排列
wrap：所有Flex项目多行排列，按从上到下的顺序
wrap-reverse：所有Flex项目多行排列，按从下到上的顺序

```


![](image/image_2.png "")

flex-flow

flex-wrap flex-firection的缩写

```CSS
.box {
    flex-flow: <‘flex-direction’> || <‘flex-wrap’>
}
```




justify-content

项目在主轴上面的对其方式以及额外空间的分配方式

```CSS
.box  {
    justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
}
- `flex-start`(缺省)：从启点线开始顺序排列
- `flex-end`：相对终点线顺序排列
- `center`：居中排列
- `space-between`：项目均匀分布，第一项在启点线，最后一项在终点线
- `space-around`：项目均匀分布，每一个项目两侧有相同的留白空间，相邻项目之间的距离是两个项目之间留白的和
- `space-evenly`：项目均匀分布，所有项目之间及项目与边框之间距离相等

```


![](image/image_3.png "")



容器的属性：

- flex-direction：决定主轴的方向（即子 item 的排列方法）flex-direction: row | row-reverse | column | column-reverse;
- flex-wrap：决定换行规则 flex-wrap: nowrap | wrap | wrap-reverse;
- flex-flow： .box { flex-flow: || ; }
- justify-content：项目对其方式，水平主轴对齐方式
- align-items：项目对齐方式，竖直轴线方向
- align-content：在交叉轴方向的对齐方式及额外空间分配

项目的属性（元素的属性）：

- order 属性：定义项目的排列顺序，顺序越小，排列越靠前，默认为 0
- flex-grow 属性：定义项目的放大比例，即使存在空间，也不会放大
- flex-shrink 属性：定义了项目的缩小比例，1当空间不足的情况下会等比例的缩小，如果 定义个 item 的 flow-shrink 为 0，则为不缩小，负值无效
- flex-basis 属性：定义了在分配多余的空间，项目占据的空间。
- flex：是 flex-grow 和 flex-shrink、flex-basis 的简写，默认值为 0 1 auto。
- align-self：允许单个项目与其他项目不一样的对齐方式，可以覆盖





## 居中：

### 水平居中

- 对于** 行内元素** : `text-align: center`;
- 对于**块级元素**：
	（1）width和margin实现。`margin: 0 auto`;(子元素包含float：left)
		css3新特性：fit-content
	```CSS
	.parent{
	/*   width: -moz-fit-content;*/
	/*   width: -webkit-fit-content;*/
	    width:fit-content;
	    margin:0 auto;
	}
	```
	
	（2）flex布局中的justify-content
	```CSS
	.parent{
	    display: flex;
	    justify-content: center;
	}
	```
	
	（3）css3新特性：transform
	![](image/image_4.png "")
	```CSS
	.son{
	    position:absolute;
	      left:50%;
	      transform:translate(-50%,0);
	}
	```
	
	（4）绝对定位，负值margin-left
	```CSS
	.son{
	    position:absolute;
	    width:固定;
	    left:50%;
	    margin-left:-0.5宽度;
	}
	```
	

### 垂直居中

- 对于** 行内元素** : line-height 等于父元素高度
	```CSS
	
	```
	
- 对于**行内块级元素**：
	①：若元素是行内块级元素, 基本思想是使用display: inline-block, vertical-align: middle和一个伪元素让内容块处于容器中央.
	![](image/image_5.png "")
	```CSS
	.parent::after, .son{
	  display:inline-block;
	  vertical-align:middle;
	}
	
	```
	
	（2）**元素高度不定**
		①：为了使用vertical-align, 我们需要设置父元素display:table, 子元素 display:table-cell;vertical-align:middle;
		②：
		```CSS
		.parent {
		  display: flex;
		  align-items: center;
		}
		```
		
		③：
		```CSS
		.son{
		    position:absolute;
		    top:50%;
		    transform: translate(0,-50%);
		}
		
		```
		
	（3）**元素高度固定**
		①
		```CSS
		.son{
		    position:absolute;
		    top:50%;
		    height:固定;
		    margin-top:-0.5高度;
		}
		```
		
		②：设置父元素相对定位(position:relative), 子元素如下css样式
		```CSS
		.son{
		    position:absolute;
		    height:固定;
		    top:0;
		    bottom:0;
		    margin:auto 0;
		}
		```
		

水平居中较为简单, 共提供了8种方法, 一般情况下 text-align:center,marin:0 auto; 足矣

- ① text-align:center;
- ② margin:0 auto;
- ③ width:fit-content;
- ④ flex
- ⑤ 盒模型
- ⑥ transform
- ⑦ ⑧ 两种不同的绝对定位方法

垂直居中, 共提供了8种方法.

- ① 单行文本, line-height
- ② 行内块级元素, 使用 display: inline-block, vertical-align: middle; 加上伪元素辅助实现
- ③ vertical-align
- ④ flex
- ⑤ 盒模型
- ⑥ transform
- ⑦ ⑧ 两种不同的绝对定位方法
若元素是行内块级元素, 基本思想是使用display: inline-block, vertical-align: middle和一个伪元素让内容块处于容器中央.
若元素是行内块级元素, 基本思想是使用display: inline-block, vertical-align: middle和一个伪元素让内容块处于容器中央.

## 隐藏元素：
1.`opacity：0`，该元素隐藏起来了，**但不会改变页面布局**，并且，如果该元素已经绑定 一些事件，如click 事件，那么点击该区域，也**能触发**点击事件的
2.`visibility：hidden`，该元素隐藏起来了**，但不会改变页面布局**，但是**不会触发**该元素已 经绑定的事件 ，隐藏对应元素，在文档布局中仍保留原来的空间（重绘）
3.`display：none`，把元素隐藏起来，并且**会改变页面布局**，可以理解成在页面中把该元素。 不显示对应的元素，在文档布局中不再分配空间（**回流+重绘**）

