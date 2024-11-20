---
title: "[译]为什么我使用Vue来替代React"
date: 2024-11-18T10:36:25+08:00
lastmod: 2024-11-18T10:36:25+08:00
draft: false
keywords: ["vue","react","框架选择"]
description: "翻译国外的文章，记录作者通过对比的方式来决定如何在Vue和React中做出选择"
tags: ["vue","react"]
categories: ["翻译","Web编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: true
borderImage: false
codeTabSeperator: "::"
moreMeta: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

本文翻译自[**Why I chose Vue over React**](https://medium.com/@serpentarium13/why-i-chose-vue-over-react-b082d81315ab)。

<!--more-->

![封面图](/blog_img/translate/web/why-i-chose-vue-over-react/react-vs-vue.webp "封面图") 

## 简介

当前的`JavaScript`生态系统不容乐观，有大量的东西：不同的库、框架以自己的方式实现了相同的逻辑。在我看来，这说不上好坏，我们有更多的选择，但对于初学者来说由于无法真正的识别好坏，这可能会很痛苦，虽然受欢迎程度能协助判断，但它们都其实很受欢迎。



我个人认为在该领域有两个有利的竞争者:`React`和`Vue`，很多人争论哪种更好，本文根据我个人的观点和经验来阐述这一话题。

## 开始

我开始学习`React`的时候对其了解并不多，只完成了一些关于`HTML`、`CSS`和`JavaScript`的课程学习。由于`React`当时最受欢迎，我认为学习它是进入该领域的正确方式，从刚发布了应用路由器的`Next.js`开始，阅读文档并完成了我的第一个项目，它是一个专门用于分享梦想的留言板。我有点喜欢Carl Jung，如果你知道我说的是啥，当完成它时，我感觉良好，它包含了授权、配置页面、表单和一些动画方面的良好实践。如果你想查看，可点击[此连接](https://workshop-rebuild.vercel.app/)(当前只有加载和表单功能正常工作)。



尽管我已经从`React`中获得了自己想要的东西，我仍然在寻求新的机会，这就是我发现`Vue.js`起始。它已经处于成熟阶段，已经完成了向Composition API的过滤并且看起来几乎与`React`相同。一开始我被其文档吓着了，但看了几次之后我就爱上了它并且现在可以很肯定的说它是我最喜欢的web编程框架。

## 学习曲线

那时我还没有真正的接触过`React`的文档，我从`Vue`文档开始阅读由创建者自己编写的教程，而不是由某些内容制作者编写的教程。虽然目前`React`的文档十分成熟，我仍然认为`Vue`的文档在学习上更好，优点如下：

* 文档解释了一切，`React`中的一些关键点和隐藏行为没有被很好的记录，其文档只覆盖了基本功能，因而没有太多让每个`React`开发者都知道的技巧以便与当前潮流同步。
* 官方学习课程，Vueschool.io和Vuemastery.com是学习`Vue`的一个很好的起点，而`React`有一些文档，其中大部分可能已经过时了。
* 开发者友好的API，对我而言，`Vue`拥有更好、更独特的工具库可以用它来开发应用程序。从第1页开始就有详细的描述，相对于`React`其赋予了我们对应用程序更多的控制。除此之外，文档中也有太多对新手而言的隐藏项(展示周期，useEffect)。

当然，学习`React`对我后来学习`Vue`提供了很大帮助，在对一个框架有扎实的基础和一些理解之后，可以更快的学习其它框架。尽管如此，我仍然给新人推荐`Vue`，其在开发方面比`React`更加宽容，并且很难出现失误。

## 更好的选择

如前所述，`Vue`拥有比`React`更好的自定义API，关键特性如下：

### 不用useEffect

这意味着不需要处理对单一原则视而不见的人所带来的缺点，在`React`中，`useEffect`是一个工具中两个不同的东西，组件生命周期的守护者和副作用的创造者。

```javascript
useEffect(() => {
  // do something
}, [])
```

这段代码有啥作用？生效！

```javascript
useEffect(() => {
  return () => // do something after
}, [])
```

这段呢？生效并返回？

```java
useEffect(() => {

}, [a, b, c, d, eslint-error-that-suggests-to-gtfo, e, j])
```

数组?!



当你只拥有一个具有无限数量用例的愚蠢抽象是，为什么要保留这些`onMounted`, `onUnmounted`, `onBeforeMount`, `onRouteChange` 和`watch`?想在组件安装时触发某些操作？提供一个空组数就行，为啥？因为抽象！



确实，有一些约定或许大部分`React`开发者都知道，为啥不将其引入库中让体验变得更好呢？自己动手吧。

### 两种数据绑定方式

`React`采用`Flux`架构，简单地说就是数据只能单向流动，采用此种方式可以方便追踪数据变更，因为只有一个事实来源，尽管`Vue`几乎相同，其也有一些优点。



首先是输入绑定，在`React`中需要编写如下代码来将输入值绑定到组件状态

```javascript
export const Input() => {
    
  const [inputValue, setInputValue] = useState<string>("")
  
  const handleChange = (ev: ChangeEvent<HTMLInputElement>) => {
      const {value} = ev.target as HTMLInputElement;
      setInputValue(value)
  }

  return <input onChange={handleChange} value={inputValue}/>
}
```

每个输入组件都需要这么干，而在`Vue`中只需要如下

```html
<template>
  <input v-model="inputValue"/>
</template>

<script lang="ts" setup>
   const inputValue = ref<string>("")
</script>
```

是的，虽然可以在`React`中编写一个自定义钩子用于处理所有功能而不是在所有的组件中重写，但为啥会这样？



第二个是在子组件和父组件之间的双向绑定，这意味着你可以通过props与父组件进行沟通，从子节点通过emit返回父节点,在`React`中通过子组件处理数据时需要编写如下代码：

```javascript
const Parent = () => {
  const handleData = (data: IData) => // do something
}

const Child = ({handleData}: {handleData: (data: IData) => any}) => {
  return <button onClick={handleData}> HandleData! </button>
}
```

在`Vue`中对应的实现为

```html
//Child
<template>
  <button @click="$emit('ev')"> Emit! </button> 
</template>

<script lang="ts" setup>

defineEmits<{ev: [something: string]}>();

</script>

//Parent
<template>
  <Child @ev="handleEv"/>
</template>

<script lang="ts" setup>
const handleEv = (something: string) => console.log("Handled", something) 
</script>
```

哪种方式更好呢？假设在`React`中有一个包含20个属性的组件(虽然是一个很少出现的边界场景，但仍然可能出现)，该组件中属性接口代码看起来如下：

```javascript
interface ComponentProps {
  ...valueProperties...
  onChange: Function,
  onEmit: Function,
  onPause: Function,
  onPuke: Function,
  onRerender: Function,
  onCry: Function,
  onShit: Function,
  onDeath: Function,
  onLeave: Function,
  onMeme: Function
}
```

是的，你可以理解函数绑定从`on`开始，但为什么呢？若函数没有传递给组件，可能会导致你的应用程序崩溃。而在`Vue`中，它们将通过`@`进行区分，所有其他的开发者都能更好的理解发生了什么。

### 编译器而非库

另一个优点是`Vue`为`.vue`文件提供了编译器，而不是纯`JavaScript`表达式。咋一听可能有点可怕并且过于复杂，但当你深入了解`React`中`jsx`的可怕之处以我下面将要提及的一些东西之后，你就会明白我在说啥。



当`React`引入`jsx`的时候，有些人有些反感，从某种程度上来说他们是对的。在我们脑海中经典的web由3部分组成，`HTML`、`CSS`和`JavaScript`，它们应该通过文件扩展名来区分，关注点分离应该可以起作用。但在现实世界中，有很多可重用的逻辑，并且考虑到基于组件的接口创建方法的初衷，我认为将它们结合起来是一个不错的决定，因为在三个文件之间切换肯定不是一种好的体验。但`React`做到了，`Vue`做得更好，原因如下

1. 不需要`CSS`文件，`Vue`有一个作用域样式块可用于放置所有的样式，如果不需要可以不被其他组件共享，基于此，`React`中的`CSS-in-JS`全部过时，同时不需要过度设计和创建大量的`css modules`仅仅只是为了在其中编写十几行代码，同时如果不喜欢的话，也可以避免强制使用`Tailwind`.

2. 更好的约定，由于没有`jsx`不会让人迷糊，想通过一个html标签来触发事件(如点击、选择)？只需要添加`@`并实现其即可。

   ```html
   <input @change="handleChange">
   ```

   想展示innerHtml？不用担心，不需要记住下述代码

   ```html
   <div dangerouslySetInnerHTML={{__html: data}} />
   ```

   只需要像下面这样做即可

   ```html
   <div v-html="data"/>
   ```

   代码越多也会有潜在风险，你懂的。

   需要设置多个css类名称？连接字符串即可！

   ```javascript
   className={`${aboba} ${boba} ${voba} ${roba ? 'roba' : ''}` }
   ```

   想通过其它方式实现？用[这个类库](https://github.com/lukeed/clsx)(虽然其提交小于1kb，但依旧是一个外部依赖)。

3. 对于属性继承处理的更好，例如在`React`中将`className`传递给一个将来可能动态变化的组件，需要将其写入对应属性并在稍后传递给组件的根html标签。

   ```javascript
   export const Button = ({className}: {className: string}) => {
     return <button className={className}> Classified </button>
   }
   ```

   而在`Vue`中只需要这么写

   ```html
   <template>
     <Child class="abc"/>
   </template>
   ```

   无需指定组件需要接收它，是的，这很含糊，但如果你认为这就是`React`可以变得更好的原因 —— 你似乎并没有为每个边界场景的组件编写属性，在这种情况下它需要不应该成为其主题一部分的边框。

   

   另外一个场景是有时候不需要将属性直接传递给根标签，`Vue`已经为我们提供了一个解决方案

   ```html
   <template>
     <div>
       <div v-bind="$attrs"/>
     </div>
   </template>
   ```

4. slot可以更好的实现子节点，在`React`中若想传递一些模板到子组件中需要在组件属性中定义一个`children`属性，之后使用`jsx`花括号来渲染。

   ```javascript
   const Component = ({children}: {children: ReactNode}) => {
     return <div> {children} </div>
   }
   ```

   而在`Vue`中可以更容易的实现，同时还提供了更多的自由来实现动态组件。

   a.相对于具体的子节点实现，可使用一个`</slot>`标签来处理子节点

   ```html
   <template>
     <div>
       <slot/>
     </div>
   </template>
   ```

   b.也可以同时有多个命名的slots 

   ```html
   <template>
     <div>
       <slot/>
     </div>
     <div>
       <slot name="chicken"/>
     </div>
   </template>
   
   // Parent
   
   <template>
   <Child>
     <template #default>
       Goes in the default slot
     </template>
     <template #chicken>
       Chicken!
     </template>
   </Child>
   
   </template>
   ```

   c.Slot可以暴露数据到父组件

   ```html
   <template>
     <div>
       <slot :data={data}/>
     </div>
   </template>
   // Parent
   <template>
   
   <Child>
     <template #default={data}>
       Using data!
     </template>
   </Child>
   
   </template>
   ```

   这对于制作可重复使用的组件非常有用，这些组件可以与其他组件很好地组合使用（例如，封装库的组件（swiper 等））

### 自行渲染

`React`的出现带来了启示，利用当时占主导地位的技术，我们必须自己编写所有重新渲染逻辑，现在出现了一个终极的解决方案，可以消除其它所有问题 - 只需重新渲染所有内容！同时也解决了一些问题让开发更简单，虽然确实有缺陷，相对于检测并重新渲染，我们需要不重新渲染。



它在以前运行的很好，但我们特别是那些刚刚开始编写代码的人，不太真正的需要这么干。我并不是说我们不应该掌握渲染周期并了解其内部工作原理，但为什么每个人都需要了解它，而一些更新、更好的技术可以为我们做到这一点呢？



虽然`Vue`的渲染机制仍然同`React`一样依赖虚拟DOM(尤雨溪说2024年会有新的东西)，与超酷的`Svelte`和Solid不同，它仍然比`React`好得多，因为它不是愚蠢地重新渲染所有内容，而是做了其他事情。在编译时，`Vue`保持对每个需要更新的DOM节点的追踪，之后进行更新。不再有`useMemo`、`useCallback`和因过多Reactive而导致的过时技术带来的性能障碍。



在`Vue`中，我们更倾向寻找一个方法来重新渲染组件，以避免此问题，我认为这种方式比处理开发人员制造的很少出现的边界场景更好。

### 内置构建

至于一些适合包含在UI渲染解决方案中的实用程序，例如`React`和`Vu`，`Vue`也占了上风。当我想再次尝试`React`时，我遇到了需要使用`Vue`提供的开箱即用的类似`Transition`组件之类的东西。有了它，可以处理正在重新渲染的组件的动画。对于特定问题，`React`确实有一个针对的[解决方案库](https://reactcommunity.org/react-transition-group/)，但了解到`Vue`的实现之后，我的眼泪流了出来，开始哭泣并感到非常抱歉，我只是用纯 CSS 处理了动画，看看这个：

```html
<template> 
  <Transition>
    <some-div/>
  </Transition>
</template>

<style scoped>
.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.v-enter-active,
.v-leave-active {
  transition: 0.25s ease all;
}
</style>
```

你应该知道它要干啥。

接下来看看这个

```javascript
const defaultStyle = {
  transition: `opacity ${duration}ms ease-in-out`,
  opacity: 0,
}

const transitionStyles = {
  entering: { opacity: 1 },
  entered:  { opacity: 1 },
  exiting:  { opacity: 0 },
  exited:  { opacity: 0 },
};

function Fade({ in: inProp }) {
  const nodeRef = useRef(null);
  return (
    <Transition nodeRef={nodeRef} in={inProp} timeout={duration}>
      {state => (
        <div ref={nodeRef} style={{
          ...defaultStyle,
          ...transitionStyles[state]
        }}>
          I'm a fade Transition!
        </div>
      )}
    </Transition>
  );
}
```

![哭了](/blog_img/translate/web/why-i-chose-vue-over-react/my-eyes.gif "哭了") 

想缓存组件以便它能记住之前的数据来获得更好的性能？安装相关库！

而在`Vue`中只需要如下

```html
<KeepAlive>
  <div/>
</KeepAlive>
```

让人兴奋的是`React`已经推出了`Suspense`,`package.json`不够大到容纳它们3个。

## 总结

思考这篇文章让我得出了我认为在这场框架争论中最好的结论：你可以在任何地方做任何事，只是实现方法不同而已。如果你了解一些基础知识，就能对遇到的每个库和框架保持冷静，如果只是盯着抽象层并比较它们，将会迷失。但当要选择使用哪个框架/工具时，我建议在做出结论之前先尝试验证下。



对我而言，`React`感觉像一个有技巧的老战士，但是越来越旧了，同时而一些新晋的产品则状态良好，不受过去各种束缚，它们正在探索寻找更多。此外，这位老战士受雇于组织，而年轻的一代则是人民英雄，努力在阳光下赢得一席之地，并通过战斗将无数人联系起来(我的意思是`Vue`是真正的开源，并且不受任何类似 Meta 的支持)。



它们都战斗并获胜，老的产品(`React`)拥有多年来积累的大量粉丝群，而`Vue`的粉丝们则尖叫着承诺会支持它直到最后。



如果没有`React`，很可能就不会出现`Vue`这种框架，但是如果没有`Vue`和其他框架(如`Svelte`和`Solid`)，未来只会停滞不前。



**感谢您花时间阅读这篇文章，希望您度过一个愉快的夜晚，无论在干啥！玩得开心！**