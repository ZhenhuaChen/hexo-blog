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
确保从不同的用户默认





 



