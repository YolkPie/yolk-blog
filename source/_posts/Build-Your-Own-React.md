---
title: "Build Your Own React"
date: "2021-05-12 14:00:00"
cover: "https://m.360buyimg.com/img/jfs/t1/178814/36/3674/40468/609b6c56Ec2d63c98/7425bd2580d884e1.png"
---


# Build Your Own React

1. Step I: The createElement Function（createElement 函数）
2. Step II: The render Function （render 函数）
3. Step III: Concurrent Mode（并发模式）
4. Step IV: Fibers
5. Step V: Render and Commit Phases
6. Step VI: Reconciliation
7. Step VII: Function Components
8. Step VIII: Hooks

## Step Zero: Review

在正式开始之前，首先先回顾一下一些基本的概念。

```js
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

```jsx
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
);

```
第一行使用jsx定义元素，React.createElement从传入的参数创建一个对象。

通过类似于babel的构建工具转换为js。转换通常很简单:将标记内的代码替换为对createElement的调用，将 tag name, props and the children作为参数传递。


```jsx
// 通过 React.createElement 创建的element对象如下：
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
```

关于element,它是一个对象，其中有一些属性(type, key, ref, self, source, owner, props)，这里我们只关注 type和props。

type是一个字符串，它的值是我们想要创建的DOM元素的标签名，也是传递给文档的标签名。当你想要创建一个HTML元素时，可以使用createElement，它也可以是一个函数。

props也是一个对象，它拥有来自JSX属性的所有键和值。它还有一个特殊的属性：children。

我们需要替换的另一段React代码是对ReactDOM.render的调用。render是React更改DOM的地方，所以由我们自己进行更新。


```js
// Render
const container = document.getElementById("root")
​
const node = document.createElement(element.type)
node["title"] = element.props.title
​
const text = document.createTextNode("")
text["nodeValue"] = element.props.children
​
node.appendChild(text)
container.appendChild(node)

```

1. 首先，我们使用 element 的 type 创建一个节点，在本例中为`h1`.
2. 然后我们将 props 分配给节点，本例中为 `title`（比较熟悉的为class 和 id，可查看[HTML全局属性](https://www.w3school.com.cn/tags/html_ref_standardattributes.asp)）。为了避免混淆，使用element来引用React元素，使用node来引用DOM元素。
3. 接下来为 children 创建节点，这里children是一个string，为其创建一个text节点。使用textNode而不是设置innerText将允许我们以后以相同的方式处理所有元素。还请注意是如何设置nodeValue的，就像在h1标题中设置的一样，它几乎就像字符串的props一样:{nodeValue: "hello"}。
4. 最后我们增加这个 textNode 到 h1 中，并将h1附加到 container 中。

```js
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
​
const container = document.getElementById("root")
​
const node = document.createElement(element.type)
node["title"] = element.props.title
​
const text = document.createTextNode("")
text["nodeValue"] = element.props.children
​
node.appendChild(text)
container.appendChild(node)
```

以上为使用js完成创建元素到渲染元素的代码实现。

## Step I: The createElement Function

```jsx
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
ReactDOM.render(element, container)

const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

现在开始写一个我们自己版本的React来代替React的版本。

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
```

子数组也可以包含字符串或数字等基本值。因此，我们将把所有不是对象的东西都包装在它自己的元素中，并为它们创建一个特殊类型:TEXT元素。当没有子元素时，React不会包装原始值或创建空数组，这么做的目的是简化代码。

接下来，我们自定义一个名字来代替React

```jsx
const Didact = {
  createElement,
}

// const element = Didact.createElement(
//   "div",
//   { id: "foo" },
//   Didact.createElement("a", null, "bar"),
//   Didact.createElement("b")
// )
​/** @jsx Didact.createElement */

const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

但是我们仍然想在这里使用JSX的语法。我们如何告诉babel使用Didact的createElement而不是React的。

可以通过 `/** @jsx Didact.createElement */`注释来告诉babel。当babel编译JSX时，它将使用我们定义的函数。

## Step II: The render Function

目前，我们只关心向DOM添加内容。稍后我们将处理更新和删除。

```jsx
function render(element, container) {
  // TODO create dom nodes
}
​
const Didact = {
  createElement,
  render,
}

...

Didact.render(element, container)
```

我们首先使用元素类型创建DOM节点，然后将新节点附加到容器中。

