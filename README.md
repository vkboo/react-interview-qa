# React 328道最全面试题 & 个人作答

> [React 328道最全面试题](https://juejin.cn/post/6844903892853981198)

### React
#### 1. 什么时候使用状态管理器？
1. 从组件上考虑：状态需要共享到多个组件，任何组件都可以拿到并响应共享状态中的数据，任何组件也都可以修改共享状态中的数据；
2. 针对第一点，`Context API`、全局变量加发布订阅的模式都可以解决，但是上面提到的方式又一个缺点，当子组件可以任意调用，以此来改变全局状态时，需要一种机制去限制，让修改数据、获取数据通过统一的状态管理器去做，这样如果出现问题，可以降低调试成本；
3. 高级功能：时间旅行等。
#### 2. render函数中的return如果没有使用`()`会有什么问题？
js是可不加分号的语言，所以在没有`()`的情况下，return后直接换行，会认为return了undefined，加上了括号，就不存在这种问题；不加的话，切记不能换行
#### 3. componentWillUpdate可以直接修改state的值吗？
不能。`componentWillUpdate`发生在`state`或`props`改变的情况下，在`shouldComponentUpdate`返回`true`且在`render`前执行，如果在`componentWillUpdate`中进行了`setState`或者`Redux`中的`dispatch(action)`操作，会导致`Maximum update depth exceeded`错误。
#### **4. 说说你对React的渲染原理的理解
React的数据源是两个部分：`state`和`props`；`state`的改变且当前组件的`JSX`引用了这个state，会引起当前组件render函数的渲染，props的改变，不管子组件的JSX中有没有挂载这个prop，都会触发子组件的render函数执行；render函数的执行，不代表会进行DOM的渲染，React内部的Fiber算法，会对新旧的虚拟DOM进行比较，如果不一致，才会进行新DOM的渲染。
#### 5. 什么是渲染劫持
一般通过hoc函数实现，重新目标组件的props state render函数，实现不限于组件渲染、条件渲染等功能；以下实现一个最简单的渲染劫持demo
```jsx
import React from 'react';
export default class Child extends React.Component {
    render() {
        const { counter } = this.props;
        return (
            <div>CHILD: {counter}</div>
        )
    }
}

// 这个hoc劫持了counter这个prop，无论外层传进来是多少，到Child组件时都会乘以2
function hoc(WrapComponent) {
    return class extends React.Component {
        render() {
            const { counter, ...rest } = this.props;
            return <WrapComponent {...rest} counter={counter * 2} />
        }
    }
}


const HocChild = hoc(Child);
export default class CodeTest extends React.Component {
    render() {
        return (
            <HocChild
                counter={this.state.counter}
            />
        )
    }
}
```
#### 6. React Intl是什么原理？
本质上就是Context API实现的，外层Context.Provider，内部再引用Provider提供的值进行渲染。
#### 7. 你有使用过React Intl吗？
使用过。基本demo如下：
```jsx
// i18n/en.js
const en = { 'app.learn': 'Learn {name}' };
export default en;

// ###########################

// i18n/ja.js
const ja_JP = { 'app.learn': '学び {name}' };
export default ja_JP;

// ###########################

// i18n/zh.js
const zh_TW = { 'app.learn': '學習 {name}' };
export default zh_TW;

// ###########################
import React, { useState } from 'react'
import { IntlProvider, FormattedMessage } from 'react-intl'
import en from '../../i18n/en';
import zh from '../../i18n/zh';
import ja from '../../i18n/ja';

export default function App() {
    const [locale, setLocale] = useState(navigator.language);
    let messages;

    if (locale.includes('zh')) {
        messages = zh;
    } else if (locale.includes('ja')) {
        messages = ja;
    } else {
        messages = en;
    }
    return (
        <IntlProvider
            locale={locale}
            key={locale}
            defaultLocale="en"
            messages={messages}
        >
            <IntlComponent setLocale={setLocale} />
        </IntlProvider>
    )
}

function IntlComponent({ setLocale }) {
    return (
        <div className="App">
            <header className="App-header">
                <div>
                    <button onClick={() => setLocale('en')}>英文</button>
                    <button onClick={() => setLocale('zh-Hant')}>中文</button>
                    <button onClick={() => setLocale('ja')}>日文</button>
                </div>
                <a
                    className="App-link"
                    href="https://reactjs.org"
                    target="_blank"
                    rel="noopener noreferrer"
                >
                    <FormattedMessage
                        id="app.learn"
                        values={{ name: 'react intl' }}
                    />
                </a>
            </header>
        </div>
    );
}

```
#### 8. 怎么实现React组件的国际化呢？
借助第三方库，如react-intl。定一个页面中需要用到的字符文件，使用的时候使用FormattedMessage进行填充即可。(代码如上题)
#### 9. 说说Context有哪些属性？
- 创建：
```javascript
const MyContext = React.createContext(initialValue)
```
- 注入: 
```jsx
<MyContext.Provider value={usedValue}></MyContext.Provider>
```
- 调试：
```JavaScript
Mycontext.displayName = 'MyContextComponent';
```
- 使用: 
   - 使用Consumer
```jsx
<MyContext.Consumer>
  {value => {/** JSX */}}
</MyContext.Consumer>
```
   - 使用`ContextType`(class组件)
```jsx
// 前置代码
import React from 'react';
import ThemedButton from './themed-button'

const themes = {
    dark: {
        background: '#000'
    },
    light: {
        background: '#fff'
    }
}
const ThemeContext = React.createContext(themes['dark']);

export default class ContextDemo extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            theme: themes.light,
        }
        this.onClickButton = this.onClickButton.bind(this);
    }
    onClickButton(e) {
        this.setState(state => ({
            theme: state.theme === themes.dark
                ? themes.light
                : themes.dark,
        }))
    }
    render() {
        return (
            <ThemeContext.Provider value={this.state.theme}>
                <ThemedButton onClick={this.onClickButton} />
            </ThemeContext.Provider>
        )
    }
}
// ###### themed-button.js
import React, { Component } from 'react';
import { ThemeContext } from './theme-context';
export default class ThemedButton extends Component {
    static contextType = ThemeContext;
    render() {
        return (
          {/** onClick这个prop，在父组件的函数中改变<ThemeContext.Provider /> 的value值 */}
            <button onClick={this.props.onClick}>
                { this.context.background }
            </button>
        )
    }
}
```

   - 使用`hooks`
```jsx
import React, { useContext } from 'react';
import { ThemeContext } from './theme-context';

function ThemedButton({ onClick }) {
    const theme = useContext(ThemeContext);
    return (
        <button onClick={onClick}>
            {theme.background}
        </button>
    )
}

export default ThemedButton;
```
#### 10. 怎么使用Context开发组件？
（同上题，考察React Context API的基本用法）
#### 11. 为什么React并不推荐我们优先考虑使用Context?
1. 题目比较老了，在初始推出时，Context API始终是作为一个beta性质而存在，所以不推荐使用
2. React中，最简单、影响渲染范围最小的是state & props，如果能用这两者解决的问题，没必要使用Context
3. Context如果涉及渲染的组件较多，那么很可能产生对渲染性能的问题，而社区的库，如`React-Redux``Mobx`等，则会在库内部去做性能优化
4. 简单的使用Context的话，不利于数据的追踪，而`Redux`等第三方库在这方面做的更好
#### 12. 除了实例的属性可以获取Context外哪些地方还能直接获取Context呢?
还有的方式有(上面的代码都有演示，这里就不再贴代码了)：
1. Context.Consumer组件
2. hooks: useContext
#### 13. childContextTypes是什么？它有什么用？
（过时的api，无需关注）
#### 14. contextType是什么？它有什么用？
`contextType`是React class组件的静态属性，将Context赋给这个静态属性后，再用组件的实例`.context`即可获取相应Context的值（上面有代码展示）。
#### 15. Consumer向上找不到Provider的时候怎么办？
找不到就取`React.createContext(initialValue)`中的默认值`initialValue`
#### 16. 有使用过Consumer吗？
用过。上面有代码。具体再补充一点，如果组件中需要使用多个Context的使用，需要使用多个`<Context.Consumer>`组件的嵌套，此种情况下，`useContext`肯定是更好的选择。
#### 17. 在React怎么使用Context？
（同[第9题](#9-说说context有哪些属性)）
#### 18. React15和16分别支持IE几以上？
（古老的问题，可忽略）
React15：>IE8
React16: >=IE11
#### 19. 说说你对windowing的了解
在涉及到超长列表的场景时，如果不进行分页，一次性塞入大量的DOM，会对性能造型影响。这时候，就要用上windowing即“虚拟列表”技术，它的原理是随着用户的滚动，只渲染部分（一般是用户可见的那一部分）DOM，以此来减少性能消耗。在React中，可以使用`react-window`` `库进行实现。
#### 20. 举例说明React的插槽有哪些运用场景？
一个 portal 的典型用例是当父组件有 overflow: hidden 或 z-index 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框.
#### 21. 你有用过React的插槽（Portals）吗？怎么用？
用过。核心的api是`ReactDOM.createPortal(reactNode, container)`,第一个参数是可渲染的react元素，第二个参数是DOM节点，返回一个新的react元素，在render函数的return中调用方法后，真实的DOM回渲染到指定的DOM Container下，但是从React组件树来看，他是没有变化的示例代码如下：

注意: `ReactDOM.createPortal`看起来是往指定的DOM塞入react元素，但是其返回值必须在render函数或者函数式组件中真正的`return`了之后，才会渲染，这一点没有脱离React渲染组件的本质。
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
export default class PortalsDemo extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            visible: false,
        };
    }

    handleToggle = () => {
        this.setState(prevState => ({
            visible: !prevState.visible,
        }));
    }

    render() {
        const { visible } = this.state;
        return (
            <div>
                <button onClick={this.handleToggle}>Toggle</button>
                <Modal visible={visible}>
                    <Child />
                </Modal>
            </div>
        );
    }
}



