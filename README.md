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
#### 4. 说说你对React的渲染原理的理解
React的渲染在`render`函数中进行，有以下几种方式会触发`render`
1. 初始渲染
2. `this.setState`方法，参数不能为`null`,及时`this.setState({})`,或者`setState`一个相同的值，也会导致render函数的执行
3. props的改变(采用浅比较)
4. `this.forceUpdate`,render函数中如果依赖了非`state`和`props`的其它变量，就需要用这个方式强制`render`函数执行
5. 父组件更新，会导致所有子组件的render方法执行
需要注意的是，render函数的执行并不一定会导致DOM的重新渲染，React库内部首先会对新旧的VNode进行DOM Diff，然后以最小的代价去更新DOM。虽然diff算法很快，但是当应用大了之后，也可能会导致性能问题，所以就有一下的性能优化点，来组织不必要的`render`函数执行。
1. 针对class组件，使用`React.PureComponent`
2. 针对函数式组件，使用`React.memo`
3. 合理的使用`shouldComponentUpdate`函数
4. 父组件对需要传递的子组件的props，合理使用`useCallback`的hook
5. 合理的拆分组件：组件粒度更细，避免大组件的渲染
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
使用了严格模式的好处就是在使用过时的api和不安全的方法时，能够获得警告，避免使用上面这些，使得代码更健壮；
#### 23. React如果进行代码拆分？拆分的原则是什么？
* pages：各路由级的页面文件，各子文件下又有当前页使用、复用度不高的业务组件
* components: 公共组件/公共的HOC
* layout: 公用的模块级组件
* apis: ajax以及接口的封装
* models/store： Mobx/Redux关于数据管理的封装
* utils：工具方法的封装
* config: 配置文件，工程中用到的常量配置 
* assets: 公共的静态资源
* styles: 公共的样式文件

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
### 31. 你有使用过formik库吗？说说它的优缺点
Formik提供了便捷的表单操作， 如获取表单数据，表单校验，提交事件等
结合yup来设置表单校验规则非常方便
(只用过andtd Form的表单验证，看了Formik的API，是概览是差不多的)
### 32. 你有用过哪些React的表单库吗？说说它们的优缺点
用过Antd的表单组件Form。
优点如下：
1. 自动的实现受控组件，实现双向绑定
2. 内置表单验证工具与表单错误的提示UI
3. 动态表单的封装
### 33. 如果组件的属性没有传值，那么它的默认值是什么？
Boolean值：`true`
### 34. 可以使用TypeScript写React应用吗？怎么操作？

可以，例如create-react-app脚手架，使用typescript模板就行，组件文件也可以是`.tsx`作为后缀。
```
yarn create react-app my-app --typescript
```
将ts添加到已经创建好的create react app项目中
```
yarn add typescript @types/node @types/react @types/react-dom @types/jest
```
在代码中需要足以泛型与接口的使用，如果props与state可以用泛型规定其结构，props的类型可以用typescript的强制类型来规范。
### 35. `super()`和`super(props)`有什么区别？
都是调用父类的构造函数，`super()`虽然没有传参，但是在React内部，还是把props挂载到了当前组件类上，但是使用`super()`的方式，在构造函数中都无法使用`this.props`（其它地方可以使用），`super(props)`，可以在构造函数中使用`this.props`.
### 36. 你有使用过loadable组件吗？它帮我们解决了什么问题？
使用过。主要解决打包文件体积过大的问题，用于代码分割。在React.lazy组件出现之前，原生React没有提供懒加载的方案，loadable库解决了这一问题。同时还提供加加载中的loading方案与加载失败的显示方案。
### 37. 你有使用过suspense组件吗？它帮我们解决了什么问题？
这个组件还是实验阶段，不能投入生产环境使用。
用于数据获取的 Suspense 是一个新特性，你可以使用 `<Suspense>` 以声明的方式来“等待”任何内容，包括数据。
示例如下
```jsx
const ProfilePage = React.lazy(() => import('./ProfilePage')); // 懒加载
// 在 ProfilePage 组件处于加载阶段时显示一个 spinner
<Suspense fallback={<Spinner />}>
  <ProfilePage />
</Suspense>
```
### 38. 怎样动态导入组件？
1. 使用第三方库，比如`react-loadable`;
```jsx
import Loadable from 'react-loadable';
import Loading from './my-loading-component';

const LoadableComponent = Loadable({
  loader: () => import('./my-component'),
  loading: Loading,
});

export default class App extends React.Component {
  render() {
    return <LoadableComponent/>;
  }
}

```
2. 使用`React.lazy`+`Suspense`
```jsx
// 使用 React.lazy
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
### 39. 如何给非受控组件设置默认的值？
给非受控组件的`defaultValue`属性赋值
### 40. 怎么在React中引入其它的UI库，例如Bootstrap
1. npm install antd
2. ```js 
    import { Button } from 'antd';
   ```
3. ```jsx
    const App = () => (<Button>button</Button>)
   ```
### 41. 怎样将事件传递给子组件？
React没有将事件和属性的传递分开，都是作为props进行传递，只是事件的prop是一个function，根据所需的业务逻辑，子组件在相应的时候触发`this.props.onPropFuc`即触发了父组件传递过来的方法。
### 42. 怎样使用Hooks获取服务端数据？
函数式组件没有class组件中生命周期的概念，class组件中一般是在`componentDidMount`中进行api请求，对应hook中的的代码如下所示：
```js
import { useEffect } from 'react';
const App = () => {
    useEffect(() => {
        // ajax request
    }, []);
    return null;
}
```
### 43. 使用Hooks要遵守哪些原则？
1. hooks只能在函数式组件和自定义hook中使用；
2. 为了保证执行的顺序，hook只能作用在函数的顶级作用域，不能写在循环、条件语句或嵌套函数中
### 44. render方法的原理你有了解吗？它返回的数据类型是什么？
render函数的渲染原理可以参考第4题。返回的数据类型是`ReactNode`。`ReactNode`是以下数据中的一种：
* string
* number
* boolean
* null
* ReactElement
* ReactNodeArray
### 45. useEffect和useLayoutEffect有什么区别？
* useEffect 是异步执行的，而useLayoutEffect是同步执行的
* useEffect的执行时机是浏览器完成渲染之后，useLayoutEffect的执行时机是还没有渲染到DOM之前，和componentDidMount等价
### 46. 在React项目中你用过哪些动画的包？
react-transition-group: 一般的、比较简单的动画
react-motion: 复杂、精致的动画
### 47. React必须使用JSX吗?
不是必须的。`JSX`是`React.createElement(type, props, children)`的语法糖，以下分别用JSX和createElement代码，render的结果都是完全一致的。
```jsx
// jsx语法
import React, { useState, useEffect } from 'react';

const App = () => {
    const [list, setList] = useState([]);
    useEffect(() => {
        setList([1, 2, 3])
    }, []);

    return <ul className="container">
        {list.map(e => <li key={e}>{e}</li>)}
    </ul>
}

export default App;

// React.createElement
import React, { useState, useEffect } from 'react';

const App = () => {
    const [list, setList] = useState([]);
    useEffect(() => {
        setList([1, 2, 3])
    }, []);

    return React.createElement(
        'ul',
        { className: 'container', },
        list.map(e => {
            return React.createElement('li', { key: e, }, e);
        }),
    )
}

export default App;
```
### 48. 自定义组件时render是可选的吗？为什么？
函数式组件没有render方法。自定义class组件的render是必须的。
原因：render函数是class组件的核心，它返回虚拟DOM，如果没有虚拟DOM，那么组件就没有意义。
补充：当一个自定义class组件，继承另一个组件的时候，render不是必须的，它会自动继承父类的render方法。
### 49. 需要把keys设置为全局唯一吗？
不需要，只需要在同层级保持唯一即可。最好用id作`key`值，尽量不要用索引作为`key`值。
### 50. 怎么定时更新一个组件？
使用定时器，注意在组件销毁的生命周期清除定时器。
```jsx
// class组件
class Clock extends React.Component{
    constructor(props){
        super(props);
        this.state={date:new Date()};
    }
    componentDidMount(){
        this.timerID=setInterval(()=>this.tick(),1000);
    }
    componentWillUnmount(){
        clearInterval(this.timerID);
    }
    tick(){
        this.setState({
            date:new Date()
        });
    }
    render(){
        return (
            <div>
                <h2>Timer {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        );
    }
}
ReactDOM.render(
    <Clock />,
    document.getElementById('root')
);

// hook
import React, { useState, useEffect } from 'react'

export default function TimerHooks () {
  const [date, setDate] = useState(new Date())

  useEffect(() => {
    let timerId = setInterval(() => {
      setDate(new Date())
    }, 1000)

    return () => {
      clearInterval(timerId)
    }
  }, []);

  return (
    <div>
      <p>时间: {date.toLocaleTimeString()}</p>
    </div>
  )
}
```
### 51. React根据不同的环境打包不同的域名？
如果是`create-react-app`,可以通过项目根目录的`.env`、`.env.development`、`.env.production`来区分不同的环境（打包命令），在上面的文件内写入`REACT_APP_`作为前缀的环境变量，在业务代码中读取相应的环境变量即可。
不管是什么脚手架工具的项目，都可以通过在`package.json`的`scripts`，中通过`cross-env`写入环境变量，如下：
```
"dev:test": "cross-env REACT_APP_BASEURL=http://test.domain.com react-scripts start"
```
### 52. 使用webpack打包React项目，怎么减小生成的js大小？
* 使用react-loadable进行懒加载
* webpack splitChunkPlugin进行代码分割
* webpack UglifyjsWebpackPlugin进行代码压缩
* webpack CompressionWebpackPlugin进行网络传输压缩gzip
* webpack mini-css-extract-plugin抽取CSS代码
### 53. 在React中怎么使用async/await？
`create-react-app`、`umi`等搭建的项目都可以直接使用，如果脚手架不支持，可以安装Babel插件`@babel/plugin-transform-async-to-generator`，并在`.babelrc`文件中加上以下的配置
```json
{
  "plugins": [
    [
      "@babel/plugin-transform-async-to-generator",
      // 以下是可选配置
      {
        "module": "bluebird",
        "method": "coroutine"
      }
    ]
  ]
}
```
### 54. 你阅读了几遍React的源码？都有哪些收获？你是怎么阅读的？
(0遍，这道题没法回答😅)
### 55. 什么是React.forwardRef？它有什么作用？
背景：
作用:
* 转发 refs 到 DOM 组件
* 在高阶组件中使用，通过props中转转发ref到WrappedComponent,例子如下:
```jsx
import React, { useRef } from 'react';

// 子组件
class CustomInput extends React.Component {

    inputRef = React.createRef(null);

    focus() {
        this.inputRef.current.focus();
    }