```js
// 1
function render(element, container) {
  const dom = document.createElement(element.type)
​
  // 2. 递归
  element.props.children.forEach(child =>
    render(child, dom)
  )

  container.appendChild(dom)
}
```

如果元素类型是文本元素，我们将创建一个文本节点而不是常规节点。

render函数不支持的一件事是文本节点。首先，我们需要定义文本元素的外观。例如，<span>Foo</span>在React中描述的元素如下所示：

```js
const reactElement = {
  type: "span",
  props: {
    children: "Foo" // 是孩子, 但也只是一个字符串
  }
};
```

我们可以注意到，这里文本节点的children值是一个String，这里其实违背了我们最初的定义 ‘props可能有一个 children 属性，它应该是一个 Didact Elements 数组。’

```js
  // 3
 const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
```

这里我们需要做的最后一件事是将props分配给节点。

```js
  // 4
  // 当节点没有子元素的时候执行 props的分配
  // 这里的key和下面forEach的name本质上是一样的
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      // 这里的name为对象中的属性名
      // dom节点上的id值就等于element.props['id']的值
      // dom['id'] = element.props['id']
      dom[name] = element.props[name]
    })
```

以下为完整版的

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}
​
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
​
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
​
const Didact = {
  createElement,
  render,
}
​
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
Didact.render(element, container)

```

## Step III: Concurrent Mode

在添加更多的代码之前，我们需要对刚才所写的进行重构。

这是因为我们刚刚在render函数里写的一个递归调用。一旦我们开始执行渲染函数的时候，在渲染完成之前我们都不能停止，如果需要渲染的元素过多的话，这个渲染函数可能会执行太长时间。如果浏览器需要做高优先级的事情，比如处理用户输入或者保持动画流畅，就不得不等待渲染完成。


所以我们要把这个过程分成小的单元，当我们完成每个单元后，如果还有其他需要做的事情，我们会让浏览器中断渲染。

```js
let nextUnitOfWork = null
​
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}

```

我们使用[requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)来做一个循环。您可以将requestIdleCallback视为setTimeout，但是我们不会告诉它何时运行，而是在主线程空闲时，浏览器将运行回调。

React不再使用requestIdleCallback。现在它使用 scheduler package。但是对于这个用例，它在概念上是相同的。

requestIdleCallback还为我们提供了一个deadline参数。我们可以用它来检查我们有多少时间，直到浏览器需要再次采取控制。

截至2019年11月，React中的并发模式还不稳定。循环的稳定版本看起来更像这样

```js
while (nextUnitOfWork) {    
  nextUnitOfWork = performUnitOfWork(   
    nextUnitOfWork  
  ) 
}
```

要开始使用循环，我们需要设置第一个工作单元，然后编写performUnitOfWork函数，该函数不仅执行工作，而且还返回下一个工作单元。

## Step IV: Fibers

Fiber 是 React 16 中新的协调引擎。它的主要目的是使 Virtual DOM 可以进行增量式渲染

Fiber是怎么样的？

```js
let fiber = {
  tag: HOST_COMPONENT,
  type: "div",
  parent: parentFiber,
  child: childFiber,
  sibling: null,
  alternate: currentFiber,
  stateNode: document.createElement("div"),
  props: { children: [], className: "foo"},
  partialState: null,
  effectTag: PLACEMENT,
  effects: []
};
```

为了组织工作单元，我们需要一个数据结构: 一个 Fibers（纤程） 树。

1. nextUnitOfWork将是对下一个工作 Fiber 的参考.
2. performUnitOfWork拿到 Fiber,并在其上工作, 并返回一个新的 Fiber 用于下一次 - 直到所有工作完成.


每个元素都有一个fiber，每个fiber都是一个工作单元

假设我们现在想渲染一个像下面这样的 element tree

```js
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>,
  container
)
```


在渲染中，我们将创建root fiber并将其设置为 nextUnitOfWork。剩下的工作将在performUnitOfWork函数上进行，在那里我们将为每一个 fiber做三件事：

1. 增加一个元素到DOM中
2. 为元素的子元素 创建 fibers 
3. 选择下一个单元进行工作


这种数据结构的目标之一是使查找下一个工作单元变得容易。这就是为什么每个 fiber都与它的第一个子元素、下一个兄弟元素和父元素相连。

![](https://tva1.sinaimg.cn/large/00831rSTgy1gdjv84aj13j30ma0oo74u.jpg)

当我们完成对一个 fiber 的工作时，如果它有一个子元素，那么这个子元素将是下一个工作单元。在我们的示例中，当我们完成对div fiber 的工作时，下一个工作单元将是h1 fiber。

如果 fiber既没有子元素也没有兄弟元素，我们就去找叔叔:父母的兄弟姐妹。比如例子中的a和h2 fiber

同样，如果父结点没有兄弟结点，我们继续通过父结点，直到找到有兄弟结点的父结点，或者到达根结点。如果我们已经到达了根节点，这意味着我们已经完成了渲染的所有工作。


```js
// 这里是之前 的render
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
​
let nextUnitOfWork = null
```

现在先让我们将render从以上的代码中移除。我们将创建DOM节点的部分保留在它自己的函数中，稍后我们将使用它


```js
function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type)
​
  const isProperty = key => key !== "children"
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name]
    })
