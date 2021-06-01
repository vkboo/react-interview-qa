# React 328道最全面试题 & 个人作答

> [React 328道最全面试题](https://juejin.cn/post/6844903892853981198)

### React
#### 1. 什么时候使用状态管理器？
React自身的数据只支持从父到子的传递，在兄弟组件和从子到父的数据传递中，需要通过回调到父组件来进行中转，所以，在构建系统的过程中，通常会将大部分的状态放在公共的父组件上。然而，大型系统中，各个模块中的数据存在相互引用，此时虽然可以引入`Context`来存放一些公共的数据，当数据量较多且复杂，Context的功能比较单一，调试与数据追踪、时间旅行处理起来都比较棘手，此时就可以引入状态管理器。
#### 2. render函数中的return如果没有使用`()`会有什么问题？
标签内换行导致的语法问题，即`JSX`一个整体本质上是一个对象，用`()`包起来也很合理
#### 3. componentWillUpdate可以直接修改state的值吗？
不能。`componentWillUpdate`发生在`state`或`props`改变的情况下，在`shouldComponentUpdate`返回`true`且在`render`前执行，如果在`componentWillUpdate`中进行了`setState`或者`Redux`中的`dispatch(action)`操作，会导致`Maximum update depth exceeded`错误。
#### 4.说说你对React的渲染原理的理解
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
#### 12. 除了实例的属性可以获取Context外哪些地方还能直接获取Context呢?
#### 13. childContextTypes是什么？它有什么用？
#### 14. contextType是什么？它有什么用？
#### 15. Consumer向上找不到Provider的时候怎么办？
#### 16. 有使用过Consumer吗？
