## 序言
    React是一个库，它不是一个框架。用于构建用户界面的Javascript库。这里大家需要认识这一点。react的核心在于它仅仅是考虑了如何将dom节点更快更好更合适的渲染到浏览器中。它本身提供的涉及框架的理念是不多的。class组件是如此，hooks组件也是如此。
## ClassComponent
我们先回顾一下，这是一个react的class组件：
```javascript
class HelloMessage extends React.Component {
  constructor (props) {
      super(props)
      this.state = {
          num: 1
      }
  }
  componentDidMount () {
      alert('mounted!')
  }
  addNum () {
      this.setState({
          num: this.state.num + 1;
      })
  }
  render() {
    return (
      <div>
        Hello {this.props.name}
      </div>
    );
  }
}

ReactDOM.render(
  <HelloMessage name="Taylor" />,
  document.getElementById('hello-example')
);
```
它包含了：
* props
* state
* setState
* render
* 生命周期

再来看看class组件中比较特殊的，具有代表意义的东西：

1、setState
    通过对象更新的方式更新组件状态，react通过diff算法自动更新dom

2、Ref
     一个可以用来访问dom对象的东西

3、this
     classComponent中通过this访问各种数据/方法
4、生命周期
如图所示：


class组件存在的问题
    * 逻辑分散
    * 数据分散
    * 组件拆分繁琐，state树往往较大

我们构建React组件的方式与组件的生命周期是耦合的。这一鸿沟顺理成章的迫使整个组件中散布着相关的逻辑。在下面的示例中，我们可以清楚地了解到这一点。有三个的方法
componentDidMount、
componentDidUpdate和updateRepos
来完成相同的任务——使repos与任何props.id同步。

```javascript
componentDidMount () {
    this.updateRepos(this.props.id)
 }
 componentDidUpdate (prevProps) {
    if (prevProps.id !== this.props.id) {
      this.updateRepos(this.props.id)
    }
 }
 updateRepos = (id) => {
    this.setState({ loading: true })
    fetchRepos(id)
      .then((repos) => this.setState({
        repos,
        loading: false
      }))
  }
```
为了解决这个问题（它是一系列问题的其中之一，这里只简单举例），react创造了一个全新的方式来解决，那就是hooks。

## FunctionalComponent
hooks的目的是让大家在一定程度上 **“使用函数的想法去开发组件”** 。
纯函数组件、高阶组件
**先粗浅理解函数式**
主要思想是把运算过程尽量写成一系列嵌套的函数调用。举例来说，现在有这样一个数学表达式：
```javascript
(1 + 2) * 3 - 4
```
传统的过程式编程，可能这样写：
```javascript
var a = 1 + 2;

var b = a * 3;

var c = b - 4;
```
函数式编程要求使用函数，我们可以把运算过程定义为不同的函数，然后写成下面这样：
```javascript
var result = subtract(multiply(add(1,2), 3), 4);
```

**函数式的特点**

1. 函数是"第一等公民"


2. 只用"表达式"，不用"语句"

"表达式"（expression）是一个单纯的运算过程，总是有返回值；"语句"（statement）是执行某种操作，没有返回值。函数式编程要求，只使用表达式，不使用语句。也就是说，每一步都是单纯的运算，而且都有返回值。

原因是函数式编程的开发动机，一开始就是为了处理运算（computation），不考虑系统的读写（I/O）。"语句"属于对系统的读写操作，所以就被排斥在外。

当然，实际应用中，不做I/O是不可能的。因此，编程过程中，函数式编程只要求把I/O限制到最小，不要有不必要的读写行为，保持计算过程的单纯性。

**纯函数组件**
FunctionComponent：

满足“不修改状态”， “引用透明” ，“组件一等公民”， “没有副作用”， “表达式”
(图)

**高阶组件**
higherOrderComponent(高阶组件)：

传入一个组件，返回另一个包装组件，向其注入数据、逻辑，达到拆分组件的目的
(图)
```javascript
import * as React from 'react';

export default function () {
    return (
        ComponentA({ Com: ComponentB })
    )
}

function ComponentA (
    { Com }: { Com: React.FunctionComponent<any> }
) {
    return (
        <div>
            <Com style={{ color: 'red' }} />
        </div>
    )
}

function ComponentB ({ style }: {
    style: any
}) {
    return (
        <div style={style}>hello high order component</div>
    )
}
```