​
  return dom
}
​
function render(element, container) {
  // TODO set next unit of work
}
​
let nextUnitOfWork = null
```

在渲染函数中，我们将nextUnitOfWork设置为光纤树的根。

```js
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}
​
let nextUnitOfWork = null
```
然后，当浏览器准备好了，它将调用我们的 workLoop，我们将从root开始进行render

首先，我们创建一个新节点并将其附加到DOM。我们跟踪fiber.dom 属性中 的dom节点。

```js
function performUnitOfWork(fiber) {
  // 1. TODO add element to Dom
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  // 2. TODO create new fibers
  // 3. TODO return next unit of work
}
```

然后我们为每一个child 创建一个新的 fiber, 我们把它添加到Fibers中把它设置成子结点或者兄弟结点，这取决于它是不是第一个子结点。


```js
  // 2. create new fibers
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
  }

  if (index === 0) {
    fiber.child = newFiber
  } else {
    prevSibling.sibling = newFiber
  }
​
  prevSibling = newFiber
  index++

​
```

最后，我们寻找下一个工作单元。我们首先对子元素进行测试，然后对兄弟元素进行测试，然后对父元素的兄弟元素进行测试，等等。

```js
// 3. return next unit of work
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
```

这就是 performUnitOfWork 函数

```js
function performUnitOfWork(fiber) {
  // 1. TODO add element to Dom
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  // 2. create new fibers
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  // 3. return next unit of work
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
​
```

## Step V: Render and Commit Phases

我们这里还有另一个问题。

```js

  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }

```
每次处理元素时，我们都会向DOM添加一个新节点。 而且，请记住，在完成渲染整个树之前，浏览器可能会中断我们的工作。 在这种情况下，用户将看到不完整的UI。 而且我们不想要那样。

因此，我们需要从此处删除更改DOM的部分。

相反，我们将跟踪 Fibers 的根。我们称它为“正在进行的工作”

```js
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let wipRoot = null
```

一旦我们完成了所有的工作(我们知道它是因为没有下一个工作单元)，我们将整个Fibers提交到DOM。

```js
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }

  // NOTE: 判断是否将整个Fibers提交到DOM
  if (!nextUnitOfWork && wipRoot) {
    commitRoot()
  }
​
  requestIdleCallback(workLoop)
}
```

我们在commitRoot函数中做到这一点。 在这里，我们将所有节点递归附加到dom。

```js
function commitRoot() {
  commitWork(wipRoot.child)
  wipRoot = null
}
​
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

## Step VI: Reconciliation

到目前为止，我们只向DOM添加了一些东西，但是更新或删除节点又该如何操作呢？

这就是我们现在要做的，我们需要将渲染函数（render）上接收到的元素与提交给DOM的最后一个Fibers进行比较

```js
function commitRoot() {
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot,
  }
  nextUnitOfWork = wipRoot
}

let currentRoot = null
```

因此，我们需要保存对提交完成后提交到DOM的最后一个Fibers的引用。我们称之为currentRoot。

我们还为每个 fiber 添加了 alternate 属性。此属性链接到旧的 fiber，即我们在前一个提交阶段提交到DOM的 fiber。

现在让我们从创建新 fiber 的 performUnitOfWork 中提取代码，到一个新的reconcileChildren函数

```js
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  reconcileChildren(fiber, elements)
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}

```

在reconcileChildren函数中，我们将调和旧的 fiber 和新的元素。

