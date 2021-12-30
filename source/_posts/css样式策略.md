---
title: react中的样式策略
date: 2021-12-28
author: 7
---
React中的样式策略主要有以下几种：

- **内联样式：** 内联样式就是在JSX元素中，直接定义行内的样式；
- **CSS样式表：** 这也是我们最常用的样式策略，使用单独的样式表，使用CSS或者SCSS等来为元素设置样式；
- **CSS模块：** CSS模块是一个文件，默认情况下所有类名和动画名都在本地范围；
- **styled-components**：这是一个用于React和React Native的样式组件库，它允许我们早应用中使用组件级样式，这些样式就是使用CSS-in-JS的技术来编写的；
- **JSS**：JSS是一个CSS创作工具，它允许我们使用JavaScript以声明式、无冲突和可重复的方式来描述样式。

这么说可能有点抽象，下面就来看看这些样式策略分别是怎么使用的，以及它们的优缺点。

### 1. 内联样式

React中的组件是由JSX元素组件的，虽然编写的不是常规的HTML，但是仍然是可以使用内联样式去定义样式的。与普通的HTML内联样式唯一的区别就是，JSX中的内联样式是一个对象，而不是一个字符串。 

下面来看一个简单的例子：

```
import React from "react";

export default function App() {
  return (
      <h1 style={{ color: "red" }}>Hello World</h1>
  );
}
复制代码
```

这个组件很简单，上面的style属性中，外面的大括号是用来告诉JSX解析器，括号中的内容是JavaScript，而不是一个字符串。而里面的大括号用来初始化一个对象。 

当包含的样式属性中出现多个词时，应该使用驼峰命名，比如text-align属性必须写成textAlign：

```
import React from "react";

export default function App() {
  return (
      <h1 style={{ textAlign: "center" }}>Hello World</h1>
  );
}
复制代码
```

由于style属性是一个对象，所以我们也可以将这个对象定义成一个常量来分隔样式，这样就可以根据需要在其他组件上重用它：

```
import React from "react";

const h1Style = {
  color: 'red',
  textAlign: "center"
}

export default function App() {
  return (
      <h1 style={h1Style}>Hello World</h1>
  );
}
复制代码
```

如果想要在重用时继续扩展这个样式对象，就可以使用ES6中的扩展运算符来实现：

```
import React from "react";

const h1Style = {
  color: 'red',
  textAlign: "center"
}

export default function App() {
  return (
      <h1 style={{...h1Style, fontSize: '25px'}}>Hello World</h1>
  );
}
复制代码
```

除此之外，我们还可以在样式对象中使用变量，这样就能实现样式的动态变化：

```
import React from "react";

export default function App({themeColor}) {
  const h1Style = {
    color: themeColor,
    textAlign: "center"
  }

  return (
      <h1 style={h1Style}>Hello World</h1>
  );
}
复制代码
```

这也是内联样式最重要的特性之一。但是，React团队并不推荐使用内联样式。内联样式也是CSS-in-JS技术的最基本的示例。 

内联样式的优点：

- **使用简单：** 使用内联样式的好处就是简单的以组件为中心来实现样式的添加；
- **扩展方便：** 通过使用对象进行样式设置，可以方便的扩展对象来扩展样式；
- **避免冲突：** 样式通过对象的形式定义在组件中，避免了和其他样式的冲突。



在大型项目中，内联样式可能并不是一个很好的选择，因为内联样式还是有局限性的：

- **不能使用伪类：** 这意味着 :hover、:focus、:actived、:visited等都将无法使用；
- **不能使用媒体查询：** 媒体查询相关的属性不能使用。
- **减低代码可读性：** 如果使用很多的样式，代码的可读性将大大降低。
- **没有代码提示**：当使用对象来定义样式时，是没有代码提示的，所以如果拼错样式属性，也很难检查出来。

### 2. CSS样式表

