**Controlled vs Uncontrolled componets:**

* Though components which are driven by props can be redered pure it would be difficult to constrct the entire application with ***controlled components***.
* The application needs to have its own ***uncontrolled componenst*** inorder to have side effects like network calls and data fetching.
* Stateful components are like actor-model with actor being model have state which is mutable.Those actors receive messages and respond to it.State might change but nobody can access it directly and nobody can share it the actor owns it.
* Controlled components are also known as stateless components basically it is an ordinary function whose rendering is pure. Uncontrolled components are also called as stateful components which owns state and  considered as impure mostly doesn't take anything like props.
* When compared to the stateless components, stateful components are more complex and less reusable more over difficult to test.

***controlled component***

```JS

interface CounterViewProps {
readonly count:number
}

export const CounterView =({count}:CounterViewProps) => {
    return (
        <Button>+</Button>
        <Text>{count}</Text>
        <Button>-</Button>
    )
}
```

***Uncontrolled component***

```JS
export const Counter = () => {
    const [count,set] = React.useState(0)
    return (
        <CounterView count={count}/>
    )
}
```
* Counter is a function which use useState hook inorder to maintain state.Hooks are a way to connect a reactive variable to a pure component.
* ***useState*** hooks takes initial value as an input and returns current state and a function to change that state.
* when the component rendered for the first time the useState instantiates the state/object.After the initial render react doesn't care about the initial value of the state.
* In the first rendering react create a slot for the initial state and next when the useState is called that value at that slot will be returned and can be changed using useState.
* Before hooks are not introduced there exists life cycle methods inorder to seperate the purity and effects of the code.
* In programming we need both effects and pure rendering. React stats that effects are hooks and rendering is pure.
* Effects can happens when ever the state changes mostlt effects happens in event handlers.Event handlers are closures which captures the value from outside with every render it has its own state.

```JS

interface CounterViewProps {
    readonly count:number
    inc():void
    dec():void
}

export const CounterView =({count,inc,dec}:CounterViewProps) => {
    return (
        <Button onClick={inc}>+</Button>
        <Text>{count}</Text>
        <Button onClick={dec}>-</Button>
    )
}

export const Counter = () => {
    const [count,setCount] = React.useState(0)

    const handleInc =() => {
        setCount(count+1)
        setCount(count+1)
    }

    const handleDec = () => {
        setCount(count-1)
    }

    return (
        <CounterView count={count} inc={handleInc} dec={handleDec}/>
    )
}

export const App = () => {
    return (
        <Counter/>
    )
}
```

***problems with useState***

> Whenever the Counter is render though setCount is called twice count will be the same won't increment twice it is because the every time handleInc called the count setCount takes count from closure i.e from outside world.This way closures can be problematic.

```JS
   const handleInc =() => {
        setCount(count => count+1)
        setCount(count => count+1)
    }
```

>Inorder to avoid the problem with closure we can be pass function to the setState.The function which is passed to setState is pure and the count variable is not captured from outside.what captured here is setCount and its identity will remain same.

* Many of the times wh shouldn't worry about reactive variable we need to worry about variable only in rendering. In every render count is const but between renders it is mutable.

```JS const [count,setCount] = React.useState(slowFn(initial))```     slowFn is an ordinary javascript function it will be called everytie counter rendered but useState will use it only once i.e in the first render.

```JS const [count,setCount] = React.useState(()=>slowFn(initial))``` In this case react calls it once and forgets about it but it will create everytime whenever the counter renders.

In Uncontrolled components it is fine to take props so that we can control it from outside.

If we have complex state or two values exists sort of dependent in each other or having complex logic like timers, subscriptions, fetching using without useQuery we can use useReducer and can avoid useState.

**useMemo**

Whenever setState is called the whole component tree will be rerendered.

```JS
export const App = () => {
    return (
        <Counter initial={100}/>
        <Counter initial={200}/>
        <Counter initial={300}/>
    )
}
```

In the above example whenever the count of associated counter changes the specific counter will be rerendred the remaining counters will not rerender.

```JS
export const Hello = () => <h1>hello world</h1>

export const Counter = () => {
    return (
        <Counter initial={100} inc={handleInc} dec={handleDec}/>
        <Hello/>
    )
}
```

Where as in this case where ever the setcount is called it makes the whole component tree re-render.Though Hello component does not have any props to re-render it setCount from counter makes re-render.

Inorder to improve performance and unnecessary re-renders we can go for React.memo

```JS
export const CounterView =React.memo(({count,inc,dec}:CounterViewProps) => {
    return (
        <Button onClick={inc}>+</Button>
        <Text>{count}</Text>
        <Button onClick={dec}>-</Button>
    )
})
```

* React.memo maintains state. Everytime a component re-renders it will shallow compare the new props with the previous props.if the shallow comparision us true then it will re-render the component the previous VDOM structre remains the same eventually no DOM mutations happens.
* If we don't want shallow comparision we can pass deep comparision function to React.memo. But it will be costlier than our rendering as most of the cases shallow comparision will do the job.

```JS const Hello = React.memo(()=><h1>Hello world</h1>)``` React.memo will only renders it once i.e when the initial render of the component happens after that it won't re-render.But in this case there is no props to Hello component it won't add much rendering cost as it is a single element rendering will be cheap.

***useCallback***

* Everytime setCount is called the component renders and increment function will be instantiated every time with different identity and differnt prop as memo won't matter at all.This can be changed using ***useCallback***

```JS 
    const inc = React.useCallback(()=>{
        setCount(count+1)
    })
```

useCallback is a function to which we can pass state/modifying state to its array. when ever state inside that array changes it will create a new function otherwise it will not construct a new function every time.In the above it will only get called once.

```JS 
    const inc = React.useCallback(()=>{
        setCount(count+1)
    },[count])
```

Everytime count variable changes the callback function will be called and array will get compared with the previous array if it not the same then the callback function will be function to the Inc otherwise it will return the previous function it had stored in the first rendering.

```JS 
    const inc = React.useCallback(()=>{
        setCount(count => count+1)
    },[count])
```

In the above case Inc function is created only once so do dec function. These functions will never effect the rendering of Counterview component the only value force the Counterview is change in count value.
 
If we observe the above setCount is used as dispatch. It is almost like a reducer with only one action. The function we pass to useReducer always takes its state it does not depend on you capturing and providing it a value. It always depends upon dipatch that reducer is called by react then captures state.It always has state which it will be passed to reducer that simplifies things.




