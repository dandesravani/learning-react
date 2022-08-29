**jotai**

* Every component in react is an object and every component has its own state as well as identity.
* As our application is a component tree react identifies each and every component based on its position in the tree.

```JS
export const App = () => {
    return (
        <Counter initial={100}/>
        <Counter initial={200}/>
        <Counter initial={300}/>
    )
}
```
If the Counter in this second position is gets deleted then react will not be able to identify which component is missed in which position.

>
 ```JS 
const [count,setCount] = React.useState(0) 
const [text,setText] = React.useState(0) 
```
If multiple useState exists and one of the useState is called then react can able to identify the value that is associated with respective useState but identifying the value with in a tree is easy where as identifing with arrays is hard.

The first we call useState react assings each slot to each variable it maintains these values in an array it internally makes use of linked list.There is nothing maintained in react to actually identify among multiple useStates.

The problem with perticular approach in react is everytime component renders the invocation of these hooks should be in the exact same order the exact number if times since it is array based model.This way we cannot use hooks in event handlers,loops or conditions.

```JS
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
        <Counter/>
        <Counter/>
    )
}
```

Imagine we have 3 counter and we want one value to be shared among all the all three counters and we dont want 3 counters maintains their own state in that case we can use ***jotai***

jotai is an reactive librery which uses atoms. Atoms are nothing but reactive variables and we can keep them outside if the component.We should make use of function atoms and provide it with value we change.

```JS 
const countAtom = atom({count:100})
```

Atoms are like reactive variables as we can pass them as props. If we want to use the customAtom inside the component then we can make use of useAtom.

```JS
export const Counter = ({countAtom}:any) => {
    const [{count},setCount] = useAtom(countAtom)

    const handleInc =React.useCallback(() => {
        setCount(counter=>({
            count:counter.count+1
        }))
    },[])

    const handleDec =React.useCallback(() => {
        setCount(counter=>({
            count:counter.count-1
        }))
    },[])

    return (
        <CounterView count={count} inc={handleInc} dec={handleDec}/>
    )
}
```
useAtom returns the same as useState as it takes initial state as parameter and return array consists of current state and function which is used to change the state.

Though it seems like useState API useAtom is global and any component which uses useAtom shares the same atom variable. Everytime atom changes it makes every component that uses useAtom will re-render.

If we want we can have multiple atoms
```JS
const countAtom = atom({count:100})
const count2Atom = atom({count:200})
const count3Atom= atom({count:300})

export const App = () => {
    return (
        <Counter countAtom={countAtom}/>
        <Counter countAtom={count2Atom}/>
        <Counter countAtom={count3Atom}/>
    )
}
```
One of the advantange of having reactive variables is its derived values.we can  use derived values just as we use atoms.

```JS
const atoms = [countAtom,count2Atom,count3Atom]

const sumAtom= atom(get=>atoms.map(get).reduce(acc,v)=>acc+v.count,0)

export const App = () => {
    const [sum] = useAtom(sumAtom)
    return (
        <ChakraProvider>
            <div>
                {atoms.map(atom=>(
                    <Counter countAtom={atom}>
                ))}
            </div>
            <h1>{sum}</h1>
        <ChakraProvider>
    )
}
```
After rendering the App component we can see the sum of all the counter components.

Most applications might not need global state some might need page level state and some need local state. we need to identify what kind of state we need depends upon our application.

If we need global state we can make use of Redux or Reactive variables. If a application needs to maintain complex state and involves in lots of derived values then it is a very good idea to use Reactive variables.

If we want to make use of per page state we can consider using useReducer or useReducer along with provider based solution.

If we keep a folder for all state variables from tree state,page state and application level state then all variables are available to all different components then it is better to use Redux than reactive variables.

If we are fetching data from server side then it will not be necessary to it in state instead we could use libraries like useQuery and useSWR.

> Reactive variables are beneficial in such a situation that non connected components need to share some perticular state everywhere else try to use useReducer or useState if needed use global state.

***useEffect***

* useEffect is used to perform sideEffects. whenever reactive variables or derived variables changes and based on that we need to perform some action we can make use of useEffect.
* useEffect hooks takes functions as its first parameter and dependency array as second parameter.This callback function inside the useEffect is constructed everytime component is re-render but the react will use it or not depends upon the value we give to dependency array.
* useEffect is not part of rendering. Anything can not be done while rendering we can do it in useEffect like actions. Actions happen in event handlers.Actions which are can not be performed in handlers can be performed in useEffect.
* The callback function inside the useEffect will be called after the component is being rendered into the browser.
 
 ```JS
    export const App = () => {
        const [count,setCount] = React.useState(100)
        const [text,setText] = React.useState('')

        React.useEffect(()=>{
            console.log('helo')
        },[count,text])

        return (
            <h1>Hello</h1>
        )
    }
 ```

> useEffect takes a function or side effect which needs to executed and dependency array.In the first rendering it will retain those values that are there in the dependency array and invoke callback function.Next time that the useEffect passed with values like count and text it will shallow compare the array with the array that it has already stored. If the shallow compare return true then useEffect will not call the callback function otherwise it will invoke the callback function.

