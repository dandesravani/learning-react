**custom hook**

Inorder to fetch data from server we can write a hook which gives data and error. When the request (fetcher) is not yet finished, data will be undefined. And when we get a response, it sets data and error based on the result of api call and rerenders the component.

custom hook accepts a key.The key is a unique identifier of the request, normally the URL of the API. And the API call accepts key as its parameter and returns the data asynchronously.

There are several libraries which are used to make HTTP requests. Fetch and axios and redaxios are most popularly used.

* To send data, fetch() uses the body property for a post request to send data to the endpoint, while Axios uses the data property
* The data in fetch() is transformed to a string using the JSON.stringify method
* Axios automatically transforms the data returned from the server, but with fetch() you have to call the response.json method to parse the data to a JavaScript object. 
* With Axios, the data response provided by the server can be accessed with in the data object, while for the fetch() method, the final data can be named any variable.
* ***redaxios*** uses fetch to implement axios so that it can be used both on client as well as server using axios api.

***useFetch.ts***

```JS
import axios from 'redaxios'

type FetchState = {
    data?:unknown
    error?:unknown
    isLoading:boolean
}

type FetchAction = {type:"success",payload:unknown} | {type:"failure",payload:unknown}

const fetchReducer = (state:FetchState,action:FetchAction):FetchState =>{
    switch(action.type) {
        case 'success':
            return {data:action.payload,error:undefined,isLoading:false}
        case 'failure':
            return {data:state.data,error:action.payload,isLoading:false}
    }
}

const initial = {data:undefined,error:undefined,isLoading:true}

const useFetch = (url:string) => {
    const [state,dispatch] = React.useReducer(fetchReducer,initial)

    React.useEffect (() => {
        axios.get(url)
        .then(res=>dispatch({type:"success",payload:res.data}))
        .catch(err=>dispatch({type:"failure",payload:err}))
    },[])

    return state
}
```

We need to fetch data from server it should not be in rendering since rendering is for pure functions and computed values, we can make use of useEffect.

Here we are using useReducer instead of useState is because it is pure and it stays out of the component.The good thing about useEffect in here is it is not using any set functions.Whenevr dispatch happens react notes the action which eventually triggers a re-render of the component. When re-render happens useReducer thinks there an action and applies the action by passing it to fetchReducer.

If we want to fetch data only once then we can give an empty array as dependency array to the useEffect. If we want to fetch data every time url changes then we need to give url to dependency array.useFetch will make sure that everytime url changes it willl fetch data with respect to that url.


```JS
export const Todos = () => {
    const {data:todos,error} = useFetch('/api/todos')

    //if error exists while fetching
    if(error) {
        return <Text>{JSON.stringify(error,null,2)}</Text>
    }

    // if fetching data is successful
    if(todos) {
    return <pre>{JSON.stringify(todos,null,2)}</pre>
    }

    return <h1>Loading...</h1>
}
```

> useFetch does not care about component mounting and unmounting and we can not say when it could fetch data from server mean while there is a chance that the component could unmount.Inorder to avoid that we can keep track of component mounting and unmounting

```JS
const useFetch = (url:string) => {
    const [state,dispatch] = React.useReducer(fetchReducer,initial)
    const [cancelled,setCancelled] = React.useState(false)

    React.useEffect (() => {
        if(cancelled) {
            return 
        }
        axios.get(url)
        .then(res=>{
            if(cancelled) {
                return 
            }
            dispatch({type:"success",payload:res.data})
        })
        .catch(err=>{
            if(cancelled) {
                return 
            }
            dispatch({type:"failure",payload:err})
        })

        return () => setCancelled(true)
    },[url])

    return state
}
```
> If the component get unmounted then setCancelled will be called then unnecessary dispatch won't happens and also when ever the url changes the previous action will get cancelled and will fetch again.

The callback function that useEffect takes should a synchronous function so we need to handle catch. if we are using async await then it would be good idea to use try catch.

```JS
export const Timer = () => {
    const [seconds,setSeconds] = React.useState(0)

    useEffect(() => {
        const id = setInterval(()=>{
            setSeconds(seconds+1)
        },1000)

        return () =>clearInterval(id)
    })

    return <h1>{seconds}</h1>
}
```
The above component incremets seconds by one for every 1000ms.For every 1000ms the setSeconds will be called and Timer would re-render which makes useEffect calls its callback function makes function clear its interval and sets new interval which leads to lost in few milli seconds.Imagine we have  another setState function irrespective of that the component will re-render which makes clears interval and set up a new interval.Inorder to avoid these problems we can have a dependency array.

```JS
     useEffect(() => {
        const id = setInterval(()=>{
            setSeconds(seconds => seconds+1)
        },1000)

        return () =>clearInterval(id)
    },[])
```

> Irrespective of how many setStates and re-renders the component have useEffect will only call its callback funtion once after the component being rendered into the browser.During the lifetime of component there is only one setInterval will be called and when it unmounts one clearInterval will be called.In the previous example the seconds is captured from outside this problem has also been fixed.

```JS
export const Timer = () => {
    const [seconds,setSeconds] = React.useState(0)
    const [step,setStep] = React.useState(1)

     useEffect(() => {
        const id = setInterval(()=>{
            setSeconds(seconds => seconds+step)
        },1000)

        return () => clearInterval(id)
    },[step])

    return(
        <>
            <h1>{seconds}</h1>
            <input onChange={(evt)=>setStep(Number(evt.target.value))}/>
        </>
    ) 
}
```

Imagine the interval has started and it is 250ms and then setStep has called and then 250ms are lost because we are clearing the previous interval and set up a new one and 1000ms unnecessarily calling setInterval and clear interval because of change in step.

```JS
const timerReducer = () => {
    switch(action.type) {
        case 'seconds':
            return {...state,seconds:state.seconds+state.step}
        case 'step':
            return {...state,step:action.step}
    }
}

export const Timer = () => {
    const [{seconds},dispatch] = React.useReducer(timerReducer,{seconds:0,step:1})

     useEffect(() => {
        const id = setInterval(()=>{
            dispatch({type:"seconds"})
        },1000)

        return () => clearInterval(id)
    },[])

    return(
        <>
            <h1>{seconds}</h1>
            <input onChange={(evt)=>dispatch({type:"step",step:Number(evt.target.value)})}/>
        </>
    ) 
}
```

Even though component renders multiple time because of dispatches it contains it doesn't matter the callback function will be called once there by single setInterval and when it is unmount then clear Interval will be called once.Instead of capturing the closure of state it is better to use function version of setState i.e useReducer.