CSS样式表应该是我们最常用的使用定义样式的方式。除了原生的CSS，现在最长使用的还有SASS预处理器来定义样式。这些样式表可以根据需要应用样式的位置来导入到React组件中。 

下面来看一个例子：

```
// style.scss
.box {
  margin: 40px;
  border: 1px solid black;
}

.box-content {
  font-size: 16px;
  text-align: center;
}
复制代码
```

为了在Box组件中使用这个样式，需要将SASS文件导入：

```
import React from 'react';
import './style.scss';

const Box = () => (
  <div className="box">
    <p className="box-content"> Hello World </p>
  </div>
);

export default Box;
复制代码
```

在普通的HTML中，我们会使用class来定义类，而JSX中会使用className来定义，因为class是JavaScript中的一个保留字。 

在使用CSS样式表这种样式策略时，我们还可以使用现有的框架，比如Bootstrap等，这些框架提供了现有的类和组件，可以将它们直接插入到React组件中，而不需要为每个元素设置样式。 

CSS样式表的优点：

- **关注分离：** 实现了样式和JavaScript的分离，像往常一样编写CSS语法；
- **使用CSS所有功能：** 此方法允许我们使用CSS的任何语法，包括伪类、媒体查询等；
- **缓存和性能：** 标准的CSS文件有利于浏览器优化，在本地缓存文件从而允许重复的方法，以提高性能；
- **易编写**：CSS样式表在书写时会有代码提示；

当然，CSS样式表也是有缺点的：

- **产生冲突：** CSS选择器都具有相同的全局作用域，如果选择器使用错误，有可能造成样式的冲突；
- **可读性：** 如果结构不正确，随着应用程序变得复杂，CSS或者SASS样式表就的很长，并且难以阅读；
- **难以整理：** 随着样式表变得越来越复杂，一些旧的或者未使用的样式属性就很难清理；
- **没有真正的动态样式：** 在CSS表中难以实现动态设置样式。

### 3. CSS模块

一个CSS模块中所有的类名和动画名默认都是局部范围的CSS文件。在使用CSS模块时，每个React组件都有自己的CSS样式文件，它的使用单位仅限于对应的组件文件。这样，在构建时，就不会产生类名的冲突。

下面来安CSS模块是如何在React中使用的：

```
//style.css
 :local(.container) {
   margin: 40px;
   border: 2px solid red;
 }
 :local(.content) {
   font-size: 15px;
   text-align: center;
 }
复制代码
```

想要使用:local(.className)，就需要在webpack中进行配置才可以使用的。我们可以在webpack中添加对应的加载器：

```
test: /.css$/,
loader: 'style!css-loader?modules&importLoaders=1&localIdentName=[name]__[local]___[hash:base64:5]' 
复制代码
```

为了在Box组件中使用这个CSS模块，就需要将模块文件导入到组件中进行使用：

```
import React from 'react';
import styles from './style.css';

const Box = () => (
  <div className={styles.container}>
    <p className={styles.content}> Styling React Components </p>
  </div>
);

export default Box;
复制代码
```

styles是一个包含在style.css中的样式对象，该对象包含定义的所有类。这就是CSS模块的基本使用方式。 

我们还可以使用CSS模块的方式来继承样式，使用composes关键字可以实现样式的继承：

```
:local(.MediumParagraph) {
  font-size: 20px;
}

:local(.BlueParagraph) {
  composes: MediumParagraph;
  color: blue;
  text-align: left;
}

:local(.GreenParagraph) {
  composes: MediumParagraph;
  color: green;
  text-align: right;
}
复制代码
```

在CSS模块中，默认会导出本地样式（`:local(...)`），也可以使用（`:global(...)`）来导出全局样式。本地样式被会编译成唯一标识的类名，而全局样式编译后还是原来的名称。 

使用CSS模块的优点：

- **模块化：** 实现样式的模块化，避免样式冲突；
- **SSR时无重复：** 在使用服务端渲染（SSR）时没有代码的重复；

