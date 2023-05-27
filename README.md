#### useState
1. react的所有状态都应视为不可变的，要避免直接改变引用类型的数据，而是始终set一份新的数据。

> 为什么要这样？React是这样说的
> 
> 调试：如果你使用console.log并且不改变状态，你过去的日志将不会被最近的状态变化所破坏。所以你可以清楚地看到状态在渲染之间是如何变化的。
> 
> 优化：如果前一个道具或状态与下一个相同，则常见的 React优化策略依赖于跳过工作。如果你从不改变状态，那么检查是否有任何变化是非常快的。如果prevObj === obj，您可以确定它内部没有任何变化。
> 
> 新功能：我们正在构建的新 React 功能依赖于将状态视为快照。如果您正在改变过去版本的状态，这可能会阻止您使用新功能。
> 
> 需求变更：某些应用程序功能，如实现撤消/重做、显示变更历史或让用户将表单重置为较早的值，在没有任何变化时更容易实现。这是因为您可以在内存中保留过去的状态副本，并在适当的时候重用它们。如果您从可变方法开始，以后可能很难添加此类功能。
> 
> 更简单的实现：因为 React 不依赖于变异，所以它不需要对你的对象做任何特殊的事情。它不需要劫持它们的属性，总是将它们包装到代理中，或者像许多“反应式”解决方案那样在初始化时做其他工作。这也是 React 允许您将任何对象放入状态的原因——无论对象有多大——没有额外的性能或正确性缺陷。

2. setState之后，不是立即去通知模板渲染的，而是排队在下一次渲染之前，批量set当次渲染需要改变的数据。

> 这使您可以更新多个状态变量——甚至来自多个组件——而不会触发太多重新渲染。但这也意味着在您的事件处理程序及其中的任何代码完成之前， UI 不会更新。这种行为，也称为批处理，可以让你的 React 应用程序运行得更快。它还避免处理令人困惑的“半成品”渲染，其中仅更新了一些变量。

3. 同一个状态，在一次渲染中，需要多次更改的处理方式：
```JavaScript
const [number, setNumber] = useState(0);
// x 当次渲染中number始终为0，也就是两次setNumber(0+1)
setNumber(number + 1)
setNumber(number + 1) 

// √ 传递一个函数作为参数，该函数会根据前一个状态计算下一个状态队列
setNumber(n => n + 1) 
setNumber(n => n + 1) 
```

#### useEffects
> Effects 是 React 范式的逃生通道。它们让你“走出”React 并将你的组件与一些外部系统同步，比如非 React 小部件、网络或浏览器 DOM。
> 
> **如果不涉及外部系统（例如，如果您想在某些道具或状态更改时更新组件的状态），则不需要 Effect。删除不必要的 Effects 将使您的代码更易于理解、运行速度更快并且更不容易出错。**

> 您不需要 Effects 来转换数据以进行渲染。例如，假设您想在显示列表之前对其进行过滤。您可能很想编写一个 Effect 在列表更改时更新状态变量。然而，这是低效的。**当你更新状态时，React 将首先调用你的组件函数来计算屏幕上应该显示什么。然后 React 会将这些更改“提交”到 DOM，更新屏幕。然后 React 将运行您的 Effects。如果您的 Effect也立即更新状态，则整个过程将从头开始**为避免不必要的渲染过程，请转换组件顶层的所有数据。只要您的道具或状态发生变化，该代码就会自动重新运行。

1. 个人暂时的一些感悟：  
如果要依赖一个状态1，而改变另一个状态2。  
可以直接在函数组件顶层写因状态1的函数判断，然后赋值给状态2，这样会先计算出变化，再渲染UI。  
而用Effect，是状态1改变先渲染UI，Effect监听到状态1改变，再让状态2改变，从而再次去渲染UI。

2. 用法
```JavaScript
  useEffect(() => {
    console.log("打印新数据:" + data);
    // data改变后要做的一些事
    // 或者组件挂载要做的事(可以依赖一个空数组)
    return () => {
      console.log("打印旧数据:" + data);
      // data改变前要做的一些事
      // 或者组件销毁要做的事(可以依赖一个空数组)
    };
  }, [data]);
```

#### useContext
使孙子组件可以访问顶层组件，配合createContext使用。
```JavaScript
import { createContext, useContext } from "react";

const MyCtx = createContext();
export default function Father() {
  return (
    <MyCtx.Provider value="你的数据">
      <Child />
    </MyCtx.Provider>
  );
}
function Child() {
  const ctx = useContext(MyCtx);
  return <>{ctx}</>; //你的数据
}

```

#### useRef
1. 非常适合存储不影响组件视觉输出的信息。
> 特点  
> 您可以在重新渲染之间存储信息（不同于在每次渲染时重置的常规变量）。  
> 更改它不会触发重新渲染（与触发重新渲染的状态变量不同）。  
> 该信息对于组件的每个副本都是本地的（不像外部变量，它们是共享的）。

2. 使用 ref 来操作 DOM 特别常见
```JavaScript
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  function handleClick() {
    inputRef.current.focus();
  }
  return <input ref={inputRef} />;
```

#### useReducer
1. useReducer与useState非常相似，但它允许您将状态更新逻辑从事件处理程序移动到组件外部的单个函数中。来比较一下：  
> a. 代码大小：一般来说，使用 useState 你必须预先编写更少的代码。 使用 useReducer，您必须同时编写 reducer 函数和调度操作。 但是，如果许多事件处理程序以类似方式修改状态，useReducer 可以帮助减少代码。  
> b. 可读性：当状态更新很简单时，useState 非常容易阅读。 当它们变得更复杂时，它们会使您的组件代码膨胀并使其难以扫描。 在这种情况下，useReducer 可以让您清楚地将更新逻辑的方式与事件处理程序发生的事情分开。  
> c. 调试：当您遇到 useState 的错误时，可能很难判断状态设置错误的位置以及原因。 使用 useReducer，您可以将控制台日志添加到您的 reducer 中，以查看每个状态更新，以及它发生的原因（由于哪个操作）。 如果每个动作都是正确的，你就会知道错误出在 reducer 逻辑本身。 但是，与 useState 相比，您必须逐步执行更多代码。  
> d. 测试：reducer 是一个不依赖于您的组件的纯函数。 这意味着您可以隔离地单独导出和测试它。 虽然通常最好在更真实的环境中测试组件，但对于复杂的状态更新逻辑，断言你的 reducer 为特定的初始状态和操作返回特定的状态可能很有用。  

2. 用法：
```JavaScript
import { useReducer } from 'react';

function reducer(state, action) {
  console.log(action)
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```
