---
title: 核心Diff算法
date: 2020-09-19 17:48:55
tags:
  - diff
  - vue
  - react
categories: diff
keywords: diff,vue,react
description: 虚拟DOM的核心之一就是它的Diff算法，其中最为核心的就是核心Diff算法，只有在新旧虚拟DOM的子节点都是多个的时候，核心Diff算法才会派上用场。
cover: https://img11.360buyimg.com/imagetools/jfs/t1/145814/24/8751/761861/5f65d465Ec74e3d82/13384de14843dee4.png
top_img: https://img11.360buyimg.com/imagetools/jfs/t1/123065/11/13011/13900/5f65d482E37a11a22/3d2e9003128c5106.jpg
---

### 前言
虚拟DOM的核心之一就是它的Diff算法，其中最为核心的就是核心Diff算法，只有在新旧虚拟DOM的子节点都是多个的时候，核心Diff算法才会派上用场。

无论何种类型的核心Diff算法，它们采用的核心思想是一致的：
- 1、找到需要移动的节点，并移动它们；
- 2、添加新的节点；
- 3、移除不存在的节点。

新旧虚拟DOM子节点的可能情况如下：

|  旧的children的个数   | 新的children的个数  |  操作  |
|  :----:  | :----:  |  ----  |
|  1  |  1  |  patch  |
|  1  |  0  |  remove  |
|  1  |  n  |  remove旧的子节点，mount新的子节点  |
|  0  |  1  |  mount  |
|  0  |  0  |  无操作  |
|  0  |  n  |  mount  |
|  n  |  1  |  remove旧的子节点，mount新的子节点  |
|  n  |  0  |  remove  |
|  n  |  n  |  核心Diff  |

### 虚拟DOM的结构
```js
export interface VNode {
  _isVNode: true                  // _isVNode是一个始终为 true 的值，有了它，我们就可以判断一个对象是否是 VNode 对象
  el: Element | null              // 当一个 VNode 被渲染为真实 DOM 之后，el 属性的值会引用该真实DOM
  flags: VNodeFlags
  tag: string | FunctionalComponent | ComponentClass | null
  data: VNodeData | null
  children: VNodeChildren
  childFlags: ChildrenFlags
}
```

说明：
- _isVNode：是一个始终为 true 的值，有了它，我们就可以判断一个对象是否是 VNode 对象。
- el：当一个 VNode 被渲染为真实 DOM 之后，el 属性的值会引用该真实DOM。
- flags：VNode 的类型。
- tag：标签名称。
- data： VNode 数据，用于对 VNode 进行描述。假如一个 VNode 的类型是 html 标签，则 VNodeData 中可以包含 class、style 以及一些事件。
- children：子节点。
- childFlags：子节点的类型。

### 无key时的核心Diff算法
我们经常会遇到一个可排序的列表，假设我们又一个由`li`标签组成的列表，它们是`ul`标签子节点，我们可以使用下面的数组来表示 ul 标签的 children：
```js
[
  h('li', null, 1),
  h('li', null, 2),
  h('li', null, 3)
]
```

接着由于数据变化导致了列表的顺序发生了变化，新的列表顺序如下：
```js
[
  h('li', null, 3),
  h('li', null, 1),
  h('li', null, 2)
]
```

我们能够注意到：更新前后的真实 DOM 元素都是 li 标签。我们可以复用 li 标签，这样就能减少“移除”和“新建” DOM 元素带来的性能开销。当新旧 VNode 所描述的是相同标签时，那么这两个 VNode 之间的差异就仅存在于 VNodeData 和 children 上，所以我们完全可以通过遍历新旧 VNode，并一一比对它们，这样对于任何一个 DOM 元素来说，由于它们都是相同的标签，所以更新的过程是不会“移除”和“新建”任何 DOM 元素的，而是复用已有 DOM 元素，需要更新的只有 VNodeData 和 children。

更新操作如下图表示：

![diff2](diff2.png)

当新的 children 比旧的 children 的长度要长时，多出来的子节点是没办法应用 patch 函数的，此时我们应该把多出来的子节点作为新的节点添加上去。类似的，如果新的 children 比旧的 children 的长度要短时，我们应该把旧的 children 中多出来的子节点移除，如下图所示：