使用CSS模块的缺点：

- **额外的构建工具：** 需要额外的构建工具（webpack）；
- **书写麻烦：** 本地模块和全局模块书写起来很麻烦；
- **驼峰命名：** 只允许使用驼峰命名。

### 4. styled-components

Styled Components 是一个使用CSS-in-JS技术实现的样式组件库，它是一个为React和React Native设计的库。它允许我们在应用中使用JavaScript和CSS混合起来编写样式级组件。并且它是支持Sass的，不需要添加任何库。

根据HTTP Archive 的调查显示，在所有CSS-in-JS方案中，Styled Components的使用占到了一半以上： ![imagepng](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d735a05f8eda43c29c8b9a99455d630e~tplv-k3u1fbpfcp-watermark.awebp) 

它是使用与CSS 模块的操作方法相同，这是一种编写CSS的方式，该CSS的范围仅限于单个组件，页面中的任何其他元素甚至组件都是无法访问它的。

#### （1）基本使用

那该如何使用Styled Components库呢？来看看使用步骤： （1）安装库

```
npm install styled-components --save
复制代码
```

（2）引入库

```
import styled from "styled-components";
复制代码
```

（3）定义样式组件

```
import React from "react";
import styled from "styled-components";

const Title = styled.h1`
  font-size: 20px;
  text-align: center;
  color: red;
`;

export default function App() {
  return <Title>Hello World</Title>;
}
复制代码
```

这里就定义了一个样式组件Title，可以看到，它的实现还是非常简单的，语法就是：styled.标签名，后面跟一个模板字符串，其内容就是组件的样式属性。这样就创建了一个带样式的React组件。

#### （2）样式嵌套

我们还可以在样式组件中嵌套样式，比如：

```
import React from "react";
import styled from "styled-components";

const Title = styled.div`
  text-align: center;

    img {
        height: 100px;
        width: 200px;
    }
    p {
        font-size: 20px;
    color: red;
    }
`;

export default function App({imgUrl}) {
  return (
    <Title>
        <p>Hello World</p>
        <img src={imgURl} alt="" />
    </Title>
  );
}
复制代码
```

#### （3）组件传参

我们还可以给样式组件传递参数：

```
import React from "react";
import styled from "styled-components";

const Title = styled.div`
  text-align: center;

    img {
        height: 100px;
        width: 200px;
    }

    p {
        font-size: 20px;
    color: ${props => props.color};
    }
`;

export default function App({imgUrl, themeColor}) {
  return (
    <Title>
        <p color={themeColor}>Hello World</p>
        <img src={imgURl} alt="" />
    </Title>
  );
}
复制代码
```

这样就实现了动态改变文字的颜色。

#### （4）样式组件的class

从上面组件中可以看出，我们定义的样式组件时没有class类名的，实际上类名的定义是style-components库帮我们完成的，来看一个实际的例子：

```
const SeniorTitle = styled.h1`
    width: 100%;
    height: 29pt;
    line-height: 29pt;
    background-color: #E0F2E1;
    border-radius: 10pt;

    span {
        text-align: center;
    }

    p {
        font-size: 14pt;
        color: ${props => props.color};
    }
`;
复制代码
```

上面是定义的一个h1组件，最终编译后，类名是css-1d3spwa，其中 span 的样式编译后如下：

```
.css-1d3spwa span {
    text-align: center;
}
复制代码
```

![imagepng](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7c58d6c9b7b46eca43b4774b8d9814d~tplv-k3u1fbpfcp-watermark.awebp)

#### （5）样式组件继承

我们还可以使用其他组件的样式，并继承它们所有的样式（如果有相同的样式，则会覆盖继承来的样式），来看例子：

```
const BoringButton = styled.button`
  color: blue;
    background-color: green；
`;

const CoolButton = styled(BoringButton)`
  color: pink;
`;
复制代码
```

