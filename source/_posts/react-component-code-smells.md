---
title: "H5&小程序开发-React组件流行代码规范"
date: "2022-05-07"
cover: "https://m.360buyimg.com/img/jfs/t1/76636/29/17681/399340/6275d8e5E5aba2d55/576d8a466b212acd.jpg"
---

## React component code smells

### 一、多个 porps 传递到单个组件

表明该组件应该拆分；遇到或者想在这个列表再添加一个组件时

#### 1、该组件能做多少事？

要确认是否可以将组件拆分为多个较小的组件

#### 2、可否合并？

compose components 组成组件，而不只处理一个组件内的所有逻辑

```js
<ApplicationForm
  user={userData}
  organization={organizationData}
  categories={categoriesData}
  locations={locationsData}
  onSubmit={handleSubmit}
  onCancel={handleCancel}
/>
```

处理后：

```js
<ApplicationForm onSubmit={handleSubmit} onCancel={handleCancel}>
  <ApplicationUserForm user={userData} />
  <ApplicationOrganizationForm organization={organizationData} />
  <ApplicationCategoryForm categories={categoriesData} />
  <ApplicationLocationsForm locations={locationsData} />
</ApplicationForm>
```

#### 3、是否传递了很多配置？

最好是组合成一个对象，除数据外

```js
<Grid
  data={gridData}
  pagination={false}
  autoSize={true}
  enableSort={true}
  sortOrder="desc"
  disableSelection={true}
  infiniteScroll={true}
/>
```

处理后：

```js
const options = {
  pagination: false,
  autoSize: true,
  enableSort: true,
  sortOrder: 'desc',
  disableSelection: true,
  infiniteScroll: true,
  ...
}
<Grid
  data={gridData}
  options={options}
/>
```

### 二、不兼容的 props

避免传递彼此不兼容的的 props， 例：先创建通用 input 组件来处理文本，但还增加处理电话号码：

```js
function Input({ value, isPhoneNumberInput, autoCapitalize }) {
  if (autoCapitalize) capitalize(value);

  return <input value={value} type={isPhoneNumberInput ? "tel" : "text"} />;
}
```

后两个不能同时使用，电话号码无法大写，所以需要将组件分解成多个小组件，让一些逻辑可以它们之间共享

```js
function TextInput({ value, autoCapitalize }) {
  if (autoCapitalize) capitalize(value);
  useSharedInputLogic();

  return <input value={value} type="text" />;
}

function PhoneNumberInput({ value }) {
  useSharedInputLogic();

  return <input value={value} type="tel" />;
}
```

### 三、复制 props 到 state

不要通过将 props 复制到 state 来 停止数据流

```js
function Button({ text }) {
  const [buttonText] = useState(text);

  return <button>{buttonText}</button>;
}
```

通过将 text prop 作为 useState 的初始值传递，实际上忽略了 text 的所有更新值。 若 text prop 更新，组件将 render 其第一个值
下面更实际的例子，当我们从 prop 中获取一些新值，运行 slowFormatText 函数格式化文本属性，但这个缓慢的计算需要大量时间执行

```js
function Button({ text }) {
  const [formattedText] = useState(() => slowlyFormatText(text));

  return <button>{formattedText}</button>;
}
```

下面改进：让其处于 state，解决不必要重新运行，但同上都停止了组件更新。更好的方法是使用 useMemo hook 来记住结果，slowlyFormatText 仅在 text 更改时才运行且没有停止组件更新

```js
function Button({ text }) {
  const formattedText = useMemo(() => slowlyFormatText(text), [text]);

  return <button>{formattedText}</button>;
}
```