class Modal extends React.Component {
    constructor(props) {
        super(props);
        this.el = document.createElement('div');
    }

    componentDidMount() {
        document.body.appendChild(this.el);
    }


    componentWillUnmount() {
        document.body.removeChild(this.el);
    }

    render() {
        const { visible } = this.props;
        return visible
            ? ReactDOM.createPortal(
                this.props.children,
                this.el
            )
            : null;
    }
}


function Child() {
    // 这个按钮的点击事件会冒泡到父元素
    // 因为这里没有定义 'onClick' 属性
    return (
        <div className="modal">
            弹框内容
        </div>
    );
}

```
#### 22. React的严格模式有什么好处？
通过`<React.strictMode>`内置组件包装，即开启了React的严格模式，内部组件的代码需遵循一定的规则：
* 识别不安全的生命周期
* 关于使用过时字符串 ref API 的警告
* 关于使用废弃的 findDOMNode 方法的警告
* 检测意外的副作用
* 检测过时的 context API
#### 23. React如果进行代码拆分？拆分的原则是什么？
1. 首屏就要展示的组件不拆分
2. 根据路由进行拆分
#### 24. React组件的构造函数有什么用？
* 调用父类（即`React.Component`）的构造函数，内部`super(props)`，使当前类的`this`关键字可用（关于为什么要`super(props)`可以看[这里](https://overreacted.io/zh-hans/why-do-we-write-super-props/)）
* 对类中函数的`this`进行绑定
* 定义初始的`state`的值
* 定义实例变量，如`ref`
#### 25. React组件的构造函数是必须的吗？
不是。函数式组件没有构造函数，类组件，根据ES6的语法，不写入构造函数，默认也会载入构造函数里面只有一句代码`super(this)`用以调用父类的构造函数，用以让当前类得到`this`对象
#### 26. React中在哪捕获错误？
React可以使用特定的类组件包裹目标组件，进行错误边界的处理。错误边界组件需要声明静态方法`getDerivedErrorFromError(error)`和实例方法`componentDidCatch(error, errorInfo)`对子组件的错误进行捕获，在`render`方法中，可以自定义降级UI的渲染显示；示例代码如下;
```jsx
// 定义错误边界
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

// 使用
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```
#### 27. React怎么引入svg的文件？
1. 把svg视作和普通图片一样引入，然后在`img`标签中使用
2. `import {ReactComponent as Logo} from './logo.svg'`，然后把`Logo`作为React组件使用
#### 28. 说说你对Relay的理解
Relay是Facebook在React.js Conf（2015年1月）上首次公开的一个新框架，用于为React应用处理数据层问题。在Relay中，每个组件都使用一种叫做GraphQL的查询语句声明对数据的依赖。组件可以使用 this.props 访问获取到的数据。主要用于超大型项目中。（简单的说就是在使用GraphQL接口标准时，在React层的处理库）

#### 29. 在React中你有经常使用常量吗？
* 按照ES规范，代码中不变的量都会通过`const`定义成常量
* 函数式组件中，函数组件也会定义成一个常量
* 结合Redux使用时，把`actionType`定义成一个字符串常量
#### 30. 为什么说React中的props是只读的？
React是单项数据流，props是父组件到子组件的值，为了数据的稳当和代码可预见性，props被设计成只读的。
