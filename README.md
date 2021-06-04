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
react-transition-group
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
原因：***??
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
### 59. 在React中怎么阻止事件的默认行为？
### 60. 你最喜欢React的哪一个特性（说一个就好）？