这样，下面的组件就获得了上面组件的样式，并覆盖了重复了样式color。

#### （6）使用css辅助函数

如果我们需要在多个样式组件中使用通用的样式，css辅助函数就派上用场了，来看看css辅助函数是如何使用的：

```
import React from "react";
import styled, {css} from "styled-components";

const commonStyle = css`
  color: white;
    font-size: 20px;
`;

const Button = styled.button`
  ${commonStyle};
  background-color: red;
`;

const AnotherButton = styled.button`
  ${commonStyle};
  background-color: green;
`;
复制代码
```

这里定义的commonStyle在两个button组件中使用时，不会有自己单独的类，而是会添加到这两个button组件对应的类中。

#### （7）引用其他样式组件

我们还可以引用其他的样式组件，和继承时是类似的：

```
const BoringButton = styled.button`
  color: blue;
    background-color: green；
`;

const CoolButton = styled.div`
    ${BoringButton} {
        color: pink;
    }
`;
复制代码
```

#### （8）结合TypeScript使用样式组件

如果我们项目使用React + TypeScript来编写，那么当给样式组件传递props时，就可能会报错： ![imagepng](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94403ef994b84329883898b99cedc128~tplv-k3u1fbpfcp-watermark.awebp) 这时我们可以通过泛型的形式给上述组件定义一个props类型：

```
interface ITitleWithoutNumber {
    paddingRight: string;
}


const TitleWithoutNumber = styled.h1<ITitleWithoutNumber>`

`
复制代码
```

这样就限制了组件props的类型，如果想添加一个新的props值，就需要先在interface接口中增加新的参数及类型。

#### （9）全局主题

在style-components中我们还可以使用全局的主题，它通过ThemeProvider API来实现：

```
import React from "react";
import { ThemeProvider } from "styled-components";

const theme = {
  colors: {
    powderWhite: "#FFFDF9",
    persianGreen: "#06B49A",
    lightBlue: "#AFDBD2",
    onyx: "#36313D"
  },
  fonts: ["sans-serif", "Roboto"],
  fontSizes: {
    small: "1em",
    medium: "2em",
    large: "3em"
  }
};

const Theme = ({ children }) => (
  <ThemeProvider theme={theme}>{children}</ThemeProvider>
);

export default Theme;
复制代码
```

这里导出了一个Theme组件，它提供了一个theme主题，供子组件使用，可以这样来使用：

```
import React from "react";
import Theme from "./Theme";
import styled from "styled-components";

const Container = styled.div`
  width: 100%;
  border: ${props => `1px solid ${props.theme.colors.onyx}`};
  background-color: ${props => props.theme.colors.lightBlue};
  font-family: ${props => props.theme.fonts[0]};
`;

const Heading = styled.h1`
  font-size: ${({ isHeading, theme: { fontSizes } }) =>
    isHeading ? fontSizes.large : fontSizes.small};
  color: ${({ theme: { colors } }) => colors.persianGreen};
`;

const App = () => {
  return (
    <Theme>
      <Container>
        <Heading isHeading={true}>Hello World</Heading>
        <h2>styled-components</h2>
      </Container>
    </Theme>
  );
};
export default App;
复制代码
```

可以使用这种方式来实现全局的主题。

#### （10）优点和缺点

style-components的优点：

- **开箱即用的Sass语法**：在style-components支持开箱即用的Sass语法，而不需要进行额外的安装或设置；
- **支持使用主题：** 在styled-components 提供了一个ThemeContext，可以直接传递主题对象的方法，方便使用全局主题；
- **动态样式：** 可以使用props来动态的设置和改变样式；
- **没有类型冲突：** 样式组件会我们生成唯一的类名，不会与其他组件的类型产生冲突；
- **便于维护：** 我们不需要在各种样式文件中查看样式，只需要在样式组件中查看其样式即可，便于维护；

style-components的缺点：