## Hooks与实践
Hooks你可以理解为具备class组件功能的函数式组件——至少它的理想是函数式

**HooksAPI**
1、useState

2、useReducer
    
3、useEffect
    
4、useLayoutEffect
     
5、useMemo    

6、useCallback

7、useContext

8、useRef

**Hooks基本规则**
1、只在最顶层使用 Hook(**why? 后文提到**)

不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。(如果你对此感到好奇，在下面会有更深入的解释。)


2、只在 React 函数中调用 Hook
3、严格控制函数的行数（相信我，超过500行的函数，任何人都会头秃）
**hooks生命周期对应**
```javascript
export default function HooksComponent () {
    const [ date, setDate ] = React.useState(new Date());
    const div = React.useRef(null);

    const tick = React.useCallback(() => {
        setDate(new Date());
    }, [])

    const color = React.useCallback(() => {
        if (div.current) {
            if (div.current.style.cssText.indexOf('red') > -1) {
                div.current.style.cssText = 'background: green;'
            } else {
                div.current.style.cssText = 'background: red;'
            }
        }
    }, [ div ]);

    const timerID = React.useMemo(() => {
        console.log('==组件是否更新（概念有区别）==')
        return setInterval(
            () => tick(),
            1000
        );
    }, [ div ])

    React.useEffect(() => {
        console.log('==组件加载完成==')
        return () => {
            console.log('==组件将要销毁==')
            clearInterval(timerID);
        }
    }, [])

    React.useEffect(() => {
        console.log('==组件更新完成==')
        color();
    }, [ date ])

    console.log('==组件将要加载==')
    console.log('==组件获取新的props==')
    console.log('==组件将要更新==')


    return (
        <DemoComponent />
    )
}
```
(图)

## Hooks原理demo实现

实现文件：
```javascript
import * as React from 'react';

type State = {
    [key: string]: any
}

export type Reducer = (state: State, action: string) => State;

export type UseReducer = typeof demoReducer;

type Queue<T> = {
    root: Queue<T> | null,
    next: Queue<T> | null,
    prev: Queue<T> | null,
    value: T,
    index: number
}

const memo: Queue<State>[] = [];

let index = 0;

function demoReducer (reducer: Reducer, initalState?: State, update?: () => void) : any[] {
    let has = !!memo[index];
    if (!memo[index]) {
        if (index === 0) {
            memo[0] = GeQueue<State>(initalState, index);
            memo[0].root =  memo[0];
        } else {
            memo[index] = GeQueue<State>(initalState, index);
            memo[index].root =  memo[0];
            memo[index].prev = memo[index - 1];
            memo[index-1].next = memo[index];
        }
        index = index + 1;
    }
    let state;
    if (has) {
        state = memo[index].value;
        index = index + 1;
    } else {
        state = memo[index - 1].value;
    }
    let bindex = index;
    return [
        state,
        (action: string) => {
            memo[bindex - 1].value = reducer(memo[bindex - 1].value, action)
            update();
        }
    ]
}

function GeQueue<T>(value: T, index: number): Queue<T> {
    return {
        root: null,
        prev: null,
        next: null,
        value,
        index
    }
}

export function DemoHoc ({ Com }: any) {
    const [ u, setU ] = React.useState(1)
    return (
        <div>
            <Com useReducer={(reducer: Reducer, initalState?: State) => {
                return demoReducer(reducer, initalState, () => {
                    index = 0;
                    setU(u + 1);
                })
            }} />
        </div>
    )
}
```
使用文件：
```javascript
import * as React from 'react';
import { DemoHoc, UseReducer } from './useReducer/useReducerDemo'
import { Button } from 'antd';

export default function HookDemo () {
    return (
        <DemoHoc Com={DemoComponent} />
    )
}

const initialState = {count: 0};

function reducer(state: any, action: any) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}


function DemoComponent({ useReducer }: {
    useReducer: UseReducer
}) {
    const [ state, dispatch ] = useReducer(reducer, initialState);
    const [ state2, dispatch2 ] = useReducer(reducer, {...initialState});
    
    return (
        <div>
            Count: {state.count}
            <Button onClick={() => dispatch({type: 'decrement'})}>-</Button>
            <Button onClick={() => dispatch({type: 'increment'})}>+</Button>
            <div></div>
            Count2: {state2.count}
            <Button onClick={() => dispatch2({type: 'decrement'})}>-</Button>
            <Button onClick={() => dispatch2({type: 'increment'})}>+</Button>
        </div>
    )
}
```