    render() {
        const { count } = this.props;
        return (
            <p>
                <span>自定义input: {count}</span>
                <input ref={this.inputRef} />
            </p>
        )
    }
}

// 高阶组件封装
function hoc(WrappedComponent) {
    class HWrappedComponent extends React.Component {
        render() {
            const { forwardRef, ...rest } = this.props;
            return (
                <WrappedComponent {...rest} ref={forwardRef} />
            )
        }
    }

    return React.forwardRef((props, ref) => {
        return <HWrappedComponent {...props} forwardRef={ref} />
    })

}

const HocCustomInput = hoc(CustomInput);

const App = () => {
    const ref = useRef(null);
    const handleFocus = () => {
        ref.current.focus();
    }
    return (
        <>
            <HocCustomInput ref={ref} count={12} />
            <button onClick={handleFocus}>focus</button>
        </>
    )
};

export default App;
```
### 56. 写个例子说明什么是JSX的内联条件渲染
```jsx
import { useState } from "react";

const App = () => {
    const [flag] = useState(true)
    const button = null;
    if (flag) {
        button = <button>1</button>
    } else {
        button = <button>0</button>
    }
    return (
        <>
            { flag ? <h1>true</h1> : <h1>false</h1> }
            { flag && <span>ok</span> }
            { button }
        </>
    )
};

export default App;
```
### 57. 在React中怎么将参数传递给事件？ 
箭头函数和`Function.prototype.bind`来实现事件中参数的传递.
```jsx
import { useState } from "react";

const App = () => {
    const handleA = (...args) => {
        console.log(args); // [syntheticEvent, 'a', 'b']
    }
    const handleB = (...args) => {
        console.log(args); // ['a', 'b', syntheticEvent]
    }
    return (
        <>
            <button onClick={event => handleA(event, 'a', 'b')}>AAA</button>
            <button onClick={handleB.bind(null, 'a', 'b')}>BBB</button>
        </>
    )
};

export default App;
```
### 58. React的事件和普通的HTML事件有什么不同？
React的事件系统是合成事件。
在React17之前，所有的事件都是挂载在document上的，页面中各节点的事件都是在document上触发通过事件委托分发下来的。
React17的版本中，所有事件变更到挂载到`ReactDOM.render`挂载的根节点。并且取消了事件池
优点：将事件绑定在document统一管理，防止很多事件直接绑定在原生的dom元素上。造成一些不可控的情况;React 想实现一个全浏览器的框架，为了实现这种目标就需要提供全浏览器一致性的事件系统，以此抹平不同浏览器的差异。

### 59. 在React中怎么阻止事件的默认行为？
```javascript
class Component extends React.Component {
    handleEvent = event => {
        // 合成事件的方法，非原生的preventDefault
        event.preventDefault();
    }
    render () {
        //...
    }
}
```
### 60. 你最喜欢React的哪一个特性（说一个就好）？
Hooks.可以对代码进行解耦，更优雅、直观的拆分和复用代码。
### 61. 在React中什么时候使用箭头函数更方便呢？
* 事件函数需要传参
* class组件中需要绑定事件的`this`，可以使用`public class fields`语法，用箭头函数的方式定义一个函数是类的属性
### 62. 你最不喜欢React的哪一个特性（说一个就好）？
* CSS的方案还有缺陷，利用css module可以解决css作用域的问题，但是在代码调试上因为浏览器的类名和代码类名不一致，所以并不直观。
### 63. 说说你对React的reconciliation（一致化算法）的理解
reconciliation指的是React在render之后对旧的元素树转换成新的元素树的最小操作的一套算法。该算法主要通过以下两方面进行优化。
* 两个不同类型的元素会产生不同的树
    - 对比不同类型的元素时，React对卸载掉整个元素以及其子元素，用新的元素来替代
    - 对比相同类型的元素时，React会保持元素不变，仅更新组件中改变了的属性，处理完成之后，继续递归对子节点进行比较
    - 对比同类型的组件时，组件的实例会保持不变，因为可以保持state的不变，React将更新props，触发组件更新相关的生命周期
* 通过开发者给组件制定`key`属性，来告知渲染哪些子元素在不同的渲染下可以保持不变
    - `key`属性解决的就是列表在进行一对一比较的过程中，新元素树，从中间或者顶部插入的问题；例如，从顶部插入，那么react一对一的往下比较，那么每次比较都是不相同的，react会重建每一个元素；指定了key值之后，react会按照key值进行比较，react就会知道原有的列表只是往下移动了而已，创建的元素只有顶部的一个
### 64. 使用PropTypes和Flow有什么区别？
* Flow 是一个针对react项目所有 JavaScript 代码的静态类型检测器，需要单独添加依赖并手动运行（换成ts也是类似的答案）
* PropTypes是针对组件级别的类型检测
### 65. 怎样有条件地渲染组件？
（同[第56题](#56-写个例子说明什么是JSX的内联条件渲染)）
### 66. 在JSX中如何写注释？
注释属于js语法层面的代码，所以jsx中的注释需要用`{ }`包住，且仅支持`/* */`这种注释的形式，如下：
```jsx
const Demo = () => (
    <div>
        {/* 注释内容 */}
        <span>
            123
        </span>
    </div>
);
```
### 67. constructor和getInitialState有不同？
constructor是生命周期的第一步，用于做一些初始化的工作，比如state的初始值，事件this的绑定等。
(getInitialState是老API了，相当于ES5使用React.createClass下的构造函数，不用了解)
### 68. 写例子说明React如何在JSX中实现for循环
```JSX
const Demo = () => {
    const list = [1,2,3];
    return (
        <ul>
            {/* Array.prototype.filter 同理 */}
            { list.map(e => <li key={e}>{e}</li>) }
        </ul>
    )
}
```
### 69. 为什么建议Fragment包裹元素？它的简写是什么？
动机：jsx表达式必须用一个父元素把它们包起来，Fragments用于一个组件返回多个自元素列表的情况，需要用`<React.Fragment>`把它们包起来，避免总是需要使用一个父元素如`<div>`在包裹组件。且`<React.Fragment>`不会产生多余的DOM标签。
简写：`<>{ReactNodeList}</>`，注意简写不能加上`key`属性，只有`<React.Fragment>`的写法才可以
### 70. 你有用过React.Fragment吗？说说它有什么用途？
有用过。jsx表达式必须用一个父元素把它们包起来，Fragments用于一个组件返回多个自元素列表的情况，需要用`<React.Fragment>`把它们包起来，避免总是需要使用一个父元素如`<div>`在包裹组件。且`<React.Fragment>`不会产生多余的DOM标签。
### 71. 在React中你有遇到过安全问题吗？怎么解决？
* XXS攻击：使用了`dangerouslySetInnerHtml`属性，可以直接调用浏览器原生的`innerHTML`接口，设置富文本，可能会导致XXS攻击的问题，所以需要对用户设置的内容进行过滤
### 72. React中如何监听state的变化？
* class组件使用`shouldComponentUpdate`
```jsx
import React from 'react';
export default class Demo extends React.Component {
    state = { count: 1 }
    handleAdd = () => {
        this.setState(prevState => ({
            count: prevState.count + 1,
        }))
    }
    shouldComponentUpdate(nextProps, nextState) {
        if (nextState.count !== this.state.count) {
            console.log({
                'old count': this.state.count,
                'new count': nextState.count,
            })
        }
        return true;
    }
    render () {
        return <>
            <h1>{this.state.count}</h1>
            <button onClick={this.handleAdd}>ADD</button>
        </>
    }   
}
```
* 函数式组件使用`useEffect`
```jsx
import { useState, useEffect, useRef } from 'react';

function useUpdate(fn, deps) {
    const isFirst = useRef(true);
    useEffect(() => {
        if (isFirst.current) {
            return isFirst.current = false;
        }
        fn()
    }, deps)
}

function usePrevValue (value) {
    const prevRef = useRef(value);
    useEffect(() => {
        prevRef.current = value;
    }, [value]);
    return prevRef.current;
}

const Demo = () => {
    const [count, setCount] = useState(0);
    const prevCount = usePrevValue(count);
    useUpdate(() => {
        console.log({
            'old count': prevCount,
            'new count': count,
        })
    }, [count])

    return <>
        <h1>{count}</h1>
        <button onClick={() => setCount(count + 1)}>ADD</button>
    </>
}
export default Demo;
```
### 73. React什么是有状态组件？
状态即是React中的state，有状态组件就是React组件中有state在维护的组件，在Hooks之前，一般class组件都是有状态组件(如果只用props，也是无状态组件)，Functional组件都是无状态组件
### 74. React v15中怎么处理错误边界？
(过时的API，无需关注，最新的错误边界处理可以参考[第26题](#26-React中在哪捕获错误？)
### 75. *React Fiber它的目的是解决什么问题？
React 15 的 StackReconciler 方案由于递归不可中断问题，如果 Diff 时间过长（JS计算时间），会造成页面 UI 的无响应（比如输入框）的表现，vdom 无法应用到 dom 中。

为了解决这个问题，React 16 实现了新的基于 requestIdleCallback 的调度器（因为 requestIdleCallback 兼容性和稳定性问题，自己实现了 polyfill），通过任务优先级的思想，在高优先级任务进入的时候，中断 reconciler。

为了适配这种新的调度器，推出了 FiberReconciler，将原来的树形结构（vdom）转换成 Fiber 链表的形式（child/sibling/return），整个 Fiber 的遍历是基于循环而非递归，可以随时中断。

更加核心的是，基于 Fiber 的链表结构，对于后续（React 17 lane 架构）的异步渲染和 （可能存在的）worker 计算都有非常好的应用基础

### 76. React为什么不要直接修改state？如果想修改怎么做？
直接修改state的情况，React库内部无法检测到state的变化，从而来触发re-render。如果要修改state的值，需要在组件中调用`this.state()`方法.
```jsx
```
### 77. create-react-app有什么好处？
* 快速创建React标准工程（React+React-Router）
* 提供了可选择的ESLint、CSS扩展语言的配置
* 提供了诸如typescript等各种开发模版
### 78. 装饰器(Decorator)在React中有什么应用？
在React中装饰器的本质就是用装饰器函数包裹一个类，所以，所有的HOC方案，都可以用装饰器的语法糖来调用，显得非常的直观。
如自定义的HOC，react-router的withRouter函数，react-redux的connect函数，Mobx中的@observable、@computed、@action,mobx-react中的@observer.
### 79. 使用高阶组件(HOC)实现一个loading组件
```JSX
// app.jsx
import React from 'react';
import Child from './Child.jsx';
import HocLoading from './hocLoading';

const LoadingChild = HocLoading(Child);

class Demo extends React.Component {

    state = {
        loading: true,
    }

    componentDidMount() {
        setTimeout(() => {
            this.setState({
                loading: false,
            })
        }, 2000)
    }

    render() {
        return <LoadingChild loading={this.state.loading}>parent content</LoadingChild>
    }
}

export default Demo;

// hocLoading.js
import React from 'react';
export default function HocLoading (WrappedComponent) {
    class ComponentLoading extends React.Component {
        render () {
            const { loading } = this.props;
            return (
                loading
                ? <h4>loading...</h4>
                : <WrappedComponent {...this.props} />
            )
        }
    }
    return ComponentLoading;
}

// Child.jsx
import React from 'react';

class Child extends React.Component {
    render() {
        return <>
            <h1>This is Content!</h1>
            <div>{this.props.children}</div>
        </>
    }
}

export default Child;
```
### 80. *如何用React实现滚动动画？
1. 监听`window.scroll`事件，在事件回调中监听`document.documentElement.scrollTop`的值`y`
2. 根据需求，当`y`值到达一定的值时，利用state控制需要动画的目标元素的显隐
3. 利用`react-transition-group`库控制动画
### 81. 说出几点你认为的React最佳实践
1. 合理的精细化组件，避免单个组件文件的代码行数过长
2. 像后台管理这种通用型比较强的界面，可以把相同的逻辑抽离成业务hook
3. 状态组件和无状态组件分离，状态组件做逻辑的处理，无状态组件只是一个纯函数，做纯粹的展示
4. 为了性能的考虑，尽量使用函数式组件
5. 使用React.PureComponent React.memo useMemo useCallback等结合使用，优化组件更新时的渲染性能
### 82. 你是如何划分React组件的？
* layout: 结构性的展示型组件，一般是无状态的，如header、footer等
* components: 公用的组件，既有props的传入，内部也有自己的state进行处理
* pages: 页面级组件，有状态组件，通过调用上面两种类型的组件，传入相应的props，一般与路由挂在一下，逻辑上会进行ajax请求等副作用操作
### 83. 举例说明如何在React创建一个事件
```jsx
/** 原生的javascript也有事件的emit，这里是借用的node的events模块 */
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
var EventEmitter = require('events').EventEmitter;
let emitter = new EventEmitter();

class ListItem extends Component {
  static defaultProps = {
    checked: false
  };
  constructor(props) {
    super(props);
  }
  render () {
    return (
      <li>
        <input type="checkbox" checked={this.props.checked} onChange={this.props.onChange} />
        <span>{this.props.value}</span>
      </li>
    );
  }
}

class List extends Component {
  constructor(props) {
    super(props);

    this.state = {
      list: this.props.list.map(entry => ({
        text: entry.text,
        checked: entry.checked || false
      }))
    };
    console.log(this.state);
  }