```js
function reconcileChildren(wipFiber, elements) {
  let index = 0

  // NOTE: 旧的Fiber
  let oldFiber = wipFiber.alternate && wipFiber.alternate.child
  let prevSibling = null
  //NOTE: ​while ( index < elements.length || oldFiber != null) {...
}
  while ( index < elements.length || oldFiber != null) {
    const element = elements[index]
    let newFiber = null
    // NOTE: 比较
    const sameType =
      oldFiber &&
      element &&
      element.type == oldFiber.type
  ​
    if (sameType) {
      // ①. update the node
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      }
    }
    if (element && !sameType) {
      // ② add this node
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT",
      }
    }
    if (oldFiber && !sameType) {
      // ③ delete the oldFiber's node
      oldFiber.effectTag = "DELETION"
      deletions.push(oldFiber)
    }
  }
}
```

我们同时对旧fiber (wipFiber.alternate)的子元素和我们想要协调的元素数组进行迭代。

如果我们忽略同时遍历一个数组和一个链表所需的所有样板文件，那么在while中最重要的部分就剩下了:oldFiber和element。元素是我们想要呈现给DOM的东西，而oldFiber是我们上次呈现的东西。我们需要对它们进行比较，看看是否需要对DOM进行任何更改。

我们需要对它们进行比较，看看是否需要对DOM进行任何更改。

我们用类型来比较它们：

1. 如果旧的 Fiber 和新的 element 具有相同的类型，我们可以保留DOM节点并 使用新的 props 进行更新
2. 如果类型不同并且有一个新元素，这意味着我们需要创建一个新的DOM节点
3. 如果类型不同，有一个旧的 fiber，我们需要删除旧的节点

在这里React也会使用 keys，这使得 reconciliation 更好。例如，它检测子元素在元素数组中的位置何时改变。

①. 当旧的 fiber 和元素具有相同的类型时，我们创建一个新 fiber，以保持DOM节点不受旧 fiber 的影响，而props不受元素的影响。我们还为 fiber 添加了一个新属性:effectTag，值为 'UPDATE'。稍后，在提交阶段，我们将使用此属性。

②. 然后，对于元素需要新的DOM节点的情况，我们使用 effectTag 为 'PLACEMENT' 标记标记新的fiber。

③. 对于需要删除节点的情况，我们没有新的fiber，所以我们将effect标签添加到旧的fiber中。

但是，当我们将fiber tree提交到DOM时，我们从正在进行的工作根中执行，根中没有旧的fibers。因此，我们需要一个数组来跟踪要删除的节点。

```js
function render(element, container) {
  ...
  deletions = []
  ...
}
​
let deletions = null
```

然后，当我们将更改提交到DOM时，我们还将使用来自该数组的 fiber。

```js
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
```

现在，让我们更改commitWork函数来处理新的 effectTags

如果fiber具有以一个 'PLACEMENT' 的 effect tag，我们将执行与前面相同的操作，将DOM节点追加到来自父 fiber 的节点。如果是'DELETION'，则相反，删除子节点。如果是 UPDATE，则需要使用 props 更新现有的DOM节点。

```js
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom

 if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
  ) {
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    )
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }

  commitWork(fiber.child)
  commitWork(fiber.sibling)
}

```

增加一个 updateDom 函数，我们将旧Fiber的props和新Fiber的props进行对比，去掉已经消失的props，设置新的或者更改过的props。

```js
const isProperty = key => key !== "children"
const isNew = (prev, next) => key => prev[key] !== next[key]
const isGone = (prev, next) => key => !(key in next)

function updateDom(dom, prevProps, nextProps) {
  // 1. Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = ""
    })
​
  // 2. Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name]
    })
}
​
```

我们需要更新的一种特殊类型的 props 是事件监听器，因此如果 props 名称以on前缀开头，我们将以不同的方式处理它们。

如果事件处理程序发生更改，则将其从节点中删除。然后我们添加新的处理器。

```js
const isEvent = key => key.startsWith("on")
const isProperty = key =>
  key !== "children" && !isEvent(key)
function updateDom(dom, prevProps, nextProps) {
  //3. Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(key =>!(key in nextProps) || isNew(prevProps, nextProps)(key))
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.removeEventListener(
        eventType,
        prevProps[name]
      )
    })

  // 4. Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.addEventListener(
        eventType,
        nextProps[name]
      )
    })
}
```

## Step VII: Function Components

接下来我们需要添加的是对函数组件的支持。首先让我们更改示例。我们将使用这个简单的函数组件，它返回一个h1元素。