注：有时我们确实需要一个 prop，在它身上所有更新都被忽略。例如颜色选择器，需要该选项设置初始选择颜色，但当用户选择一种颜色时，我们不希望更新覆盖用户的选择，这种情况可以将 prop 复制到 state，但为了引导用户，大多数人都会在 prop 前面加上初始默认值
[弹性组件：](https://overreacted.io/writing-resilient-components/)

### 四、从函数返回 JSX

不要从组件内部的函数返回 JSX

```js
function Component() {
  const topSection = () => {
    return (
      <header>
        <h1>Component header</h1>
      </header>
    );
  };

  const middleSection = () => {
    return (
      <main>
        <p>Some text</p>
      </main>
    );
  };

  const bottomSection = () => {
    return (
      <footer>
        <p>Some footer text</p>
      </footer>
    );
  };

  return (
    <div>
      {topSection()}
      {middleSection()}
      {bottomSection()}
    </div>
  );
}
```

乍一看还可以，但代码难推理，应避免使用。可以内联 JSX
注：因为创建了新组件，所以不必将其移动到新文件中。有时如果多个组件紧密耦合，则将它们保留在同一个文件中时有意义的。

### 五、state 中的多个 booleans

要避免使用多个布尔值表示组件的状态， 尤其时编写组件的后续拓展组件功能时，容易出现多个布尔值来表示组件状态。
例：单击按钮发出 web 请求的小型组件

```js
function Component() {
  const [isLoading, setIsLoading] = useState(false);
  const [isFinished, setIsFinished] = useState(false);
  const [hasError, setHasError] = useState(false);

  const fetchSomething = () => {
    setIsLoading(true);

    fetch(url)
      .then(() => {
        setIsLoading(false);
        setIsFinished(true);
      })
      .catch(() => {
        setHasError(true);
      });
  };

  if (isLoading) return <Loader />;
  if (hasError) return <Error />;
  if (isFinished) return <Success />;

  return <button onClick={fetchSomething} />;
}
```

单击按钮时，将 isLoading 设置为 true 并用 fetch 发起 web 请求，请求成功时，则将 isLoading 设置为 false 并将 isFinished 设置为 true，否则将 hasError 设置为 true；
能用，但是难推断组件状态，易出错。可能最终出现不可能的状态，比如意外将 isLoading 和 isFinished 同时设置为 true；
枚举，最好能管理状态的方法。其他语言中，它是一种定义变量的方法，该变量只能设置为预定义的常量值集合，技术上来说，枚举在 js 中不存在，可使用字符串作为枚举

```js
function Component() {
  const [state, setState] = useState("idle");

  const fetchSomething = () => {
    setState("loading");

    fetch(url)
      .then(() => {
        setState("finished");
      })
      .catch(() => {
        setState("error");
      });
  };

  if (state === "loading") return <Loader />;
  if (state === "error") return <Error />;
  if (state === "finished") return <Success />;

  return <button onClick={fetchSomething} />;
}
```

消除了不可能状态的可能性，并且更具有推理性；若并入 TS 的类型系统：

```js
const [state, setState] =
  (useState < "idle") | "loading" | "error" | ("finished" > "idle");
```

### 六、useState 太多

避免在同一组件使用太多 useState hooks
有太多 useState hooks 的组件内部功能太多，需要分解成多个组件，但是在某些复杂情况下，需要单个组件中管理某些复杂状态
例：自动完成输入组件中的某些状态和几个功能

```js
function AutocompleteInput() {
  const [isOpen, setIsOpen] = useState(false)
  const [inputValue, setInputValue] = useState('')
  const [items, setItems] = useState([])
  const [selectedItem, setSelectedItem] = useState(null)
  const [activeIndex, setActiveIndex] = useState(-1)

  const reset = () => {
    setIsOpen(false)
    setInputValue('')
    setItems([])
    setSelectedItem(null)
    setActiveIndex(-1)
  }

  const selectItem = (item) => {
    setIsOpen(false)
    setInputValue(item.name)
    setSelectedItem(item)
  }

  ...
}
```

一个重置功能函数重置所有状态，一个 selectItem 函数来更新某些状态，这些函数必须使用 useState 中的许多状态设置器来完成预期任务。以后我们可能还需要操作更多，维护会越来越困难。
useReducer hook 来代替管理状态：

```js
const initialState = {
  isOpen: false,
  inputValue: "",
  items: [],
  selectedItem: null,
  activeIndex: -1
}
function reducer(state, action) {
  switch (action.type) {
    case "reset":
      return {
        ...initialState
      }
    case "selectItem":
      return {
        ...state,
        isOpen: false,
        inputValue: action.payload.name,
        selectedItem: action.payload
      }
    default:
      throw Error()
  }
}

function AutocompleteInput() {
  const [state, dispatch] = useReducer(reducer, initialState)

  const reset = () => {
    dispatch({ type: 'reset' })
  }

  const selectItem = (item) => {
    dispatch({ type: 'selectItem', payload: item })
  }

  ...
}
```

使用简化器，封装管理状态的逻辑，将复杂性移出组件。现在可以分别考虑状态和组件，便于理解。
注：useState 和 useReducer 都具有各自的优缺点和不同的用例。减速器推荐：[状态减速器模式](https://kentcdodds.com/blog/the-state-reducer-pattern-with-react-hooks)

### 七、大量 useEffect

避免使用大量 useEffect 产生多种影响，易出错，难推理。 当释放钩子的时候，错误做法就是将太多的东西放到一个 useEffect 中。

```js
function Post({ id, unlisted }) {
  ...
  useEffect(() => {
    fetch(`/posts/${id}`).then(' do something ')

    setVisibility(unlisted)
  }, [id, unlisted])
  ...
}
```

虽然效果不太大，但它仍可做很多事，当私有 prop 变化时，即使 id 不变，我们也可 fetch post
为了捕捉此类错误，通过 依赖项更改时来执行此操作来描述效果。当 id 或未更改，fetch post 更新可见
若包含 or 或 and，则表示存在问题，下面拆分成两个 useEffect

```js
function Post({ id, unlisted }) {
  ...

  useEffect(() => { // when id changes fetch the post
    fetch(`/posts/${id}`).then('...')
  }, [id])

  useEffect(() => { // when unlisted changes update visibility
    setVisibility(unlisted)
  }, [unlisted])

  ...
}
```

可以降低组件复杂性，易推理，降低创建错误的风险

### 相关链接

[https://antongunnarsson.com/react-component-code-smells/#incompatible-props ](https://antongunnarsson.com/react-component-code-smells/#incompatible-props)