In a way useEffect is reacting to the change in values existing in the dependency array.It synchronizes the values with external system (effect).


 ```JS
    export const App = () => {
        const [count,setCount] = React.useState(100)
        const [text,setText] = React.useState('')

        React.useEffect(()=>{
            const id=setInterval(()=>{
                console.log("hello")
            },[1000])
            return () => clearInterval(id)
        },[count,text])

        return (
            <h1>Hello</h1>
        )
    }
 ```

> Being a DOM api setInterval will not be having any idea when to stop or start as it does not any have idea where or which component it is being used.we need to stop it manually. That is why setInterval returned back a variable which we can make use to clear the interval.In this way useEffect is responsible in releasing those intervals.

* setinterval will happend to be called everytime either count changes or text changes.Every time a value in the dependency array changes the previous will be cancelled ``` () => clearInterval(id)``` means this will call first and then the callback function inside useEffect will be called.

* If we give empty array as dependency array the function will be called once after component being rendered to the browser as we can say it is  mounting after that ``` () => clearInterval(id)``` this function will be called known as unmounting.

***Problems with useEffect***
 ```JS
    export const App = () => {
        const [count,setCount] = React.useState(100)
        const [text,setText] = React.useState('')

        React.useEffect(()=>{
            setCount(count++)
        },[text])

        return (
            <h1>Hello</h1>
        )
    }
 ```

The first time setText is called text changes then setCount is called eventually the component will rendered again.using useEffect along with set function inside it might cause multiple re-renders.
 
```Js
    React.useEffect(()=>{
            setCount(count++)
        },[count])   
```

This could cause infinite loop as every time count changes setCount will be called.This way reactive variables are seperate and derived values are seperate always try to create a tree not a graph.

```Js
    React.useEffect(()=>{
            setCount(text.length * 100)
        },[count])   
```

> useEffect with set function is almost always a bad idea.In the above case though it works it is unusal to keep count as seperate state as we could calculate it (derived value) based on existing value (text).we can consider count as simple value always try to maintain a ***minimal state***.

```JS
    export const App = () => {
        const [count,setCount] = React.useState(100)
        const [text,setText] = React.useState('')

        const handleClick = () => {
            setCount(count++)
        }

        React.useEffect(()=>{
            setTimeout(handleClick,1000)
            setCount(text.length * 100)
        },[handleClick])

        return (
            <h1 onClick={handleClick}>Hello</h1>
        )
    }
 ```

Imagine everytime text changes we want to invoke handleClick function after 1000ms.As we are using setTimeout funtion inside the useEffect we need to pass in function in the dependency array.Everytime handleClick changes useEffect needs to re-render.

Every time handleClick is called it makes entire component re-render eventually with every new render the new handle click function instantiated which leads to again change in handleClick in this way it leads to inifinite loop. Inorder to avoid such situations we can use useCallback.


```JS
    export const App = () => {
        const [count,setCount] = React.useState(100)
        const [text,setText] = React.useState('')

        const handleClick = React.useCallback(() => {
            setCount(count++)
        },[])

        React.useEffect(()=>{
            setTimeout(handleClick,1000)
            setCount(text.length * 100)
        },[])

        return (
            <h1 onClick={handleClick}>Hello</h1>
        )
    }
 ```

 As useCallback calls the function inside it everytime but it returns the same as it was before if the value in the dependency array remains unchanged.As handleClick won't change because of which useEffect does not need to be called again and again becuase handleClick is different.

```JS
  const handleClick = React.useCallback(() => {
            setCount(count++)
        },[])
```

The count captured here is because of first render it will remain like that.if we want to handleClick everytime foo changes then we need to do

```JS
  const handleClick = React.useCallback(() => {
            setCount(count++)
        },[count])
```

Again it is not a great solution as count will be always a different value as we calling setCount which makes component re-render then count will be different then the function will be different.After 1000ms milli seconds the handleClick function is called and setCount happens.Considering these cases we can do only simple thing i.e creating all of the handlers inside the useEffect.

```JS
   export const App = () => {
        const [count,setCount] = React.useState(100)
        const [text,setText] = React.useState('')

        React.useEffect(()=> {
              const handleClick =()=> {
            setCount(count++)
        }
            setTimeout(handleClick,1000)
            setCount(text.length * 100)
        },[count])

        return (
            <h1 onClick={handleClick}>Hello</h1>
        )
    }
```

If we need a function inside the useEffect then create it inside the useEffect and what ever the variables we use from outside can give to dependency array.

If you are making use of useEffect and having event handler outside of it and not using any dependency array means what ever it is captured before will be making use of just because dependency array didn't change

In another case if you add dependency that function might change everytime it might re-render multiple times and eventually leads to infinite loop if there are no other variables in dependency array. To tackle such situation we need to use useCallback but we need to careful with synchronization.

So the simpler way to do it is keep handler function inside it or try to write it as pure function with props and use it inside the useEffect.