```js
/** @jsx Didact.createElement */
function App(props) {
  return <h1>Hi {props.name}</h1>
}
const element = <App name="foo" />
const container = document.getElementById("root")
Didact.render(element, container)
```
注意，如果我们将jsx转换成js，它将是
```jsx
function App(props) {
  return Didact.createElement(
    "h1",
    null,
    "Hi ",
    props.name
  )
}
const element = Didact.createElement(App, {
  name: "foo",
})
```

函数组件有两种不同的方面:

1. 来自函数组件的fiber没有DOM节点
2. 子组件通过运行函数而不是直接从props获取

```js
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  reconcileChildren(fiber, elements)
  ...
}
```

我们检查fiber类型是否是一个函数，根据这个函数，我们进入一个不同的更新函数。在updateHostComponent中，我们执行与前面相同的操作。

```js
function performUnitOfWork(fiber) {
  const isFunctionComponent =
    fiber.type instanceof Function
  if (isFunctionComponent) {
    updateFunctionComponent(fiber)
  } else {
    updateHostComponent(fiber)
  }
  
  ...
}
​
function updateFunctionComponent(fiber) {
  // TODO
}
​
function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
  reconcileChildren(fiber, fiber.props.children)
}

```

在updateFunctionComponent中，我们运行函数来获取子元素。以fiber为例。类型是App函数，当我们运行它时，它返回h1元素。然后，一旦我们有了子元素，reconciliation 以同样的方式进行，我们不需要改变任何东西。

```js
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
```

我们需要改变的是commitWork函数。现在我们有了没有DOM节点的fiber，我们需要改变两件事。

首先，要找到DOM节点的父节点，我们需要沿着fiber tree往上走，直到找到带有DOM节点的fiber为止。

由`const domParent = fiber.parent.dom` 变为以下

```js
let domParentFiber = fiber.parent
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent
  }
  const domParent = domParentFiber.dom
​
```

在删除节点时，我们还需要继续操作，直到找到带有DOM节点的子节点。

```js
 else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParent)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
​
function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom)
  } else {
    commitDeletion(fiber.child, domParent)
  }
}
```

## Step VIII: Hooks

最后一步。现在我们有了函数组件，让我们加上状态。

让我们将示例更改为典型的计数器组件。每次点击它，状态都会增加1。请注意，我们正在使用Didact。获取和更新计数器值。

```js
const Didact = {
  createElement,
  render,
  useState,
}
​
/** @jsx Didact.createElement */
function Counter() {
  const [state, setState] = Didact.useState(1)
  return (
    <h1 onClick={() => setState(c => c + 1)}>
      Count: {state}
    </h1>
  )
}
const element = <Counter />
const container = document.getElementById("root")
Didact.render(element, container)
```

这里是我们从例子中调用计数器函数的地方。在这个函数中，我们调用useState

```js
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
​
function useState(initial) {
  // TODO
}

```

我们需要在调用函数组件之前初始化一些全局变量，以便可以在useState函数内部使用它们。首先，我们把工作放在fiber中进行。我们还将一个hooks数组添加到fiber中，以支持在同一个组件中多次调用useState。我们跟踪当前的hookIndex。

```js
let wipFiber = null
let hookIndex = null
​
function updateFunctionComponent(fiber) {
  wipFiber = fiber
  hookIndex = 0
  wipFiber.hooks = []
  const children = [fiber.type(fiber.props)]
  ...
}
```

当函数组件调用useState时，我们检查是否有旧的hooks。使用hookIndex来检查fiber的交替。

如果我们有一个旧hooks，我们将状态从旧hooks复制到新hooks，否则，我们将初始化状态。

然后将新hooks添加到fibers中，将hookIndex增加1，并返回状态。

```js
function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex]
  const hook = {
    state: oldHook ? oldHook.state : initial,
  }
​
  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state]
}
```

useState还应该返回一个用于更新状态的函数，因此我们定义了一个setState函数，该函数接收一个action（对于Counter示例，此动作是将状态加1的函数）。

我们将该动作推送到添加到Hooks上的队列中。

然后，我们执行与在render函数中所做的类似的操作，将新的进行中的工作根设置为下一个工作单元，以便工作循环可以开始新的渲染阶段。

```js
function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex]
  const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: [],
  }

  // actions

  const setState = action => {
    hook.queue.push(action)
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    }
    nextUnitOfWork = wipRoot
    deletions = []
  }
​
​
  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state, setState]
}
```

但是我们还没有开始执行。我们在下次渲染组件时这样做，我们会从旧的Hoos队列中获取所有action，然后一个接一个地将它们应用到新的Hooks状态中，这样当我们返回状态时，它就更新了。