![diff3](diff3.png)
![diff4](diff4.png)

### React的核心Diff算法
前面，我们通过减少 DOM 操作的次数使得更新的性能得到了提升，但它仍然存在可优化的空间。我们通过观察新旧 children 可以很容易的发现：新旧 children 中的节点只有顺序是不同的，所以最佳的操作应该是通过移动元素的位置来达到更新的目的。

能够移动元素的关键在于：我们需要在新旧 children 的节点中保存映射关系，以便我们能够在旧 children 的节点中找到可复用的节点。这时候我们就需要给 children 中的节点添加唯一标识，也就是我们常说的 key，在没有 key 的情况下，我们是没办法知道新 children 中的节点是否可以在旧 children 中找到可复用的节点的。

为了明确的知道新旧 children 中节点的映射关系，我们需要在 VNode 创建伊始就为其添加唯一的标识，即 key 属性。

```js
// 旧 children
[
  h('li', { key: 'a' }, 1),
  h('li', { key: 'b' }, 2),
  h('li', { key: 'c' }, 3)
]
// 新 children
[
  h('li', { key: 'c' }, 3)
  h('li', { key: 'a' }, 1),
  h('li', { key: 'b' }, 2)
]
```

有了 key 我们就能够明确的知道新旧 children 中节点的映射关系，如下图所示：

![diff5](diff5.png)

知道了映射关系，我们就很容易判断新 children 中的节点是否可被复用：只需要遍历新 children 中的每一个节点，并去旧 children 中寻找是否存在具有相同 key 值的节点。

现在我们已经找到了可复用的节点，并进行了合适的更新操作，下一步需要做的，就是判断一个节点是否需要移动以及如何移动。

我们可以先考虑不需要移动的情况，当新旧 children 中的节点顺序不变时，就不需要额外的移动操作，如下：

![diff6](diff6.png)

- 1、取出新 children 的第一个节点，即 li-a，并尝试在旧 children 中寻找 li-a，结果是我们找到了，并且 li-a 在旧 children 中的索引为 0。
- 2、取出新 children 的第二个节点，即 li-b，并尝试在旧 children 中寻找 li-b，也找到了，并且 li-b 在旧 children 中的索引为 1。
- 3、取出新 children 的第三个节点，即 li-c，并尝试在旧 children 中寻找 li-c，同样找到了，并且 li-c 在旧 children 中的索引为 2。

总结一下我们在“寻找”的过程中，先后遇到的索引顺序为：0->1->2。这是一个递增的顺序，这说明如果在寻找的过程中遇到的索引呈现递增趋势，则说明新旧 children 中节点顺序相同，不需要移动操作。相反的，如果在寻找的过程中遇到的索引值不呈现递增趋势，则说明需要移动操作。

我们在寻找过程中有一个重要的数字，寻找过程中在旧 children 中所遇到的最大索引值。如果在后续寻找的过程中发现存在索引值比最大索引值小的节点，意味着该节点需要被移动。

现在我们已经有办法找到需要移动的节点了，接下来要解决的问题就是：应该如何移动这些节点？为了弄明白这个问题，我们还是先来看下图：

![diff7](diff7.png)

新 children 中的第一个节点是 li-c，它在旧 children 中的索引为 2，由于 li-c 是新 children 中的第一个节点，所以它始终都是不需要移动的，只需要调用 patch 函数更新即可。

li-c 节点更新完毕，接下来是新 children 中的第二个节点 li-a，它在旧 children 中的索引是 0，由于 0 < 2 所以 li-a 是需要移动的节点。新 children 中的节点顺序实际上就是更新完成之后节点应有的最终顺序，只需把 li-a 节点对应的真实 DOM 移动到 li-c 节点所对应真实 DOM 的后面。

当新 children 中包含了一个全新的节点时，我们尝试在旧的 children 中寻找该节点时，是找不到可复用节点的，这时就没办法通过移动节点来完成更新操作。此时应该将该节点挂载到合适的位置。