  onItemChange (entry) {
    const { list } = this.state;
    this.setState({
      list: list.map(prevEntry => ({
        text: prevEntry.text,
        checked: prevEntry.text === entry.text ? !prevEntry.checked : prevEntry.checked
      }))
    });
    //这里触发事件
    emitter.emit('ItemChange', entry);
  }
  render () {
    return (
      <div>
        <ul>
          {this.state.list.map((entry, index) => {
            return (
              <ListItem
                key={`list - ${index}`}
                value={entry.text}
                checked={entry.checked}
                onChange={this.onItemChange.bind(this, entry)} />
            );
          })}
        </ul>
      </div>
    );
  }
}

class App extends Component {
  constructor(props) {
    super(props);
  }
  componentDidMount () {
    this.itemChange = emitter.addListener('ItemChange', (msg, data) => console.log(msg));//注册事件
  }
  componentWillUnmount () {
    emitter.removeListener(this.itemChange);//取消事件
  }
  render () {
    return (
      <List list={[{ text: 1 }, { text: 2 }]} />
    )
  }
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
### 84. 如何更新组件的状态？
class组件：state的变化，即`this.setState`，还有props的变化，都会引起re-render，diffDOM之后，如果两次渲染的VNODE有差异，就会引起组件的更新。还可以使用`this.forceUpdate`，但不推荐使用，一般是render中依赖了除state和props的变量才会使用。
函数是组件：useState中state的变化。
### 85. 怎样将多个组件嵌入到一个组件中？
* 作为组件的children
* render props
### 86. React的render中可以写{if else}这样的判断吗？
不能。jsx中只能写js表达式，不能写js语句。
关于js表达式和js语句的区别：语句是为了进行某种操作，一般情况下不需要返回值，而表达式都是为了得到返回值，一定会返回一个值（这里的值不包括undefined）；写之前可以想象jsx中的js代码是否能够放在`if()`中，如果可以，就可以放在jsx中。
### 87. React为什么要搞一个Hooks？
降低耦合，提高代码的复用性。同时给函数式组件赋予了状态，class组件能做的事情，几乎都可以用函数式组件来实现，提高性能。
在传统的class组件中，UI和逻辑是绑定在一起的，虽然可以通过hoc的方式进行抽离，但是hoc这种方式并不直观，理解成本较大，hook则解决了这一问题，hooks的书写方式，代码简单明了，复用行极强，通过hooks提供的`useState`实现了数据的响应，同时通过`useEffect`，又实现了对各生命周期的模拟，大大加强了hooks的功能性。
### 88. React Hooks帮我们解决了哪些问题？
见[第87题](#87-React为什么要搞一个Hooks？)
### 89. *使用React的memo和forwardRef包装的组件为什么提示children类型不对？
过去使用Component、FC等类型定义组件时一般不需要我们定义props里children的类型，因为在上述类型里已经帮你默认加上了 { children?: ReactNode } 的定义。但是@types/react的维护者认为这样导致children几乎没有类型约束，组件开发者应该显式地声明children类型。所以对新的API应该都不会自动加上children的定义了，需要开发者手动添加。（[这里](https://github.com/haizlin/fe-interview/issues/844)找的答案，没看懂，说的好像是ts react吧）
### 90. 有在项目中使用过Antd吗？说说它的好处
使用过。
好处：
* 统一精美的UI
* 大量现成高质量的基础组件，丰富的API，方便快速开发
* tree-shaking，实现了按需加载，减少代码体积
* 活跃的社区氛围，周边组件也比较丰富

### 91. 在React中如何去除生产环境上的sourcemap？
create-react-app脚手架中,使用了`build`命令后，生产环境中的代码就已经去除了sourcemap了。本质上来说，是否sourcemap都是webpack来进行配置的，webpack中的devtool选项来控制sourcemap的类型，如果要去除source，则省略devtool选项
### 92. 在React中怎么引用sass或less？
利用webpack，引入node-sass sass-loader或者less-loader,并根据样式文件的后缀名，在webpack中进行rule的配置，配置完成了，webpack则会正确的解析sass何less文件。
### 93. 组件卸载前，加在DOM元素的监听事件和定时器要不要手动清除？为什么？
要。一般在`componentWillUnMount`或者是函数式组件的`useEffect`函数的返回函数中进行清除动作。
因为，无论是事件的监听、定时器操作都不回随着组件的卸载而清除，如果不手动进行清除的话，会一直占用浏览器的内存，有内存泄漏的风险
### 94. 为什么标签里的for要写成htmlFor呢？
jsx是在js中的，`for`是js中的关键字，所以改成了`htmlFor`
### 95. 状态管理器解决了什么问题？什么时候用状态管理器？
状态管理器解决的问题
1. 让组件的通信不再局限于通过props父传子这种模式，可以在任意组件间进行传递
2. 数据状态与UI组件分离，减少耦合
3. 状态管理器的各种库，往往限制了数据操作的方法（如Redux只能通过`dispatch`操作数据），确保数据的操作流向，利于问题的排查
什么时候使用，见[第1题](#1-什么时候使用状态管理器？)
### 96. 状态管理器它精髓是什么？
* 数据从各种UI组件中抽离，数据可以注入到各层级的组件
* 通过react-redux、mobx-react等中间库，让数据响应UI的变化
* 限制数据的操作方式，确保数据的流向
### 97. 函数式组件有没有生命周期？为什么？
v16.8前，没有，那时候的函数式组件就是无状态组件，只用在简单的数据展示。
v16.8之后，也没有，随着hooks的引入，函数式组件看起来像有生命周期了，用useEffect函数就可以模拟，但是最要抛弃生命周期的概念，useEffect只是处理副作用，下面是模拟各生命周期用useEffect函数的实现
```jsx
import { useRef, useEffect, useState } from "react";

function useMount (fn) {
    useEffect(fn, []);
}

function useUpdate (fn) {
    const isFirst = useRef(true);
    useEffect(() => {
        if (isFirst.current) {
            isFirst.current = false;
            return;
        }
        fn();
    })
}

function useUnMount (fn) {
    useEffect(() => {
        return fn;
    }, [])
}

const TComponent = props => {
    useMount(() => {
        console.log('componentDidMount');
    });
    useUpdate(() => {
        console.log('componentDidUpdate');
    });
    useUnMount(() => {
        console.log('componentWillUnMount');
    });
    return <h1>TComponent: {props.children}</h1>
}

const Demo = () => {
    const [visible, setVisible] = useState(true);
    const [count, setCount] = useState(0);
    return <>
        <button onClick={() => { setVisible(!visible) }}>Toggle</button>
        <button onClick={() => { setCount(count + 1) }}>Update</button>
        { visible && <TComponent>{count}</TComponent> }
    </>
}

export default Demo;
```
### 98. 在React中怎么引用第三方插件？比如说jQuery等
(这个问题奇怪)
1. 安装`npm i jquery -S`
2. 引入`import $ from 'jquery';`
3. 使用`$(ref)`,注意使用ref的方式引用api，且要在组件挂载到dom之后，才能引用
### 99. React的触摸事件有哪几种？
1. onTouchStart
2. onTouchMove
3. onTouchEnd
4. onTouchCancel：当一些更高级别的事件发生的时候（如电话接入或者弹出信息）会取消当前的touch操作，即触发ontouchcancel
### 100. 路由切换时同一组件无法重新渲染的有什么方法可以解决？
切换前后给无法重新渲染的组件赋予不同的key
### **101. React16新特性有哪些？
* 内部diff算法用Fiber实现，主要是异步渲染，防止主线程被阻塞
* render 支持返回数组和字符串
* 错误边界
* Portals
* React.createContext
* Profiler
* Hooks
* React.lazy
* React.memo
### 102. 你有用过哪些React的UI库？它们的优缺点分别是什么？
用过antd。增加开发的便利性，大量可以直接使用的组件。
### 103. `<div onClick={handlerClick}>单击</div>`和`<div onClick={handlerClick(1)}>单击</div>`有什么区别？
onClick后大括号中的表达式的返回值应该是一个函数，本题中，前者`handlerClick`是一个函数，点击div后触发这个方法，是常规的用法；后者`handlerClick(1)`，一开始`handlerClick(1)`就会执行，如果执行后返回值是函数fn，那么点击div后触发的方法是fn.
### 104. 在React中如何引入图片？哪种方式更好？
1. 
```jsx
import logo from '@/assets/logo.png'
const Demo = () => (
    <div>
        <img src={logo} />
    </div>
)
```
2. 
```jsx
const Demo = () => (
    <div>
        <img src={require('@/assets/logo.png').default} />
    </div>
)
```
应该都是一样的，都会经过webpack的处理，如果满足webpack的 `url-loader`,`options.limit`的规定值以下，就会把图片转化成base64进行引用

### 105. 在React中怎么使用字体图标？
1. 在工程中加入字体文件
2. 在css文件中`@font-face`定义字体名称、引用字体文件
3. react工程的入口引入上面的css文件
4. 在组件的className中添加自定义字体对应的类名即可使用
### 106. React的应用如何打包发布？它的步骤是什么？
在使用create-react-app脚手架的工程中，打包的命令是`build`,使用`build`命令进行打包了之后，会在dist中生成打包后的文件，将打包后的文件丢到服务器中指定的目录即可。
### 107. ES6的语法'...'在React中有哪些应用？
* `this.setState`，设置一个对象时，保持对象是不可变数据，需要用对象的拓展运算符合并就对象的属性，需要修改的目标对象放在最后用以被覆盖
* Rudex的reducer中需要返回新的state，也要保持对象的不可变数据，返回一个全新的对象
* props的透传（一般用在hoc中）
### 108. 如何封装一个React的全局公共组件？
（我这里理解的全剧公共组件是像Vue一样，使用Vue.component全局注册，React中好像没有类似的方法）
### 109. 在React中组件的props改变时更新组件的有哪些方法？
这题我理解的是props改变导致render历经的生命周期，依次如下：
1. `static getDerivedStateFromProps`
2. `shouldComponentUpdate`
3. `render`
4. `getSnapshotBeforeUpdate`
5. `componentDidUpdate`
### 110. immutable的原理是什么？
React判断组件是否更新，在比较state和props时，使用的是浅比较。
对于引用类型来说，浅比较即`===`,如果直接修改对象的属性，那么对象的引用地址没有发生变化，则不会引起触发react的更新；所以在更新state和props时候，都必须使用immutable数据，保持更新后的数据引用地址是改变了的。
### 111. 你对immutable有了解吗？它有什么作用？
见[第110题](110-immutable的原理是什么？)
### 112. 如何提高组件的渲染效率呢？
* class组件继承React.PureComponent，也可以使用shouldComponentUpdate根据业务判定是否需要更新组件
* function组件用React.memo hoc一层
* 在jsx中事件的定义不要用箭头函数
* 使用useMemo、useCallback缓存数据和函数
* 是使用Redux时，精细化mapStateToProps的返回值（不要一次性穿过整个state），可以使用reselect缓存selector的数据
* 使用`key`
### 113. 在React中如何避免不必要的render？
见[第112题](112-如何提高组件的渲染效率呢？)
### 114. render在什么时候会被触发？
见[第4题](4-说说你对React的渲染原理的理解)
### 115. 写出React动态改变class切换组件样式
```jsx
import { useState } from 'react';
const Demo = () => {
    const [hidden, setHidden] = useState(false);
    return <>
        <h1 class={hidden ? 'hidden' : 'visible'}>hello</h1>
    </>
}

// 一般对css class的处理我们都会引入classnames，会更方便
import { useState } from 'react';
const Demo = () => {
    const [hidden, setHidden] = useState(false);
    return <>
        <h1 class={classnames({
            hidden: hidden,
            visible: !hidden,
        })}>hello</h1>
    </>
}
```
### 116. *React中怎么操作虚拟DOM的Class属性？
（这道题问的有点奇怪，没看懂）
### 117. 为什么属性使用className而不是class呢？
jsx本质上还是js语句中的一部分，`class`是js的关键字，不能直接使用
### 118. 请说下react组件更新的机制是什么？
不管是class组件还是函数式组件，React的组件本质上都是函数，从根组件到下面大大小小的子组件，React组件树即构成了函数的嵌套，React的渲染/更新即时从顶层函数执行，一步一步的递归执行内部嵌套函数的过程，这是一个原生的JS执行栈执行的过程。
在React16之前，这个执行过程是不能够被打断的，会一次性的同步直接完，所以可能会导致事件响应延迟、动画卡顿等性能问题。
React16之后，随着Fiber机制的引入，这个执行的过变成了异步的，Fiber会按照浏览器原生的requestIdelCallback API对整个执行过程进行时间分片，在每一个时间片内，除了进行react的解析执行之外，同事也会进行动画、事件响应等操作，保持页面的流畅性
### 119. 怎么在JSX里属性可以被覆盖吗？覆盖的原则是什么？
可以，后面的覆盖前面的，确保目标属性放后面即可
```jsx
const Demo = props => {
    // Child组件的count属性始终会被5这个值覆盖
    return <Child {...props} count={5} />
}
```
### 120. 怎么在JSX里使用自定义属性？
和原生html一样，用`data-`前缀的属性即可
```jsx
const Demo = () => <h1 data-age="12">hello<h1>;
// 可以用以下的原生js方法进行取值
h1Ele.dataset.age // 12(string)
```
### 121. 怎么防止HTML被转义？
React会将所有要显示到DOM的字符串转义，防止XSS。所以，如果JSX中含有转义后的实体字符，比如& copy; (©)，则最后DOM中不会正确显示，因为React中自动把& copy；中的特殊字符转义了。解决办法为：
1. 直接使用UTF-8字符 ©（在mac中使用 option 或 option+shift，再加其它键就可以快速输入符号
2. 使用对应字符的Unicode编码；
3. 使用数组组装
4. 直接插入原始的HTML。
```jsx
const Demo = () => (
    <>
        {/* 有问题，显示是 &copy; */}
        <div>{'&copy;'}</div>
        {/* 以下的都ok，显示是 © */}
        <div>{'©'}</div>
        <div>{'\u00a9'}</div>
        <div>{[<span>&copy;</span>]}</div>
        <div dangerouslySetInnerHTML={{__html: '&copy;'}}></div>
    </>
);
```
### 122. 经常用React，你知道React的核心思想是什么吗？
* MVVM(Model-View-ViewModel)，从数据到视图的单项数据流，数据的变化会引起视图层的更新，一般不会直接操作DOM
* 组件化，props单项传递
### 123. 在React中我们怎么做静态类型检测？都有哪些方法可以做到？
* 通过第三方的库来做，比如`prop-types`等
* 通过静态语法检查，比如`typescript``flow`等
### 124. 在React中组件的state和setState有什么区别？
state是组件实例中挂载的状态值，setState是组件实例挂载的方法，作用是改变state的值，并且通知到view层，引起UI层的刷新，setState是异步的。
### 125. React怎样跳过重新渲染？
1. 针对class组件，使用`React.PureComponent`
2. 针对函数式组件，使用`React.memo`
3. 合理的使用`shouldComponentUpdate`函数
4. 父组件对需要传递的子组件的props，合理使用`useCallback`、`useMemo`的hook,保证props的最小变化
### 126. React怎么判断什么时候重新渲染组件呢？
这里是重新渲染理解成重新渲染DOM的话，React的判断依据是通过render前后，对比两次的虚拟DOM树，如果前后两次的DOM树完全相同，则不会重新渲染，否则会根据两次树的差异，重新渲染部分DOM。

如果重新渲染指的是re-render（即执行render方法）的话，有以下的方法都会使组件re-render
1. 初始渲染
2. `this.setState`方法，参数不能为`null`,及时`this.setState({})`,或者`setState`一个相同的值，也会导致render函数的执行
3. props的改变(采用浅比较)
4. `this.forceUpdate`,render函数中如果依赖了非`state`和`props`的其它变量，就需要用这个方式强制`render`函数执行
5. 父组件更新，会导致所有子组件的render方法执行
### 127. 什么是React的实例？函数式组件有没有实例？
针对class组件，React的实例指的是这个Calss继承`React.Component`所产生的实例，如state、方法都是挂载在这个实例上，组件每引用一起，就会生成一次组件的实例。
函数式组件不是构造函数的调用模式，本质上只是普通函数，所以没有实例。
### 128. 在React中如何判断点击元素属于哪一个组件？
利用ref的api，然后使用原生DOM的`contains`方法进行判断，demo如下
```jsx
import React from 'react';

const Child = React.forwardRef((props, ref) => {
    return <div ref={ref}>
        <button onClick={props.onClickButton}>aaa</button>
    </div>
})

class Demo extends React.Component {
    childRef = null;
    handleClick = (e) => {
        console.log(this.childRef.contains(e.target)); // true
    }
    render() {
        return <>
            <Child onClickButton={this.handleClick} ref={ref => this.childRef = ref} />
        </>
    }
}


export default Demo;
```
### 129. 在React中组件和元素有什么区别？
* React组件分两种，class组件和函数式组件，它的本质是一个类或者普通函数
* React元素，render函数的返回值，或者是函数式组件的返回值可以是一个react元素，它是一个`JSX`表达式，本质上是一个对象;利用`React.createElement`,`React.cloneElement`方法的返回值也是React元素
### 130. 在React中声明组件时组件名的第一个字母必须是大写吗？为什么？
是的。因为在html标准中，标签命都是小写中间横杠的命名方式，React为了与其区分，防止与现有或者以后出现的html标签重名，约定采用大驼峰的命名方式,Babel在对jsx进行解析时也是以此为规则，首字母是大写即使React组件，否则Babel会识别成普通html标签.
### 131. 举例说明什么是高阶组件(HOC)的反向继承？
```jsx
import React from 'react';
class Person extends React.Component {
    state = {
        name: 'vk',
        age: 12,
    }
    componentDidMount () {
        console.log('mounted')
    }
    render () {
        const { name, age } = this.state;
        return <div>
            <div>名字: {name}</div>
            <div>年龄: {age}</div>
            <div>props#count: {this.props.count}</div>
        </div>
    }
}

const hoc = WrappedComponent => class HocComponent extends WrappedComponent {
    // state和生命周期还有其它的方法都会进行完全的复写
    state = {
        name: 'bo',
        age: 21,
    }
    componentDidMount () {
        console.log('hoc mounted')
    }
    render () {
        return (
            <div>
                <p>高阶组件</p>
                { super.render() }
            </div>
        )
    }
}
const HocPerson = hoc(Person);
const Demo = () => <HocPerson count={111} />
export default Demo;
```
### 132. 有用过React Devtools吗？说说它的优缺点分别是什么？
用过。

优点：
* 可以看到组件的嵌套状态，也可以针对高阶组件比较多的情况，也可以筛选组件的嵌套状态
* 可以直观的看到组件的props/state等值，并可以进行直接的修改
* 选中组件后，可以在console通过`$r`变量引用组件，进行事件触发等操作
* 可以通过组件直接定位到真实DOM的elements

缺点: 
* 对于hooks的支持不太好，无法直观的看出hooks的值
### **133. 举例说明什么是高阶组件(HOC)的属性代理？
```jsx
/** 比如修改第三方库 antd，把Button组件的loading属性，改成wait */
import React from 'react';
import { Button } from 'antd';
const hoc = WrappedComponent => {
    class HocComponent extends React.Component {
        render () {
            const { forwardRef, wait, ...rest } = this.props;
            const newProps = {
                ...rest,
                loading: wait,
            }
            return <Button ref={forwardRef} {...newProps}  />
        }
    }
    return React.forwardRef((props, ref) => {
        return <HocComponent forwardRef={ref} {...props} />
    })
}

const NewButton = hoc(Button);

const Demo = () => {
    // loading的prop已经不生效了，用wait代替
    return <NewButton wait={true}>你好</NewButton>
}

export default Demo;
```
### 134. React的isMounted有什么作用？
(废弃api，不用关心)
### 135. React组件命名推荐的方式是哪个？为什么不推荐使用displayName？
React通过大驼峰的形式命名组件，无论是class组件还是函数式组件，都尽量不要使用匿名导出的方式，这样的话，在devTools上显示的就是类名或函数名，这样也就没必要使用displayName了。
### 136. React的displayName有什么作用？
方便React调试工具devtools中components对组件名称的显示，方便调试。
### 137. 说说你对React的组件命名规范的理解
React组件的命名需要采用大驼峰的命名规范。因为HTML标签的标准是小写字母，中间间隔横杠的形式，React组件都是自定义组件，为了与其区分开来，采用了大驼峰的命名规范；同事Babel在解析JSX时也是通过这个规则，遇到大驼峰的就识别成React组件，小写字母中间横杠的就是html标签。
### 138. 说说你对React的项目结构的理解
见[第23题](23-React如果进行代码拆分？拆分的原则是什么？)
### 139. React16废弃了哪些生命周期？为什么？
废弃的生命周期有：`componentWillMount`、`componentWillReceiveProps`、`componentWillUpdate`
原因：因为React引入了Fiber机制，上述提到的生命周期可能会执行多次，不再是一次执行，失去了意义
### 140. 怎样在React中开启生产模式？
* `npm run build`
* 使用`cross-env`，把`NODE_ENV`改成`production`
* 使用`DefinePlugin`插件，把`NODE_ENV`改成`production`
### 141. React中getInitialState方法的作用是什么？
(废弃API，作用与class组件中的constructor类似，不用关注)
### 142. React中你知道creatClass的原理吗？
(废弃API，不用关注)
### 143. React中验证props的目的是什么？
* 提高代码的健壮性，避免运行时报错，减少不可预见的错误
* 如果使用flow、typescript进行验证，还可以有编辑器的提示
### 144. React中你有使用过getDefaultProps吗？它有什么作用？
(过时API，同组件的静态属性`defaultProps`, 它是组件的静态数据，给组件的props赋予默认值)
### 145. React中你有使用过propType吗？它有什么作用？
用过，用于限定组件各props属性的类型与是否是必填，提高组件的健壮性，避免运行时报错
### 146. React中怎么检验props？
使用组件的静态属性`propTypes`,demo如下
```jsx
import React from 'react';
import PropTypes from 'prop-types';
class Child extends React.Component {
    static propTypes = {
        onClickButton: PropTypes.func.isRequired,
    }
    render () {
        return <button onClick={this.props.onClickButton}>aaa</button>
    }
}
class Demo extends React.Component {
    handleClick = (e) => {
        console.log('CLICK ME');
    }
    render() {
        return <>
            <Child onClickButton={this.handleClick} />
        </>
    }
}
export default Demo;
```

同时，也可以使用静态typescript中泛型的特性，进行校验，如下
```jsx
import React, { FC } from 'react';
/**
 * 声明Props类型
 */
export interface MyComponentProps {
  className?: string;
  style?: React.CSSProperties;
}

export const MyComponent: FC<MyComponentProps> = props => {
  return <div>hello react</div>;
};
```
### 147. React.createClass和extends Component的区别有哪些？
(过时API，不用关注)
### 148. 高阶组件(HOC)有哪些优点和缺点？
优点：
* 提取组件的功能成独立的HOC函数，增强复用性
* 通过让HOC对组件进行层层包装，即可往组件中叠加功能
* 不侵入原组件的代码
缺点
* 如果形成组件嵌套地狱，对调试不方便
* 组件多层嵌套，增加复杂度与理解成本
* ref隔断(React.forwardRef 来解决)
### 149. 给组件设置很多属性时不想一个个去设置有什么办法可以解决这问题呢？
使用对象拓展运算符，demo如下
```jsx
const Demo = () => {
    const props = {
        a: 1,
        b: 2,
        c: 3,
        d: 4,
        e: 5,
    }
    return <TComponent {...props} />
}
```
### 150. React16跟之前的版本生命周期有哪些变化？
* 去掉了`componentWillMount`
* 去掉了`componentWillReceiveProps`,加上了静态方法`getDerivedStateFromProps`代码，用于派生state
* 去掉了`componentWillUpdate`
* 加上了方法`getSnapshotBeforeUpdate`，用于更新后UI DOM的处理
### 151. 怎样实现React组件的记忆？原理是什么？
React组件的记忆，这里我理解的是避免React的无效渲染。
组件渲染的条件之一是对各个props进行浅比较（===），如果更新前后props改变，则会重新渲染。
所以可以从这个方面入手。这主要针对的是引用数据类型的props进行优化，如果更新前后props的地址没有变化则可以避免组件的渲染。可以使用`useCallback`、`useMemo`Hooks，在Redux的使用中，可以引入`reselect`对selector进行管理，则可以对传入的props进行缓存
### 152. 创建React动画有哪些方式？
* 利用`react-transition-group`库
### 153. 为什么建议不要过渡使用Refs？
在JSX中使用Refs可以引用到真实的DOM，也可以引用到class组件的实例。针对前者，React作为MVVM框架，应当让数据推动UI进行更新，尽量不要操作DOM，破坏代码的一致性；针对后者，组件间的练习应当尽量使用数据（state+props+Context）来进行交互，避免直接调用组件的方法，增加调试与维护成本。
### 154. *在React使用高阶组件(HOC)有遇到过哪些问题？如何解决？
* ref隔断：可以在hoc函数中返回一个`React.forwardRef`函数包裹的组件解决
### 155. 在使用React过程中什么时候用高阶组件(HOC)？
* 组件有公共的逻辑可以进行抽离(如提取`ajax`的逻辑)
* 对某一组件有特殊改动，但是又不能通过直接改动组件的代码，可以通过HOC反向继承的方式重写组件
* HOC函数实现一独立的功能，可以作用与所有的组件（如`connect`、`withRouter`）
### 156. 说说React diff的原理是什么？
* 两个不同类型的元素会产生不同的树
    - 对比不同类型的元素时，React对卸载掉整个元素以及其子元素，用新的元素来替代
    - 对比相同类型的元素时，React会保持元素不变，仅更新组件中改变了的属性，处理完成之后，继续递归对子节点进行比较
    - 对比同类型的组件时，组件的实例会保持不变，因为可以保持state的不变，React将更新props，触发组件更新相关的生命周期
* 通过开发者给组件制定`key`属性，来告知渲染哪些子元素在不同的渲染下可以保持不变
    - `key`属性解决的就是列表在进行一对一比较的过程中，新元素树，从中间或者顶部插入的问题；例如，从顶部插入，那么react一对一的往下比较，那么每次比较都是不相同的，react会重建每一个元素；指定了key值之后，react会按照key值进行比较，react就会知道原有的列表只是往下移动了而已，创建的元素只有顶部的一个
### 157. React怎么提高列表渲染的性能？
* 对同级的列表元素指定唯一的`key`值
* 针对大量的列表内容使用虚拟列表
### 158. 使用ES6的class定义的组件不支持mixins了，那用什么可以替代呢？
(minins是过时API，React中hoc、Hooks均可以实现比mixins更优的解决方案)
### 159. 为何说虚拟DOM会提高性能？
浏览器中的DOM操作比较好性能，单纯的运行JS是比较快的。
虚拟DOM本质上就是JavaScript对象，在进行React Diff的时候，是针对js对象进行比较，对比除了新旧虚拟DOM的差异之后，再批量更新到浏览器DOM上。因为是js对象的比较，所以会提高性能;同时因为diff DOM避免了不必要的DOM操作，也提高了性能
### 160. React的性能优化在哪个生命周期？它优化的原理是什么？
`shouldComponentUpdate`.根据业务需求比较新旧props，如果返回false，则不会执行render函数，即阻止了React内部对vnode的比较，这样就减少了性能损耗
### 161. 你知道的React性能优化有哪些方法？
1. 针对class组件，使用`React.PureComponent`
2. 针对函数式组件，使用`React.memo`
3. 合理的使用`shouldComponentUpdate`函数
4. 父组件对需要传递的子组件的props，合理使用`useCallback`的hook
5. 合理的拆分组件：组件粒度更细，避免大组件的渲染
### 162. 举例说明在React中怎么使用样式？
```jsx
// 一般
const Demo = () => <h1 classNames="title">demo</h1>

// classnames
import classnames from 'classnames';
const Demo = () => <h1 classNames={classnames('title')}>demo</h1>

// css-in-js styled-components

import styled from 'styled-components';
const H1 = styled.h1`
    color: #333;
`
const Demo = () => <H1>demo</H1>
```
### 163. React有哪几种方法来处理表单输入？
* 非受控组件的处理: 数据只存在于DOM上，需要获取表单元素的值需要通过Ref拿到DOM元素之后才能获取到值
* 受控组件的处理：数据通过表单元素的value属性与onChange事件，与组件的状态（可以是state也可以是props）形成绑定关系，可以方便的处理数据校验，表单提交的操作
### **164. 什么是浅层渲染？
（React测试方面的知识）
### **165. 你有做过React的单元测试吗？如果有，用的是哪些工具？怎么做的？
(没做过😅)
### **166. 在React中什么是合成事件？有什么用？
在React中定义的事件，都不是原生的DOM事件，而是由react库内部再次封装的合成事件。
部分合成事件在表现上与与原生DOM事件存在差异，比如`onChange`事件的实现；
这样的作用是让react库去抹平事件各个浏览器之间的差异，让开发者用统一的代码，在各个浏览器上实现相同的功能（比如`e.stopPropagation()`，阻止冒泡是由react库内部进行兼容性处理的）
### 167. 使用React写一个todo应用，说说你的思路
1. 拆分组件：应用 + 表单部分（input + button）、列表部分（checkbox + ul>li + delete button）
2. 数据设计：表单的待输入字段 + 列表中的List数据（List item中有content isCompleted）
3. 数据存放：所有的数据都存放到“应用”组件，事件和子数据都通过props往下传递
下面是简单的实现
```jsx
import React, { useState, useRef } from 'react';

function useTodoList(initialList) {
  const id = useRef(0);
  const [list, setList] = useState(initialList);
  // 增加Todo
  const add = text => {
    id.current++;
    setList(
      list.concat({
        id: id.current,
        text,
        isCompleted: false
      })
    );
  };
  // 删除Todo
  const del = id => {
    setList(
      list.filter(e => {
        return e.id !== id;
      })
    );
  };

  // 设置已完成
  const toggleStatus = id => {
    setList(
      list.map(e => {
        if (e.id === id) {
          return {
            ...e,
            isCompleted: !e.isCompleted
          };
        } else {
          return { ...e };
        }
      })
    );
  };

  return [
    list,
    {
      add,
      del,
      toggleStatus
    }
  ];
}

const Form = ({ onAdd }) => {
  const [value, setValue] = useState('');
  const onChange = event => {
    const value = event.target.value;
    setValue(value);
  };
  const onSubmit = () => {
    onAdd(value);
    setValue('');
  };
  return (
    <React.Fragment>
      <input value={value} onChange={onChange} />
      <button onClick={onSubmit}>提交</button>
    </React.Fragment>
  );
};

const TodoList = ({ list, onChange, onDel }) => {
  const setItemStyle = isCompleted => {
    return isCompleted ? { 'text-decoration': 'line-through' } : {};
  };
  return (
    <ul>
      {list.map(e => {
        return (
          <li key={e.id} style={setItemStyle(e.isCompleted)}>
            <input
              type="checkbox"
              checked={e.isCompleted}
              onChange={() => onChange(e.id)}
            />
            <span>{e.text}</span>
            <button onClick={() => onDel(e.id)}>删除</button>
          </li>
        );
      })}
    </ul>
  );
};

export default function App() {
  const [todoList, { add, del, toggleStatus }] = useTodoList([
    { text: 'text', id: 123, isCompleted: false }
  ]);
  return (
    <div>
      <Form onAdd={add} />
      <TodoList list={todoList} onChange={toggleStatus} onDel={del} />
    </div>
  );
}

```
### **168. React16的reconciliation和commit分别是什么？
### 169. React的函数式组件有没有生命周期？
同[第97题](97-函数式组件有没有生命周期？为什么？)
### 170. useState和this.state的区别是什么？
* useState用在函数式组件中，是React提供的一个Hook，用于管理函数式组件的状态；this.state是class组件中的状态管理
* this.state的更新使用`this.setState`的方式；useState中状态的更新是useState返回的数组的第二个元素的函数进行更新
* 获取修改后的值：this.setState可以用第二个参数函数获取，useState可以通过useEffect副作用执行函数获取
* Hook中的`setXXX`的方式设置值被进行完全的覆盖，而`this.setState`是对`this.state`对象进行浅层的合并
### 171. 请说说什么是useImperativeHandle？
在函数式组件中使用，用于自定义ref的返回值，返回值是一个对象，可以自定义对象的值是函数的内部变量、方法、Node节点等。
```jsx
/**
 * useImperativeHandle Demo
 */
import React, { useImperativeHandle, useRef } from 'react';

const FComponent = React.forwardRef((props, ref) => {
  const inputRef = useRef(null);
  const divRef = useRef(null);
  useImperativeHandle(ref, () => {
    return {
      focus,
      div: divRef.current
    };
  });
  const focus = () => {
    inputRef.current.focus();
  };
  return (
    <>
      <input ref={inputRef} />
      <div ref={divRef}>some text</div>
    </>
  );
});

export default function App() {
  const fcRef = useRef(null);
  const handleFocus = () => {
    fcRef.current.focus();
  };
  const handleGetDiv = () => {
    console.log(fcRef.current.div);
  };
  return (
    <div>
      <FComponent ref={fcRef} />
      <button onClick={handleFocus}>FOCUS</button>
      <button onClick={handleGetDiv}>GET DIV</button>
    </div>
  );
}

```
### 172. 请说说什么是useReducer？
useReducer是react内部的hook，用于复杂的数据状态管理，它的参数是一个reducer函数，返回值数组的第一个元素是state，第二个元素是dispatch函数。
上面[第167题](167-使用React写一个todo应用，说说你的思路)用useReducer的实现如下：
```jsx
import React, { useReducer, useState } from 'react';
let _id = 0;
const ACTION_TYPES = {
  ADD: 'ACTION_ADD',
  DEL: 'ACTION_DEL',
  TOGGLE_STATUS: 'ACTION_TOGGLE_STATUS'
};

const reducer = (state, action) => {
  switch (action.type) {
    case ACTION_TYPES.ADD:
      const { text } = action;
      _id = _id + 1;
      return state.concat({
        id: _id,
        text,
        isCompleted: false
      });
    case ACTION_TYPES.DEL:
      return state.filter(e => {
        return e.id !== action.id;
      });
    case ACTION_TYPES.TOGGLE_STATUS:
      return state.map(e => {
        if (e.id === action.id) {
          return {
            ...e,
            isCompleted: !e.isCompleted
          };
        } else {
          return { ...e };
        }
      });
    default:
      return state;
  }
};

const Form = ({ dispatch }) => {
  const [value, setValue] = useState('');
  const onChange = event => {
    const value = event.target.value;
    setValue(value);
  };
  const onSubmit = () => {
    dispatch({
      type: ACTION_TYPES.ADD,
      text: value
    });
    setValue('');
  };
  return (
    <React.Fragment>
      <input value={value} onChange={onChange} />
      <button onClick={onSubmit}>提交</button>
    </React.Fragment>
  );
};

const TodoList = ({ list, dispatch }) => {
  const setItemStyle = isCompleted => {
    return isCompleted ? { textDecoration: 'line-through' } : {};
  };
  return (
    <ul>
      {list.map(e => {
        return (
          <li key={e.id} style={setItemStyle(e.isCompleted)}>
            <input
              type="checkbox"
              checked={e.isCompleted}
              onChange={() =>
                dispatch({
                  type: ACTION_TYPES.TOGGLE_STATUS,
                  id: e.id
                })
              }
            />
            <span>{e.text}</span>
            <button
              onClick={() =>
                dispatch({
                  type: ACTION_TYPES.DEL,
                  id: e.id
                })
              }
            >
              删除
            </button>
          </li>
        );
      })}
    </ul>
  );
};

export default function App() {
  const [todoList, dispatch] = useReducer(reducer, []);
  return (
    <div>
      <Form dispatch={dispatch} />
      <TodoList list={todoList} dispatch={dispatch} />
    </div>
  );
}
```
### 173. 请说说什么是useRef？
useRef 接受一个参数，为 ref 的初始值。useRef 保存的变量不会随着每次数据的变化重新生成，而是保持在我们最后一次赋值时的状态.
### 174. 请说说什么是useEffect？
useEffect是副作用函数，第一个参数是函数，第二个参数是依赖的数据数组，当依赖数组中的数据变化时，触发第一个参数函数的执行。有以下的几种使用方式
1. 模拟`componentDidMount`：常用作初始ajax请求，获取ref节点等操作
```jsx
import { useEffect } from 'react';
function Demo() {
  useEffect(() => {
    // ...
  }, []);
  return <h1>Demo</h1>;
}
```
2. 模拟`componentWillUnMount`: 用作取消订阅、清除定时器，需要随着组件的卸载而从内存中移除的操作放在这里
```jsx
import { useEffect } from 'react';
function Demo() {
  useEffect(() => {
    return () => {
      // ...
    }
  }, []);
  return <h1>Demo</h1>;
}
```
3. 监听state值的变化，用最新的值进行副作用的操作
```jsx
import { useState, useEffect } from 'react';
const [userId, setUserId] = useState(null);
function Demo() {
  useEffect(() => {
    if (userId) {
      // ...
    }
  }, [userId]);
  return <h1>Demo</h1>;
}
```
### 175. 举例说明useState
```jsx
import { useState } from 'react';
const [count, setCount] = useState(0);
function Demo() {
  const doAdd1 = () => {
    setCount(prevCount + 1);
  };
  const doAdd2 = () => {
    setCount(prevCount => prevCount + 1);
  };
  return <>
    <h1>{count}</h1>
    <button onClick={doAdd1}>Add-方式1</button>
    <button onClick={doAdd2}>Add-方式2</button>
  </>;
}
```
### 176. 请说说什么是useState？为什么要使用useState？
`useState`是React原生的Hook，它接受一个参数，这个参数可以是对象或者普通的基本数据类型的值，也可以是一个有返回值的函数，`useState`函数返回一个数组，数组的第一个元素是`useState`接受的值或函数的返回值，第二个元素是设置这个值的方法；
在Hooks之前，函数式组件都是无状态的组件，引入了useState这个hook之后，让函数式组件也能够管理自己的状态
### 177. 请描述下你对React的新特性Hooks的理解？它有哪些应用场景？
理解：
1. Hooks赋予了函数式组件更强大的能力，能让让函数式组件有状态，可以做以前class组件才能做到的更有逻辑的业务
2. Hooks方便了业务的抽离，相比与以前HOC的代码不容易理解，组件容易形成嵌套地狱等问题，Hooks不仅能够针对单一的逻辑进行抽离，同时也解决了HOC的种种问题
3. Hooks就是对数据管理的抽离，以前通过`utils`的纯JS代码，现在也能够通过hooks后的封装，实现“响应式”
应用：
1. 对副作用的处理,用于模拟生命周期，对数据状态的监听，这个主要用到了`useEffect`hook，详情可以看[第174题](请说说什么是useEffect)
2. `useMemo``useCallback`对变量、函数的缓存，结合`React.PureComponent``React.memo`可以避免不需要的render
3. `useRef`声明一个状态值，不随着组件函数的执行而重新赋值，只以最后一次赋值为准
4. useContext是Context API的hooks版本，用于在组件中的context进行取值
4. 常用的第三方库的hooks: 比如`react-router`的`useLocation` `useParams` `useRouteMatch` `useHistory`，`react-redux`的`useDispatch` `useSelector`等
5. 单一业务逻辑抽离成自定义hooks
### 178. 说说你对Error Boundaries的理解
错误边界是React16新推出的一种错误处理的方式，在v16之前，React的抛错会导致页面显示的错误，v16修改这种方式成了组件如果产生了错误，那么从这个组件到根组件都会因为错误而崩溃掉，不让用户看到错误组件的信息，如果将产生错误的组件用错误边界组件包裹起来，那么错误的“冒泡”就会在错误边界这里被阻止，依据自定义的错误边界组件可以给用户抛出自定义的错误信息。
React的设计者认为不给用户显示信息，比给用户显示错误的信息要更加的合理，所以采用了这样的一种方案。
错误边界的使用示例可以查看[第26题](26-React中在哪捕获错误？)
### **179. 说说你对Fiber架构的理解
[这里](https://www.yuque.com/docs/share/73874ed8-2ea7-4c75-bccb-0fb857b45787)
### 180. 说说你是怎么理解React的业务组件和技术组件的？
业务组件即代码逻辑紧贴业务，如ajax请求，一些特定的需求特性等；
技术组件即使比较通用性的组件，如对loading的处理、ajax的hoc等
### 181. 为什么建议setState的第一个参数是callback而不是一个对象呢?
因为setState是一个异步的过程，使用setState设置了state中的某一个属性property的值之后，马上取this.state.property并不会得到刚刚设置的值。如果setState第一个参数是callback的话，callback的第一个参数是上一次setState的值，这样每次setState内部的赋值关系都是异步的，都会依赖上一次的求值，使赋值更加可靠、稳定.通过下面的Demo代码可以看出两者的差异
```jsx
import React from 'react';
class Demo extends React.Component {
    state = {
        count: 0,
    }
    componentDidMount () {
        for (var i = 0; i < 10; i++) {
            // render内展示： DEMO: 1
            // this.setState({
            //     count: this.state.count + 1,
            // })
            // render内展示： DEMO: 10
            this.setState(prevState => ({
                count: prevState.count + 1
            }))
        }
    }
    render() {
        return <>
            <h1>DEMO: { this.state.count }</h1>
        </>
    }
}
export default Demo;
```
### 182. 展示组件和容器组件有什么区别？
* 展示组件
- 无状态组件，组件组件本身不维护自己的state，接受props进行展示与简单的事件处理;
- 内部可以包含展示组件和容器组件，通常会包含一些自己的DOM标记和样式(style)
- 对应用程序的其他部分没有依赖关系，例如Reducer操作或store、路由等
- 不用关心数据是怎么加载和变动的。
* 容器组件
- 关注应用的是如何工作的，一般是有状态的，也可以接受props，内部包含了各种组件与相关的逻辑;
- 内部可以包含容器组件和展示组件，但通常没有任何自己的DOM标记，除了一些包装divs，并且从不具有任何样式
- 提供数据和行为给其他的展示组件或容器组件
### 183. Mern和Yeoman脚手架有什么区别？
* MERN是一种脚手架工具，可使用Mongo，Express，React和Nodejs轻松构建同构应用
* Yeoman也是脚手架工具，它是基于Node.js运行的，生成的技术栈与开发语言无法，它提供的是文件操作等方法，来自定义生成工程结构与代码文件
### 184. 你有在项目中使用过Yeoman脚手架吗？
(没有😭)
### **185. 你有在项目中使用过Mern脚手架吗？
(没有😭)
### 186. shouldComponentUpdate方法是做什么的？
shouldComponentUpdate是React组件的生命周期，在组件更新后触发，该方法继承子React.Component.方法的作用是可以在内部比较更新前后的state和props的值，手动控制当前组件是否需要执行render函数，函数签命是`shouldComponentUpdate(nextProps, nextState)`
### 187. 怎样在React中使用innerHTML？
使用`dangerouslySetInnerHTML`属性,该属性传入一个对象，对象中`__html`属性的值即时innerHTML的富文本代码
### 188. 你有写过React的中间件插件吗？
(React中Redux有中间件的概念，没有听过React的中间件，这道题没有理解，这道题和以下的两道题都按照Redux中间件进行回答)
简单的实现Redux的logger中间件，代码如下:
```JavaScript
const logger = store => next => action => {
    console.group(action.type)
    console.info('dispatching', action)
    let result = next(action);
    console.log('next state', store.getState())
    console.groupEnd()
    return result;
};
export default logger;
```
### **189. React的中间件机制是怎么样的？这种机制有什么作用？
### 190. React中你用过哪些第三方的中间件？
> 这里理解的是Redux的中间件，React本身好像并没有什么中间件的概念
* redux-thunk: Redux的异步处理方案,actionCreator中可以返回一个函数（即可以dispatch一个function），函数内在写异步的代码
* redux-saga: Redux的异步处理方案，没有破坏redux中dispatch一个plain object的原则，内部利用generator的方式实现异步的方式, 同时还提供了watch、put、call等工具方法更好的实现异步的调用
### **191. 不用脚手架，你会手动搭建React项目吗？
(需要实践，实践后把过程贴出来)
### 192. 请说说React中Portal是什么？
核心API是`ReactDOM.createPortal(ReactNode, DOMContainer)`，返回值是一个`ReactNode`。返回的ReactNode需要在组件的render函数(函数式组件的返回值)中进行渲染，createPortal接受的ReactNode将会挂载到DOMContainer这个元素上，和render函数中的树结构不一致；但是，从React的角度看，createPortal的节点依然保持render函数的树形结构，Props、Context等特性可以正常的通过负责组件传递到portal组件中。
### 193. React中修改prop引发的生命周期有哪几个？
1. `static getDerivedStateFromProps`
2. `shouldComponentUpdate`
3. `render`
4. `getSnapshotBeforeUpdate`
5. `componentDidUpdate`
### 194. React多个setState调用的原理是什么？
同步调用多个setState，React并不会连续多次的进行更新操作，而是会将同步中的多个setState操作合成一个，只执行一次render操作
### 195. React中调用setState会更新的生命周期有哪几个？
(>=16.4之后，与props的更新一致， <16.4的版本没有`static getDerivedStateFromProps`)
1. `static getDerivedStateFromProps`
2. `shouldComponentUpdate`
3. `render`
4. `getSnapshotBeforeUpdate`
5. `componentDidUpdate`
### 196. React中setState的第二个参数作用是什么呢？
第二个参数是一个callback函数，用于setState设置state的属性值成功之后的回调，此时调用`this.state.property`可以取到刚刚设置的最新的值
### 197. React中的setState是同步还是异步的呢？为什么state并不一定会同步更新？
由React控制的事件处理程序，以及生命周期函数调用setState不会同步更新state 。
React控制之外的事件中调用setState是同步更新的。比如原生js绑定的事件，setTimeout/setInterval等。
从代码的本质上看它们都是同步的，只是第一种情况是有批处理机制，当跟组件didMount后，再进行统一的合并更新。
这样设计的原因是可能在同步的代码中可能存在连续的多个setSate操作，react会对他们进行智能的合并，直到执行到了最后一个setState，React才回智能的合并state的，并异步的设置state的值，后续判断是否进行render操作.如果同步的每次setState都去走一遍reconciler & commit操作，则太耗性能了。
第二种情况中，原生事件没有触发批处理机制，异步代码中，同步代码中的批处理已经执行晚了，没有了flag来标记应当批处理，所以就直接执行了
### 198. React中的setState批量更新的过程是什么？
在React控制的事件和生命周期的代码中的多个setState执行完成了，react对涉及到的state进行批量的合并，再异步的对state中的属性进行赋值，后续根据shouldComponent的返回值情况决定是否re-render.
### **199. React中的setState执行机制是什么呢？
只要执行了setState方法，必然会引起组件的更新流程。
[参考文章](https://juejin.cn/post/6844903781813993486)
### 200. 在React中遍历的方法有哪些？它们有什么区别呢？
有`React.Children.map`和 `React.Children.forEach`这两个方法，他们的参数都是在组件中接受`props.children`这个ReactNode作为参数，然后进行遍历。专门提供这两个遍历方法的目的是`props.children`可能是字符串、null、数组，用`React.Children.map`可以抹平这些数据类型的差异，使之都能进行循环，并返回合理的值
### 201. 请说说你对React的render方法的理解
render是class组件中必须被重载的方法，组件执行render方法的条件可以看[第6题](4-说说你对React的渲染原理的理解)。
render方法的执行不一定会导致DOM的重新渲染，DOM的重新渲染的条件是render前后的DOM diff比较的结构不一致。
### 202. props.children.map和js的map有什么区别？为什么优先选择React的？
专门提供`props.children.map`这个遍历方法的目的是`props.children`可能是字符串、null、数组，用`React.Children.map`可以抹平这些数据类型的差异，使之都能进行循环，并返回合理的值.
### 203. 有用过React的严格模式吗？
重复问题可以看[第22题](22-React的严格模式有什么好处？)
### **204. React中的setState和replaceState的区别是什么？
(没听过replaceState)
### **205. React中的setState缺点是什么呢？
* 如果需要属性值是引用数据类型，则需要保持数据的不可变形，即需要将整个对象(数组)复制一份
### 206. 有用过React的Fragment吗？它的运用场景是什么？
参考[第70题](你有用过React.Fragment吗？说说它有什么用途？)
### 207. React组件间共享数据方法有哪些？
* props:只能通过父组件传到子组件，可以通过穿函数的props把子组件的数据通过回调函数传回来
* Context：Context.Provider下的组件可以共享Provider提供的数据，可以通过在Provider中提供操作Provider value的方法，实现在子组件中修改Provider的数据
### **208. React的状态提升是什么？使用场景有哪些？
### 209. 简单描述下你有做过哪些React项目？
XX管理系统：脚手架使用craco，可以避免`create-react-app`需要eject才能自定义webpack配置的问题；数据管理使用了dvajs，它是对Redux-Saga的封装，可以对异步的数据管理进行处理；路由上使用了react-router-dom
### 210. 在构造函数中调用super(props)的目的是什么？
这是ES6的语法。class组件继承自React.Component，super(props)之后，作为子类的组件可以拿到父类的this，使之可以调用父类的属性和方法，如`this.setState``this.props`等等
### 211. 你是如何学习React的？
看文档，看相关生态文档、看Github项目上相关项目的代码实现、看面试题、实践
### **212. 从旧版本的React升级到新版本的React有做过吗？有遇到过什么坑？
(没做过)
### 213. 你用过React版本有哪些？
只用过React16.8+
### 214. 有用过React的服务端渲染吗？怎么做的？
(没做过)
### 215. React的mixins有什么作用？适用于什么场景？
(老旧API，不需关心，现在已经被HOC和hooks代替了)
### 216. React怎么拿到组件对应的DOM元素？
通过ref，ref在class组件和函数式组件中有各自的使用方式。
* 在Class组件中
```jsx
import React from 'react';
class CComponent extends React.Component {
  refDiv = React.createRef();
  componentDidMount () {
    console.log(this.refDiv.current)
  }
  render () {
    return <div>
      <div className="test" ref={this.refDiv}>demo</div>
    </div>
  }
}

export default CComponent;

// 老旧语法
import React from 'react';
class CComponent extends React.Component {
  // refDiv = React.createRef();
  componentDidMount () {
    console.log(this.refDiv)
  }
  render () {
    return <div>
      <div className="test" ref={ref => this.refDiv}>demo</div>
    </div>
  }
}

export default CComponent;
```
* 在函数式组件中
```jsx
import React, { useRef, useEffect } from 'react';

const FComponent = () => {
  const refDiv = useRef();
  useEffect(() => {
    console.log(refDiv.current);
  }, []);
  return <div>
    <div className="demo" ref={refDiv}>test</div>
  </div>
}
export default FComponent;
```
### 217. 请描述下事件在React中的处理方式是什么？
* 合成事件
* 事件委托
* React17中事件的变化
### 218. JSX和HTML有什么区别？
* JSX是类HTML的语法，它是嵌在javascript，JSX标签返回的是一个JSX对象
* HTML当时是无法写在JS中的，所以才有了JSX，因为JSX是嵌入到JavaScript当中的，所以要避免JS保留字的使用，如for要写成forHTML，class需要写成className
* JSX没有自己的模版语法糖，因为可以用过原生的js表达式操作它，如只需要把表达式写在`{}`中即可，其中也可以写注释
### 219. React的书写规范有哪些？
* React元素必须使用大驼峰命名的方式
* 组件尽量命名，方便进行调试
* JSX尽量用小括号包起来，方便调试
### 220. create-react-app创建新运用怎么解决卡的问题？
(没听cra会导致有这个问题)
### 221. 使用React的方式有哪几种？
没看懂是什么意思。
如果是脚手架的选择，目前有CRA、Craco、umijs；
如果是打包工具：有最主流的webpack、Rullup、vitejs可供选择
### 222. 说说你对reader的context的理解
这个题想问的应用是render函数的上下文，而非`React.createContext`的那个Context。
不管是class组件还是函数式组件，React的组件本质上都是函数，从根组件到下面大大小小的子组件，React组件树即构成了函数的嵌套，React的渲染/更新即时从顶层函数执行，一步一步的递归执行内部嵌套函数的过程，这是一个原生的JS执行栈执行的过程。
### 223. 同时引用这三个库React.js、React-dom.js和babel.js它们都有什么作用？
* React.js: React中的组件(`Component`)、Context、hooks等核心Api，还有虚拟DOM的比较、Fiber的算法实现等
* React-dom.js 与web浏览器DOM相关的API，比如虚拟DOM的挂载，DOM的更新，Portal等
* babel.js ES6+代码的转义
### 224. 你知道Virtual DOM的工作原理吗？
(Fiber相关)
### 225. 你阅读过React的源码吗？简要说下它的执行流程
(没读过)
### 226. React中怎样阻止组件渲染？
* class组件
- 使用shouldComponentUpdate生命周期，`return false`
- 继承`React.PureComponent`只要prop没有改变(浅比较)，就不会执行render函数
* 函数式组件
- 使用React.memo包裹组件函数，props没有改变就不会执行函数
- 使用React.memo包裹函数的同时传入比较函数作为第二个参数，`return true`，组件函数就不会执行
### 227. React非兄弟组件如何通信？
* props可以从父组件传入到子组件进行通信
* props可以从父组件传入一个callback function到子组件，通过参数回调的形式实现子组件传参到父组件（如子组件是受控组件的实现）
* 利益Context API，可以实现Provider下任意组件之间的通信
### 228. React兄弟组件如何通信？
* 把兄弟组件的数据都放在共有的父组件中，数据通过父组件往下传递，改变数据的方法也作为props通过父组件往下传递
* 利益Context API，可以实现Provider下任意组件之间的通信
### 229. React非父子组件如何通信？
* 把所有组件的数据都放在共有的顶层组件中，数据通过顶层组件往下传递，改变数据的方法也作为props通过顶层组件往下传递
* 利益Context API，可以实现Provider下任意组件之间的通信
### 230. React父子组件如何通信？
* props可以从父组件传入到子组件进行通信
* props可以从父组件传入一个callback function到子组件，通过参数回调的形式实现子组件传参到父组件（如子组件是受控组件的实现）
* 利益Context API，可以实现Provider下任意组件之间的通信
### 231. React组件间的通信有哪些？
* Props
* Context
### 232. 类组件和函数式组件有什么区别？
* 写法不同，类组件是类是一个构造函数，在react内部需要被实例化；函数式组件是一个普通的函数，不需要被实例化，没有实例
* 类组件有自己的生命周期，函数式组件只能通过`useEffect`hook实现类生命周期的概念
* ref可以直接引用类组件，并调用到类组件的实例；ref不能直接引用函数式组件，需要通过`React.forwardRef`进行转发
* 函数式组件可以使用Hooks，类组件不能
* 错误边界只能通过类组件才能实现
### 233. React自定义组件你写过吗？说说看都写过哪些？
* 利用svg技术实现的手写画板
* 结合公司业务实现的自定义学段学科年级选择器
* 数据统计的统一封装(echarts等)
### 234. React组件的state和props两者有什么区别？
* state是本组件维护的状态，props是从父组件接受而来的数据，他们都是可响应的，即数据的变化(引用类型通过浅比较)可以引用render函数或者函数式组件的重新执行
* 两种数据都是immutable的，state可以通过setState进行异步的改变，props不可更改
### 235. React有几种构建组件的方式？可以写出来吗？
（没明白这道题的意思）
### 236. React中遍历时为什么不用索引作为唯一的key值？
key值的目的在Diff DOM的时候根据render前后的唯一key值快速的对树进行比较，保证key值得唯一性，如在进行中间插入或者最顶上插入时，算法会很明显得知道这是一个插入动作，然后让后续得node往后移位，如果是以索引index作为key值，那么在进入上述的操作后，插入后面的元素key值完全变了，算法不能准确的定位到树中node的位置，只能从插入起，后续的全部重新生成，影响性能
### 237. React中的key有什么作用？
通过开发者给组件制定`key`属性，来告知渲染哪些子元素在不同的渲染下可以保持不变
`key`属性解决的就是列表在进行一对一比较的过程中，新元素树，从中间或者顶部插入的问题；例如，从顶部插入，那么react一对一的往下比较，那么每次比较都是不相同的，react会重建每一个元素；指定了key值之后，react会按照key值进行比较，react就会知道原有的列表只是往下移动了而已，创建的元素只有顶部的一个
### 238. React中除了在构造函数中绑定this,还有别的方式吗？
* 可以使用public class field 语法，demo如下:
```jsx
import React from 'react';
export class CComponent extends React.Component {
    onSubmit = () => {
        // code 
    }
    render () {
        <button onClick={this.onSubmit}>submit</button>        
    }
}
```
* 在使用的时候进行`bind`绑定
### 239. 在React中页面重新加载时怎样保留数据？
每次页面加载完成后讲需要保留的数据保存到sessionStorage中，页面重新加载时再从sessionStorage中获取，如果取不到，再用默认值
### **240. 请描述下React的事件机制
* 事件合成
* 事件委托
* React17事件的变化
### 241. 怎样在React中创建一个事件？
这道题中的“创建”我理解的是，如果在React中声明一个事件，而非原生的`CustomEvent`自定义事件的创建与触发，因为这在React中存在的意义不大。
React中事件是直接绑定在标签上的，如果是HTML原生标签，则可以用`on`前缀，后加事件类型的形式进行声明，用的是小驼峰的命名，如`<div onClick={this.handleClick}></div>`;
需要注意的是，在类组件中的事件函数要注意`this`的绑定，可以在`JSX`中绑定，可以在`constructor`中绑定，可以用`public class field`的语法让事件中的`this`指向组件实例，推荐后两种方式.
### 242. 在React中无状态组件有什么运用场景？
适用于逻辑简单的纯展示的场景，如资料卡片等
### 243. 描述下在React中无状态组件和有状态组件的区别是什么？
状态即是React中的state，有状态组件就是React组件中有state在维护的组件，在Hooks之前，一般class组件都是有状态组件(如果只用props，也是无状态组件)，Functional组件都是无状态组件（hooks之前）；无状态组件适用于简单的纯展示，它们需要被有状态组件引用，有状态组件逻辑相对复杂，有自己的state，需要处理副作用等反馈到state状态上.
### 244. 写一个React的高阶组件(HOC)并说明你对它的理解
HOC是指接受一个组件作为参数且返回另一个组件的函数，高阶组件内部本身有一个组件的逻辑，所以它维护了自身的一套数据状态，在hoc内部的组件中，可以在render函数中控制组件的props，控制什么组件进行return，这就是渲染劫持。利用这个能力，可以通过HOC封装出一些使用于组件的通过化逻辑，比如下面的loading，还有ajax请求，像react-router的widthRouter，react-redux的connect，这些高阶组件的实现本质上都是上述说的这个逻辑.
但是高阶函数也有自身的缺点：
1. ref无法直接传递：可以在hoc中处理，通过`React.forwardRef`进行转发
2. 容易形成嵌套地狱，调试不直观
3. 组件层层嵌套的逻辑不易于理解
4. 不同的高阶组件可能会有嵌套顺序的要求，比较容易出错：如withRouter和connect一起用一定把withRouter写外面
```JSX
// app.jsx
import React from 'react';
import Child from './Child.jsx';
import HocLoading from './hocLoading';

const LoadingChild = HocLoading(Child);

class Demo extends React.Component {

    state = {
        loading: true,
    }

    componentDidMount() {
        setTimeout(() => {
            this.setState({
                loading: false,
            })
        }, 2000)
    }

    render() {
        return <LoadingChild loading={this.state.loading}>parent content</LoadingChild>
    }
}

export default Demo;

// hocLoading.js
import React from 'react';
export default function HocLoading (WrappedComponent) {
    class ComponentLoading extends React.Component {
        render () {
            const { loading } = this.props;
            return (
                loading
                ? <h4>loading...</h4>
                : <WrappedComponent {...this.props} />
            )
        }
    }
    return ComponentLoading;
}

// Child.jsx
import React from 'react';

class Child extends React.Component {
    render() {
        return <>
            <h1>This is Content!</h1>
            <div>{this.props.children}</div>
        </>
    }
}

export default Child;
```
### 245. React中可以在render访问refs吗？为什么？
不能，render阶段，ref还没有真实的挂载到DOM节点上，至少要在`componentDidMount`阶段进行访问
### 246. React中refs的作用是什么？有哪些应用场景？
* 作用
- 对类组件、DOM节点的引用
- 在函数式组件中声明一个不会随着函数组件的执行而变化的变量，这个变量总是以最后一次赋值为准
* 应用场景
- class组件中通过ref拿到子class组件的实例
- ref绑定到原生节点上拿到原生节点的DOM
- 通过在函数式子组件中声明`useImperativeHandle`hook，可以拿到从函数式的子组件中拿到子组件像抛出的任意属性和方法
### 247. 请描述你对纯函数的理解？
* 给定了输入的参数，则函数抛出的数据就是确定的
* 函数内没有副作用(定时器，ajax等)
### 248. 受控组件和非受控组件有什么区别？
受控组件于非受控组件是针对表单元素来讲的，React是MVVM的库，数据(Model)的更新会通过ViewModel反映到视图(View)上，然而视图的更新并不会自动的映射到数据中，按照库默认的处理方式，则所有的表单就控件都是非受控组件，如果手动指定了表单元素的onChange事件，在事件函数中手动给相应的数据进行赋值，且这个值在绑定在表单的`value`/`checked`属性上，则这个表单元素就是受控组件。
简单的说受控于非受控的区别就是数据于视图之间是否互相绑定。按照React的思想来说，一般我们在进行表单处理的时候都需要把组件处理成受控的，这样在做复杂的表单校验、提交数据的获取都会方便很多.
### 249. React中什么是非控组件？
表单元素的`value`/`checked`值、`onChange`事件没有和组件的state存在绑定关联关系，如果需要获取表单元素的值，需要使用`ref`获取DOM，然后得到元素的值.demo如下：
```jsx
import React from 'react';
class FormDemo extends React.Component {
    refUsername = React.createRef();
    refPassword = React.createRef();
    onSubmit = () => {
        const [
            username,
            password,
        ] = [
            this.refUsername.current.value,
            this.refPassword.current.value,
        ];
        console.log({
            username,
            password,
        })
    }
    render () {
        return (
            <div className="form">
                <p>
                    <span>用户名:</span>
                    <input ref={this.refUsername} name="username" />
                </p>
                <p>
                    <span>密码:</span>
                    <input ref={this.refPassword} name="password" />
                </p>
                <button onClick={this.onSubmit}>登录</button>
            </div>
        )
    }
}
export default FormDemo;
```
### 250. React中什么是受控组件？
表单元素的`value`/`checked`值与state绑定，同时表单元素的onChange可以动态的改变对应的state的值，即数据的变更可以更新视图，用户操作视图的更新也可以触发state数据的更新.
```jsx
import React from 'react';
class FormDemo extends React.Component {
    state = {
        username: '',
        password: ''
    }
    handleChange = event => {
        const { name, value } = event.target;
        this.setState({
            [name]: value,
        });
    }
    onSubmit = () => {
        const { username, password } = this.state;
        console.log({
            username,
            password,
        })
    }
    render () {
        const { username, password } = this.state;
        return (
            <div className="form">
                <p>
                    <span>用户名:</span>
                    <input value={username} name="username" onChange={this.handleChange} />
                </p>
                <p>
                    <span>密码:</span>
                    <input value={password} name="password" onChange={this.handleChange} />
                </p>
                <button onClick={this.onSubmit}>登录</button>
            </div>
        )
    }
}
export default FormDemo;
```
### 251. React中发起网络请求应该在哪个生命周期中进行？为什么？
需要在`componentDidMount`中进行。网络请求应当尽早的进行，`componentDidMount`是在`render`后执行的生命周期方法，这里可以进行安全的`setState`，如果组件的更新需要触发网络请求的话，可以在`componentDidUpdate`中进行.
### 252. 说说React的生命周期有哪些？
这里说的都是React v16.4及之后的声明周期。
* 挂载
- constructor
- static getDerivedStateFromProps
- render 
- componentDidMount
* 更新
- static getDerivedStateFromProps
- shouldComponentUpdate
- render 
- getSnapshotBeforeUpdate
- componentDidUpdate
* 卸载
- componentWillUnmount
### 253. 说说你对“在React中，一切都是组件”的理解
* React采用组件化的思想，最小的组件单位就是原生HTML元素，采用JSX的语法声明组件的调用
* React的虚拟DOM，就是一个大的组件树，从父组件层到子组件，在render函数中层层堆叠
* 从react-router v4开始，路由本身也是组件
* 各个库提供的hoc返回的也是组件，如withRouter、connect
### 254. 写React你是用es6还是es5的语法？有什么区别？
（老旧问题，没有考察的必要性）
### 255. 浏览器为什么无法直接JSX？怎么解决呢？
因为浏览器只能解决原生的JS代码，jsx不属于原生的js，它是类似于html的语法，然后转化成一个js对象。
这个转化的过程就是通过Babel的`@babel/plugin-transform-react-jsx`插件实现的。
### **256. 在使用React过程中你都踩过哪些坑？你是怎么填坑的？
* 组件不像Vue一样有(beforeRouteLeave, beforeRouteEnter)路由守卫
* Hooks中的闭包渲染问题
```JSX
import { useState } from 'react';

const Demo = () => {
    const [count, setCount] = useState(0);
    const onAdd = () => {
        setTimeout(() => {
            // setCount(count + 1);
            // 解决方案：使用回调的形式，这样setXXX必须强制依赖上一次的求值
            setCount(prevCount => prevCount + 1);
        }, 1000);
    };
    return <>
        <h2>{count}</h2>
        <button onClick={onAdd}>ADD</button>
    </>
}
export default Demo;
```
### 257. 说说你喜欢React的原因是什么？它有什么优缺点？
* 原因：
- All in JS，没有过多的语法糖、API等，核心思想就是props、state、生命周期、hooks，使用JS的技能就能解决和解释一切现象
* 优点：
- 使用了虚拟DOM，高性能
- JSX: 类似HTML的语法方便的构建UI
- MVVM，数据模型可以自动的响应在UI上
- 活跃的生态圈
* 缺点:
- React非框架，不同的开发人员有不同的写法，对开发人员要求较高
### 258. 如何解决引用类型在pureComponent下修改值的时候，页面不渲染的问题？
使用immutable对象
```jsx
import React from 'react';

class PComponent extends React.PureComponent {
    render() {
        const { name, age } = this.props.person;
        return <>
            <p>NAME: {name}</p>
            <p>AGE: {age}</p>
        </>
    }
}

class Demo extends React.Component {
    state = {
        person: {
            name: 'vk',
            age: 1,
        }
    }
    doChange = () => {
        const { person } = this.state;
        // 这种情况下，person的引用地址没有变化，diff前后浅比较相等，所以并没有引起PComponent变化
        // 如果PComponent是继承自React.Component,因为父组件的render导致了子组件也render了，是会产生变化的
        // person.name = 'vk2';
        // this.setState({ person })
        // 正确的写法
        this.setState({
            person: {
                ...person,
                name: 'vk2',
            }
        });
    }
    render() {
        const { person } = this.state;
        return <>
            <button onClick={this.doChange}>Change</button>
            <PComponent person={person} />
        </>
    }
}

export default Demo;
```
### 259. createElement与cloneElement两者有什么区别？
* `createElement`是创建一个React元素，`jsx`即是这个函数的语法糖；它的函数签命是`React.createElement(type, [props], [...children])`
* `cloneElement`是拷贝一个React元素，可选择在修改它的props后，再返回一个新的React元素；它的函数签命是`React.cloneElement(element, [props], [...children]): ReactElement`,这个函数常结合`React.Children.map`一起使用，修改`props.children`的props，来进行一些额外的操作，如下
```jsx
/** 以下代码实现点击子组件是，先执行父组件的onClick方法，再执行子组件的onClick方法
 */
import React from 'react';

function Tab(props) {
    const _onClick = (onclick) => {
        props.onClick();
        onclick();
    };
    return <ul onClick={props.onClick}>
        {
            React.Children.map(props.children, child => {
                const childProps = child.props;
                const newProps = {
                    ...childProps,
                    onClick: _onClick.bind(null, childProps.onClick)
                }
                return React.cloneElement(child, newProps);
            })
        }
    </ul>
}

function TabItem(props) {
    const onClick = event => {
        event.stopPropagation();
        props.onClick(event);
    }
    return <li onClick={onClick}>{props.children}</li>
}

function Demo() {
    const onClickTab = () => {
        console.log('onClickTab')
    }
    const onClickItem = () => {
        console.log('onClickItem')
    }
    return <Tab onClick={onClickTab}>
        <TabItem onClick={onClickItem}>111</TabItem>
        <TabItem onClick={onClickItem}>222</TabItem>
        <TabItem onClick={onClickItem}>333</TabItem>
    </Tab>
}

export default Demo;
```
### 260. 解释下React中Element 和Component两者的区别是什么？
同[第129题](129-在React中组件和元素有什么区别)
### 261. 解释下React中component和pureComponent两者的区别是什么？
PureComponent继承自Component,他们的区别体现在state和render的响应上
* 继承自Component的组件，在父组件render方法执行的条件下，不管props有没有改变，组件的render方法都会执行
* 继承自PureComponent的组件，父组件render之后，如果组件的props没有改变(引用数据类型浅比较依然与之前的对象相同)，则组件的render方法不会执行
* 执行`this.setState({})`后，继承自Component的组件的render方法会重新执行，而如果在继承自PureComponent的组件中执行以上代码，因为state对象的各值都没有变化，所以render方法不会执行
### **262. React的虚拟DOM和vue的虚拟DOM有什么区别？
### 263. 你觉得React上手快不快？它有哪些限制？
相对vue来说不快。
* 限制
- 需要学习JSX
- 需要工程化的配置
- 需要对原生JavaScript有相当的掌握
### **264. 说说你对声明式编程的理解？
### 265. React与angular、vue有什么区别？
angular不太了解。这里说下React和Vue.js的区别
* MVVM实现：Vue是采用依赖收集的方式来实现数据到视图的绑定,React使用的是Fiber Tree进行循环更新
* 事件：Vue使用原生的DOM事件，React使用合成事件，且使用了事件委托到appRoot节点(`<React17`是document节点)
### 266. React是哪个公司开发的？
facebook
### 267. React是什么？它的主要特点是什么？
React是用于构建用户界面的库。
特点：
* 组件化
* 数据到视图的单向绑定
* 数据的单项传递流
### 268. 简要描述下你知道的React工作原理是什么？
React以组件为基本的工作单位，每个组件都有自己的render方法(函数式组件直接返回)返回一个jsx结构用于描述组件的UI，组件有自己的状态state和从父组件接受的数据props；当state或props发生改变时，会引起React内部开始进行Reconciler（即DOM diff）操作，这个操作会首先检查当前节点的state和props是否改变，来决定是否更新当前节点的状态并执行响应的生命周期函数，并把节点差异存放在effect list（diff结果）中，一个节点检查完了之后会按照从顶向下、兄弟节点间逐级构造的方式进行循环这个操作，这个过程时可暂停的，依靠浏览器的`requestIdelCallback`进行调度；Reconciler完成了之后，这个组件树更新前后的diff结果已经算出来了，接下来就是不可中断commit操作，这个操作会把diff结果应用到真实的DOM上，执行`componentDidMount`或`componentDidUpdate`声明周期，整个更新流程完成。
### 269. 在React中怎样改变组件状态，以及状态改变的过程是什么？
* 使用`this.setState`改变组件的状态
* 改变的过程中，React Fiber Reconciler遍历了整个Fiber Tree，得到了最新的DOM diff结果，并把这个结果应用到真实的DOM上。同时相应的生命周期(`static getStateFromProps` `shouldComponentupdate` `render` `getSnapshotBeforeUpdate` `componentDidUpdate`)也在相应的时机执行.
### 270. 在React中你是怎么进行状态管理的？
* 使用React原生的Context+useContext
* 使用`redux` + `react-redux` + `redux-thunk`
* 使用`mobx` + `mobx-react`