```js
 const actions = oldHook ? oldHook.queue : []
  actions.forEach(action => {
    hook.state = action(hook.state)
  })
```

现在，我们已经构建了自己的React版本。

除了帮助理解React如何工作外，本文的目标之一是使人更容易深入了解React代码库。这就是为什么我们在几乎所有地方都使用相同的变量和函数名。例如，如果在真实的React应用程序的某个函数组件中添加断点，调用堆栈应该会显示:workLoop、performUnitOfWork、updateFunctionComponent。

我们没有包含很多的React特性和优化。例如，有几件事情的反应是不同的:在Didact中，我们在渲染阶段遍历整个树。React遵循一些提示和启发来跳过没有变化的整个子树。我们还在提交阶段遍历整个树。React保持一个链表，只访问有效果的fiber，只访问那些fiber。每当我们构建一个新的工作进展树，我们为每个fibers创建新的对象。React回收利用旧树的fiber。当Didact在渲染阶段收到一个新的更新时，它会丢弃正在进行的工作树，并从根节点重新开始。React为每个更新添加一个过期时间戳，并使用它来决定哪个更新具有更高的优先级。


附完整js：

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object" ? child : createTextElement(child)
      )
    }
  };
}

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: []
    }
  };
}

function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type);

  updateDom(dom, {}, fiber.props);

  return dom;
}

const isEvent = key => key.startsWith("on");
const isProperty = key => key !== "children" && !isEvent(key);
const isNew = (prev, next) => key => prev[key] !== next[key];
const isGone = (prev, next) => key => !(key in next);
function updateDom(dom, prevProps, nextProps) {
  //Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(key => !(key in nextProps) || isNew(prevProps, nextProps)(key))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2);
      dom.removeEventListener(eventType, prevProps[name]);
    });

  // Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = "";
    });

  // Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name];
    });

  // Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2);
      dom.addEventListener(eventType, nextProps[name]);
    });
}

function commitRoot() {
  deletions.forEach(commitWork);
  commitWork(wipRoot.child);
  currentRoot = wipRoot;
  wipRoot = null;
}

function commitWork(fiber) {
  if (!fiber) {
    return;
  }

  let domParentFiber = fiber.parent;
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }
  const domParent = domParentFiber.dom;

  if (fiber.effectTag === "PLACEMENT" && fiber.dom != null) {
    domParent.appendChild(fiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom != null) {
    updateDom(fiber.dom, fiber.alternate.props, fiber.props);
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParent);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom);
  } else {
    commitDeletion(fiber.child, domParent);
  }
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element]
    },
    alternate: currentRoot
  };
  deletions = [];
  nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null;
let wipRoot = null;
let deletions = null;

function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }

  if (!nextUnitOfWork && wipRoot) {
    commitRoot();
  }

  requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
  const isFunctionComponent = fiber.type instanceof Function;
  if (isFunctionComponent) {
    updateFunctionComponent(fiber);
  } else {
    updateHostComponent(fiber);
  }
  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.parent;
  }
}

let wipFiber = null;
let hookIndex = null;

function updateFunctionComponent(fiber) {
  wipFiber = fiber;
  hookIndex = 0;
  wipFiber.hooks = [];
  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}

function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex];
  const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: []
  };

  const actions = oldHook ? oldHook.queue : [];
  actions.forEach(action => {
    hook.state = action(hook.state);
  });

  const setState = action => {
    hook.queue.push(action);
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot
    };
    nextUnitOfWork = wipRoot;
    deletions = [];
  };

  wipFiber.hooks.push(hook);
  hookIndex++;
  return [hook.state, setState];
}

function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber);
  }
  reconcileChildren(fiber, fiber.props.children);
}

function reconcileChildren(wipFiber, elements) {
  let index = 0;
  let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
  let prevSibling = null;

  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;

    const sameType = oldFiber && element && element.type == oldFiber.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE"
      };
    }
    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT"
      };
    }
    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      wipFiber.child = newFiber;
    } else if (element) {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}

const Didact = {
  createElement,
  render,
  useState
};

/** @jsx Didact.createElement */
function Counter() {
  const [state, setState] = Didact.useState(1);
  return (
    <h1 onClick={() => setState(c => c + 1)} style="user-select: none">
      Count: {state}
    </h1>
  );
}
const element = <Counter />;
const container = document.getElementById("root");
Didact.render(element, container);

```