除了要将全新的节点添加到容器元素之外，我们还应该把已经不存在了的节点移除。在我们遍历完新 children 后，再遍历一次旧的 children ，并尝试拿着旧 children 中的节点去新 children 中寻找相同的节点，如果找不到则说明该节点已经不存在于新 children 中了，这时我们应该将该节点对应的真实 DOM 移除。

至此，第一个完整的 Diff 算法我们就讲解完毕了，这个算法就是 React 所采用的 Diff 算法。但该算法仍然存在可优化的空间。

### Vue2.x的核心Diff算法

React 的 Diff 算法是存在优化空间的，想要要找到优化的关键点，我们首先要知道它存在什么问题。来看下图：

![diff8](diff8.png)

在这个例子中，我们可以通过肉眼观察从而得知最优的解决方案应该是：把 li-c 节点对应的真实 DOM 移动到最前面即可，只需要一次移动即可完成更新。然而，React 所采用的 Diff 算法在更新如上案例的时候，会进行两次移动。

采用双端比较的方式，可以来避免多余的DOM移动操作。所谓双端比较，就是同时从新旧 children 的两端开始进行比较的一种方式，所以我们需要四个索引值，分别指向新旧 children 的两端，如下图所示：

![diff9](diff9.png)

在一次比较过程中，最多需要进行四次比较：

- 1、使用旧 children 的头一个 VNode 与新 children 的头一个 VNode 比对，即 oldStartVNode 和 newStartVNode 比较对。
- 2、使用旧 children 的最后一个 VNode 与新 children 的最后一个 VNode 比对，即 oldEndVNode 和 newEndVNode 比对。
- 3、使用旧 children 的头一个 VNode 与新 children 的最后一个 VNode 比对，即 oldStartVNode 和 newEndVNode 比对。
- 4、使用旧 children 的最后一个 VNode 与新 children 的头一个 VNode 比对，即 oldEndVNode 和 newStartVNode 比对。

在如上四步比对过程中，试图去寻找可复用的节点，即拥有相同 key 值的节点。这四步比对中，在任何一步中寻找到了可复用节点，则会停止后续的步骤，可以用下图来描述在一次比对过程中的四个步骤：

![diff10](diff10.png)

每次比对完成之后，如果在某一步骤中找到了可复用的节点，我们就需要将相应的位置索引后移/前移一位。以上图为例：

- 第一步：拿旧 children 中的 li-a 和新 children 中的 li-d 进行比对，由于二者 key 值不同，所以不可复用，什么都不做。
- 第二步：拿旧 children 中的 li-d 和新 children 中的 li-c 进行比对，同样不可复用，什么都不做。
- 第三步：拿旧 children 中的 li-a 和新 children 中的 li-c 进行比对，什么都不做。
- 第四步：拿旧 children 中的 li-d 和新 children 中的 li-d 进行比对，由于这两个节点拥有相同的 key 值，所以我们在这次比对的过程中找到了可复用的节点。

由于我们在第四步的比对中找到了可复用的节点，这说明：li-d 节点所对应的真实 DOM 原本是最后一个子节点，并且更新之后它应该变成第一个子节点。所以我们需要把 li-d 所对应的真实 DOM 移动到最前方即可。

对于上面的四步比较，需要进行移动操作的其实只有两步，包括刚刚的第四步，以及第三步。如果在第三步中找到了可以复用的节点，说明该节点原本是第一个节点，在更新后变成了最后一个节点，所以需要把它对应的真是 DOM 移动到最后面即可。

下面来处理非理想的情况，如果在上述四个步骤中均无法找到可以复用的节点，我们只能拿新 children 中的第一个节点尝试去旧 children 中寻找，试图找到拥有相同 key 值的节点。如果找到则意味着：旧 children 中的这个节点所对应的真实 DOM 在新 children 的顺序中，已经变成了第一个节点。所以我们需要把该节点所对应的真实 DOM 移动到最前头。

我们尝试拿着新 children 中的第一个节点去旧 children 中寻找与之拥有相同 key 值的可复用节点，然后并非总是能够找得到，当新的 children 中拥有全新的节点时，就会出现找不到的情况，我们应该将其挂载到容器中。

同时我们需要考虑循环结束后新 children 有剩余节点的情况，此时也需要将剩余的节点挂载到容器中。

