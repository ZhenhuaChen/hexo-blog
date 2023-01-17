---
title: React Hooks API (官网翻译版本)
date: 2023-01-04 17:56:26
tags:
---
# Hooks API
***Hooks 是在React 16.8 新添加的特性， 作用是可以在不写class的情况下使用state 和 react的其它的特性***

这里介绍React中内Hooks 的 API。
假如你是hooks的新手， 可以先看一下[概览](https://reactjs.org/docs/hooks-overview.html), 你也可以从[FAQ](https://reactjs.org/docs/hooks-faq.html)中发现你想要的信息.

# 基础Hooks
* **[useState](#jump_1)**
* **[useEffect](#jump_2)**
* **[useContext](#jump_3)**
<!--more-->
# 附加Hooks
* **[useReducer](#jump_4)**
* **[useCallback](#jump_5)**
* **[useMemo](#jump_6)**
* **[useRef](#jump_7)**
* **[useImperativeHandle](#jump_8)**
* **[useLayoutEffect](#jump_9)**
* **[useDebugValue](#jump_10)**
* **[useDeferredValue](#jump_11)**
* **[useTransition](#jump_12)**
* **[useId](#jump_13)**


<a id="jump_1"></a>

## useState
```
const [state, setState] = useState(initialState);
```
返回一个状态值和更新值的方法
在初始渲染期间，返回的状态与传递的第一个参数相同
SetState功能用于更新状态。它接受新的state值并重新渲染组件。

```
setState(newState)
```
在之后的渲染中，useState返回的第一个值永远都是应用更新后的最新状态

> 注意： react 保证了setState 方法的稳定性，在re-render期间不会改变，这是为什么可以忽略使用
useEffect 和 useCallback 依赖关系列表的原因



### 函数式更新
如果新的state值是根据之前的state计算而来的，你可以给setState传递一个function. 这个function将接受之前的值，
返回更新后的值，下面是使用两种形式的setState实现计数器的例子

```
function Counter({initialCount}) {
    const [count, setCount] = useState(initialCount);
    return (
        <>
            Count: {count}
            <button onClick={() => setCount(initialCount)}> Reset</button>
            <button onClick={() => setCount(prevCount => prevCount - 1)}> - </button>
            <button onClick={() => setCount(prevCount => prevCount + 1)}> + </button>
        </>
    )
}
```
上面例子中，+ 和 - 按钮使用函数式形式，因为更新值是基于之前的值， 但是Reset按钮使用常规形式，因为它总是将值设置为初始值
<br>
假如你的更新方法返回与当前state完全相同的值，则将完全跳过后续的渲染（rerender）

> 注意： 与 class 型 组件中setState 不同的是，useState不会自动合并更新的对象，你可以通过将函数式更新方法中的参数（prevState也就是之前的值）与新的值合并来实现这个功能
```
const [state, setState] = useState({});
setState(prevState => {
    //Object.assign would also work
    return {...prevState, ...updatedValues};
});

```
>另一个操作是useReducer， 它更适合管理包含多个子值的对象状态

### 惰性初始状态

初始状态参数是初始渲染期间使用的状态，在之后的渲染中他会被忽略。如果初始状态是开销比较大的计算结果，你可以提供一个函数代替，
它将仅在初始渲染时执行

```
const [state, setState] = useState(() => {
    const initialState = someExpensiveComputation(props);
    return initialState;
})

```
### 退出状态更新
假如你更新一个state Hook 的值和当前的值相同，react将在不渲染子项或触发的情况下退出(React 使用 Object.is 方法比较两个值)
注意在推出之前，react可能任需要再次渲染特别的组件。这不应该是一个问题，因为 React 不会不必要地“更深入”到树中。如果您在渲染时进行较大的计算，可以使用 useMemo 对其进行优化。

### 批处理状态更新
react会将几个状态跟新合并为一组，以提高性能， 通常提高性能不会影响程序的行为
<br />
在react18之前,只有react事件处理程序内部的更新是批处理的，从react18开始，默认批处理。注意，react
确保从几个不同的用户触发的事件中进行更新，比如：点击一个按钮两次————总是单独处理，不会批量处理。这个可以防止逻辑错误。
在极少数情况下，你需要强迫同步应用DOM更新，你可以将其包裹在flushSync中。但是，这将会损害性能，因此请仅在需要时执行此操作


<a id="jump_2"></a>

## useEffect
```
useEffect(didUpdate);
```
接收包含命令式的函数，可能有效的代码。
在功能组件的主体（称为React的渲染阶段）内不允许发生突变、订阅、计时器、日志记录和其它副作用，这样做会导致UI中
出现令人困惑的bugs和不一致
<br />
相反可以使用useEffect. 传递给useEffect的函数将在渲染提交到屏幕后运行。将effect视为从react的纯函数世界进入命令世界的逃生舱口
<br />
默认情况下，effects执行在完成render之后，但是你可以在包含的值发生改变的时候执行它们。

### 清除副作用（effect）
通常，在组件销毁之前effects 创建的资源需要被清除，比如订阅或者定时器ID。要实现这个，传递给useEffect的函数需要返回一个清除函数。比如，创建一个订阅：
```
useEffect(() => {
    const subscription = props.source.subscribe();
    return () => {
        <!-- 清除订阅 -->
        subscription.unsubscribe();
    };
});
```
组件从UI移除之前执行清除函数以防止内存泄漏。另外，如果一个组件渲染多次,在执行新的effect之前将会清除前一个的effect。在我们的示例中，这意味着每次更新都会创建一个新的订阅，为避免对每次更新都产生一个effect，请参考下一节

Effect的时间
不像componentDidMount 和 componentDidUpdate，在延迟时间期间，函数传递给useEffect的函数在渲染之后触发. 这使得它适用于很多副作用，比如设置订阅和事件处理程序，因为大多数类型工作不应该阻止浏览器更新渲染.
<br />
然而，不是所有的effects可以被延迟。比如，用户可见的dom突变必须在下一次绘制之前同步触发，以便用户不会察觉到视觉上的不一致（这种区别在概念上类似于被动事件监听器与主动事件监听器).对于这些effect类型，react提供了一个额外的hooks————useLayoutEffect。 它与useEffect有同样的特性,仅在触发时有所不同。
<br />
此外，从react18开始，传给useEffect的函数将在布局和绘制之前同步触发，当它是用户输入（例如单击）的结果时，或者当它是包含在flushSync中的更新结果时，这个行为允许事件系统或flushSync的调用者观察到effect的结果

<br />
注意：这只会影响调用传递给useEffect的函数的时间 - 这些effect内的安排的更新仍然会延迟。这与useLayoutEffect不同，后者会触发函数并立即处理其中的更新。
即使在useEffect被推迟到浏览器渲染之后，也可以保证在新的渲染之前触发useEffect。在开始新的更新之前，react总是会刷新之前渲染的effects。

### 有条件的触发effect
effect的默认行为是在每次完成渲染后触发effect。这样，如果其依赖项之一发生更改就重新创建effect
<br />
然而在某些情况下这可能会过度防御，像前面提到的订阅。在每次更新时我们不需要创建新的订阅，只有在prop发生改变时候才需要重建订阅。
<br />
为此，给useEffect传入第二个参数是由effect依赖的值组成的数组，使用像下面这样：
```
useEffect(() => {
    const subscription = props.source.subscribe();
    return () => {
        subscription.unsubscribe();
    };
},[props.source])
```
现在，只有在props.source发生改变的时候，订阅才会被重新创建.
<br />
注意：假如你使用以上的方法，确保该数组包含effect使用的组件范围内随时间变化（例如props和state）的值。否则，你的代码将引用以前渲染中的旧值，详细了解[如何处理函数](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) 以及 [当数组值更改过于频繁时改怎么做](https://reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)。
<br />
如果你仅想执行一次effect就清除它（mount 和 unmount），你可以传一个空的数组（[]）作为第二个参数。这告诉react你的effect不需要依赖props 或者 state的值，因此不需要再次执行。 这并不是作为特殊情况处理的————它直接遵循dependencies数组时如何工作的。
<br />
如果你传递一个空数组（[]）,effec中的props和state总时有它们的初始值。虽然将[]作为第二个参数传递更接近熟悉的componentDidMount 和 componentWillUnmount 心智模型，通常有更好的解决方法去避免effects太频繁的re-running。 另外，不要忘记React会延迟运行useEffect直到浏览器绘制完成，所以额外的工作不是问题。
<br />
我们推荐使用exhaustive-deps规则作为eslint-plugin-react-hooks包的一部分。当依赖项指定不正确时，它会发出警告并建议修改。
<br />
依赖数组并没有作为参数传给effect函数，不过从概念上讲，这就是它们所代表的含义：effect函数中引用的每个值也应该出现在dependencies数组中。将来，一个足够先进的编译器可以自动创建这个数组.


















 



