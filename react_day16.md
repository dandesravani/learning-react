```JS
export const Counter  = () => {
    // const [count,setCount] = React.useState(0)

   React.useEffect(()=>{
    const id = setInterval(()=>{},1000)

    return () => clearinterval(id)
   },[])

    return (
        <Text>Hello world</Text>
    )
}
```

Being a DOM api manipulation setInterval does not go well with React as the function passed to setInterval will keep getting called for every milliseconds that passes to it.so whatever it captures will be problem because the lifecycle of the setInterval function will be different from lifecycle of the component.The value captured by the callback function will retain by the setInterval until clearInterval was called so using setInterval at multiple places will lead to different kind of problems.Inorder to avoid that and to make a component more reusable we can write 

```JS
export const Counter  = () => {
    const [count,setCount] = React.useState(0)
    const [delay,setDelay] =  React.useState(1000)

   React.useEffect(()=>{
    const id = setInterval(()=>{
        setCount(c=>c+1)
    },delay)

    return () => clearinterval(id)
   },[delay])

    return (
        <Text>{count}</Text>
    )
}
```

As we can see the setter function does not capturing value from clojure it is using function form of setter means it is not taking value from current render leads to proper updation of state everytime component re-render.Even though we are capturing state correctly we can not use it with another state variable which changes or what value should go dependency array those kind of problems still exists.

The milliseconds that is supposed to pass setInterval can not always be 1000ms and it should be anything so we can make it dynamic variable.

Everytime delay changes useEffect captures the setinterval function with new delay value and clears the previous interval and sets new one.There will a few milli seconds which might lost.

```JS
const useInterval = (fn:()=>void,delay:number) => {
    React.useEffect(()=>{
    const id = setInterval(()=>{
        fn()
    },delay)

    return () => clearinterval(id)
   },[delay])
}

export const Counter  = () => {
    const [count,setCount] = React.useState(0)
    const [delay,setDelay] =  React.useState(1000)

    useInterval(()=>{
        setCount(c=>c+1)
    },delay)

    return (
        <>
            <Text>{count}</Text>
            <input onChange={(evt)=>setDelay(Number(evt.target.value))}/>
        </>
    )
}
```

With this approach we may encounter two more problems those are, Everytime delay change useEffect captures the new function or should we stick to the function that it initially stores when the component first renders.ANother one is every time delay changes should we clear the interval or not.

Whatever the function pass to useInterval it should be the latest one Inorder to get that,

```JS
    const useInterval = (fn:()=>void,delay:number) => {
        React.useEffect(()=>{
        const id = setInterval(()=>{
            fn()
        },delay)

    return () => clearinterval(id)
   },[fn,delay])
}
```

Everytime function changes useInterval will be called again and clears the previous interval sets a new interval.Again here function will be brand new. we can fix it by using React.useCallback

```JS
    useInterval(React.useCallback(()=>{
        setCount(c=>c+1)
    },[]),delay)
```


***useRef***

Though the above works well enough the function that need to be passed to useInterval should be wrapped in useCallback which leads to complexity.We can drop the useCallback but without resetting useEffect because that could lose few milliseconds.There is no need to set or clear interval everytime there is a change in function.It can set or clear everytime there is a change in delay.In this scenerio we can make use of useRef

```JS
const useInterval = (fn:()=>void,delay:number) => {
    React.useEffect(()=>{
    const id = setInterval(()=>{
        fn()
    },delay)

    return () => clearinterval(id)
   },[delay])
}

export const Counter  = () => {
    const [count,setCount] = React.useState(0)
    const [delay,setDelay] =  React.useState(1000)
    const ref = React.useRef(100)

    useInterval(()=>{
        ref.current=1000
        setCount(count+1)
    },delay)

    return (
        <>
            <Text>{count}</Text>
            <input onChange={(evt)=>setDelay(Number(evt.target.value))}/>
        </>
    )
}
```
Irrespective of render we will get the same ref means the same pointer and the pointer always pointing to the same object.It is very similar to useState in that sense it will always return back the same set.we can change our ref in sideEffects.it always holds its value in current object.

> The only difference between useState and useRef is, If we change a value in useState it will make the component re-render where as in useRef assigning a value to current will not re-render the component.

Anytime we want a variable whose modification should not lead to re-rendering of a component then we can make use of ***useRef***.

```JS
const useInterval = (fn:()=>void,delay:number) => {
    const callback = React.useRef<()=>void>(fn)

    React.useEffect(()=>{
        callback.current = fn
    })
    
    React.useEffect(()=>{
    const id = setInterval(()=>{
        fn()
    },delay)

    return () => clearinterval(id)
   },[delay])
}

```

As we are not providing any dependency array we can still see latest function geting retained and called.There will be no re-renderings or re-execution of effects.

***useImmerReducer***

* Though it seems the code for useReducer is much verbose it keeps all the state mutation code like handlers and effects out of the component which makes the compoenent pure.
* useReducer always the have access to most recent state if you define the useReducer within the component it will always have access to latest props.capturing errors can be avioded.
* Apart from useReducer and useState there is useImmerReducer.The advantage of useImmer and useImmerReducer is that we can directly make use of mutative operations and can still a new copy of it will constructed.

```JS
import React from "react";
import { useImmerReducer } from "use-immer";

const initialState = { count: 0 };

function reducer(draft, action) {
  switch (action.type) {
    case "reset":
      return initialState;
    case "increment":
      return void draft.count++;
    case "decrement":
      return void draft.count--;
  }
}

function Counter() {
  const [state, dispatch] = useImmerReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "reset" })}>Reset</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </>
  );
}

```