原理：
我们先看实现文件的这个函数（不了解ts的同学，可以忽略ts的理解）：
```javascript
const memo: Queue<State>[] = [];

let index = 0;
function demoReducer (reducer: Reducer, initalState?: State, update?: () => void) : any[] {
    let has = !!memo[index];
    if (!memo[index]) {
        if (index === 0) {
            memo[0] = GeQueue<State>(initalState, index);
            memo[0].root =  memo[0];
        } else {
            memo[index] = GeQueue<State>(initalState, index);
            memo[index].root =  memo[0];
            memo[index].prev = memo[index - 1];
            memo[index-1].next = memo[index];
        }
        index = index + 1;
    }
    let state;
    if (has) {
        state = memo[index].value;
        index = index + 1;
    } else {
        state = memo[index - 1].value;
    }
    let bindex = index;
    return [
        state,
        (action: string) => {
            memo[bindex - 1].value = reducer(memo[bindex - 1].value, action)
            update();
        }
    ]
}
```
* 最开始，我们用了一个memo对象（它是一个队列）这里简单的定义成数组。 然后又又一个index，记录数字关系
* 然后重点来了
```javascript
let has = !!memo[index];
    if (!memo[index]) {
        if (index === 0) {
            memo[0] = GeQueue<State>(initalState, index);
            memo[0].root =  memo[0];
        } else {
            memo[index] = GeQueue<State>(initalState, index);
            memo[index].root =  memo[0];
            memo[index].prev = memo[index - 1];
            memo[index-1].next = memo[index];
        }
        index = index + 1;
    }
    let state;
    if (has) {
        state = memo[index].value;
        index = index + 1;
    } else {
        state = memo[index - 1].value;
    }
    let bindex = index;
```
我们调用useReducer的时候，从memo中尝试获取index的值，若存在的话，将state赋值为当前memo里的值并返回

* 最后我们提供了一个触发更新的函数

然后-我们再看看使用的代码：
```javascript
const initialState = {count: 0};

function reducer(state: any, action: any) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}


function DemoComponent({ useReducer }: {
    useReducer: UseReducer
}) {
    const [ state, dispatch ] = useReducer(reducer, initialState);
    const [ state2, dispatch2 ] = useReducer(reducer, {...initialState});
    
    return (
        <div>
            Count: {state.count}
            <Button onClick={() => dispatch({type: 'decrement'})}>-</Button>
            <Button onClick={() => dispatch({type: 'increment'})}>+</Button>
            <div></div>
            Count2: {state2.count}
            <Button onClick={() => dispatch2({type: 'decrement'})}>-</Button>
            <Button onClick={() => dispatch2({type: 'increment'})}>+</Button>
        </div>
    )
}
```

* useReducer时经过memo的存储，我们在队列结构里存下了值，当dispatch的时候，memo中的变量将会改变。

所以各位看出来了吗？

这就是hooks隐藏的逻辑所在！
----------------------------
重点来了～ 
**hooks的本质是什么？**

## Hooks本质看坑与规范

hooks的本质是-闭包。
它在外部作用域存储了值。
当setState发生时，整个函数（相当于render）都被触发调用，那么所有的变量就会重新赋值就像是下面的关键字：
* const
* let
* function 
好多同学会觉得，奇怪了，为啥这个函数被重新调用，不会重新赋值呢？ 
关键就是闭包！
事实上hooks的每个setState都会导致function变成一个片段。在这之前的变量并不是之后的变量。
而我们在onclick等事件中引用的函数，有可能会存储旧的变量引用！这就会导致一些大问题！！！
* 例如内存泄漏
* 例如值的引用无效
那怎么解决？
react其实提供了解决办法，就是useMemo useCallback等等。你可以再深入的去理解它。

**Hooks使用大忌**

1、 外部顶层变量问题，不要把全局变量放在函数外部，特别是你在开发公共组件的时候，你可以用useRef
2、依赖项不正确问题，依赖项不正确会导致闭包问题。
3、组件拆分太粗问题，函数不建议太长，500行以内最佳
4、盲目使用usexxx，要深刻理解才能更好的使用hooks

## 最后

高效编码；健康生活；