最后一个需要考虑的情况是：当有元素被移除时的情况。如果在循环结束后旧的 children 有剩余测节点，则这些剩余的节点需要被移除。

以上就是相对完整的双端比较算法的实现，这是 Vue2.x 所采用的算法，借鉴于开源项目：[snabbdom](https://github.com/snabbdom/snabbdom)。

### Vue3.x的核心Diff算法

优化核心Diff算法，本质上还是要避免核心Diff算法的执行。所以在真正执行核心Diff算法前先进行预处理，去除相同的前缀与后缀节点，仅对它们执行 patch 操作。

![diff11](diff11.png)

在去掉相同的前缀与后缀节点后，如果新旧 children 均有剩余节点，此时需要执行核心Diff算法。

无论是 React 的 Diff 算法，还是 Vue2(snabbdom) 的 Diff 算法，其重点无非就是：判断是否有节点需要移动，以及应该如何移动和寻找出那些需要被添加或移除的节点。

首先，我们需要构造一个数组 source，该数组的长度等于新 children 在经过预处理之后剩余未处理节点的数量，并且该数组中每个元素的初始值为 -1，如下图所示：

![diff12](diff12.png)

source 数组将用来存储新 children 中的节点在旧 children 中的位置，后面将会使用它计算出一个最长递增子序列，并用于 DOM 移动。如下图所示：

![diff13](diff13.png)

可以看到 source 数组的第四个元素值仍然为初始值 -1，这是因为新 children 中的 li-g 节点不存在于旧 children 中。除此之外，还有一件很重要的事儿需要做，即判断是否需要移动节点，判断的方式类似于 React 所采用的方式。

对于移动节点，我们会根据 source 数组计算出一个最长递增子序列，并用于 DOM 移动操作。

出数组 source 的最长递增子序列为 [ 0, 1 ]。我们知道 source 数组的值为 [2, 3, 1, -1]，很显然最长递增子序列应该是 [ 2, 3 ]，但为什么计算出的结果是 [ 0, 1 ] 呢？其实 [ 0, 1 ] 代表的是最长递增子序列中的各个元素在 source 数组中的位置索引，如下图所示：

![diff14](diff14.png)

我们对新 children 中的剩余未处理节点进行了重新编号，li-c 节点的位置是 0，以此类推。而最长递增子序列是 [ 0, 1 ] 这告诉我们：新 children 的剩余未处理节点中，位于位置 0 和位置 1 的节点的先后关系与他们在旧 children 中的先后关系相同。或者我们可以理解为位于位置 0 和位置 1 的节点是不需要被移动的节点，即上图中 li-c 节点和 li-d 节点将在接下来的操作中不会被移动。换句话说只有 li-b 节点和 li-g 节点是可能被移动的节点，但是我们发现与 li-g 节点位置对应的 source 数组元素的值为 -1，这说明 li-g 节点应该作为全新的节点被挂载，所以只有 li-b 节点需要被移动。

使用两个索引 i 和 j 分别指向新 children 中剩余未处理节点的最后一个节点和最长递增子序列数组中的最后一个位置，并从后向前遍历。判断当前节点的位置索引值 i 是否与子序列中位于 j 位置的值相等，如果不相等，则说明该节点需要被移动；如果相等则说明该节点不需要被移动，并且会让 j 指向下一个位置。为了将节点挂载到正确的位置，我们需要找到当前节点的真实位置索引(i + nextStart)，以及当前节点的后一个节点，并挂载该节点的前面即可。

对于最长递增子序列的求解，这是一个算法题，使用动态规划进行求解，感兴趣的同学可以查阅相关资料。

### 总结

无论是 React 的 Diff 算法，还是 Vue2(snabbdom) 的 Diff 算法，抑或是 Vue3 的核心 Diff 算法，其重点无非就是：判断是否有节点需要移动，以及应该如何移动和寻找出那些需要被添加或移除的节点。

核心 Diff 算法的目的是避免创建和移除 DOM 的开销，尽可能的复用节点，只是进行 patch，并移动。而复用的关键是新旧 children 的节点保持映射关系，通常开发者通过指定 key prop 来暗示某些节点在不同的渲染下保持稳定。

核心 Diff 算法最好的优化手段就是避免它们的执行。