- **影响性能：** style-components在构建时会将React 组件中所有的样式定义转化为纯CSS，并将内容注入到index.html文件的`<style>`标签中。这样不仅增加了HTML文件的大小，并且无法对输出的CSS进行分块，影响了应用的性能；

### 5. JSS

JSS是一个CSS创作工具，它允许我们使用JavaScript以生命式、无冲突和可重用的方式来描述样式。它可以在浏览器、服务器端或构建时在 Node.js 中编译。JSS 是一种新的样式策略，它与框架无关，由多个包组成：核心、插件、框架集成等。 

在React中，可以使用React-JSS。React-JSS 是一个框架集成，可以在 React 应用程序中使用 JSS。它是一个单独的包，所以不需要安装 JSS 核心，只需要 React-JSS 包即可。React-JSS 使用新的 Hooks API 将 JSS 与 React 结合使用。 

下面来将JSS和内联样式进行对比。 

**内联样式：**

```
import React from 'react'

const Button = () => {
  const buttonGreen = {
    backgroundColor: "green",
    border: "1px solid white",
    borderRadius: "2px"
  };

  return(
    <button style={buttonGreen}>
      green
    </button>
  )
}
复制代码
```

**React-JSS：**

```
import React from 'react'
import {createUseStyles} from 'react-jss'

const useStyles = createUseStyles({
  buttonGreen: {
    backgroundColor: "green",
    border: "1px solid white",
    borderRadius: "2px"
  }
})

const Button = () => {
  const {buttonGreen} = useStyles()

  return(
    <button className={buttonGreen}>
      green
    </button>
  )
}
复制代码
```

可以看到，相对于普通的内联样式，React-JSS的特征如下：

- 导入了一个createUseStyles方法；
- 通过createUseStyles方法方法创建一个useStyles hook；
- 通过参数对象的方式将样式传递给createUseStyles方法；
- 通过解析useStyles hook的返回值获取到组件样式buttonGreen；
- 将buttonGreen传递给组件的className进行解析。

那如果想要传递props，两者又会有什么不同的呢？接着来看： 

**内联组件：**

```
import React from 'react'

const Button = ({backgroundColour, children}) => {
  const buttonStyles = {
    backgroundColor: backgroundColour,
    border: "1px solid white",
    borderRadius: "2px"
  };

  return(
    <button style={buttonStyles}>
      {children}
    </button>
  )
}
复制代码
```

**React-JSS：**

```
import React from 'react'
import {createUseStyles} from 'react-jss'

const useStyles = createUseStyles({
    buttonStyles: {
      backgroundColor: backgroundColour => backgroundColour,
      border: "1px solid white",
      borderRadius: "2px"
    }
})

const Button = ({backgroundColour, children}) => {
  const {buttonStyles} = useStyles(backgroundColour)

  return(
    <button className={buttonStyles}>
      {children}
    </button>
  )
}
复制代码
```

可以通过以下方式来调用上面的组件：

```
import React from 'react';
import Button from 'Button';

const SomePage = () => (
  <Button backgroundColour="blue">blue</Button>
)
复制代码
```

这里的区别也是显而易见的，需要将传递的props的值传给useStyles方法，这样就可以在createUseStyles中使用了。这就是React-JSS的基本使用了。下面来看看它的优缺点 

React-JSS的优点：

- **可重用性：** 组件是可重用的，所以一次编写即可在任何地方使用它们；
- **动态样式：** 可以使用props动态设置样式；
- **局部范围：** JSS支持局部样式。

React-JSS的缺点：

- **额外的层：** 使用React-JSS库会使得React程序多一个额外的层，这个有时是不必要的；
- **代码可读性：** 通过这种方式也会自动生成类名，它们都是唯一的，可阅读性较差，尤其是在浏览器进行调试时，难以确定这个样式是在哪里定义的。

上面介绍了五种常用的React样式策略，这五种策略并没有绝对的好或坏，都各有优缺点，可以根据实际的业务场景去使用。
