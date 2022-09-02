**react-query**

If we have got state at server on the server which is managed and manipulated over in server that is the single source of truth where as having state and manipulating the same in the client atfer fetching would be copy that state.

> Imagine we are having modal compenent and a boolean value as state inorder to show it or not where as that can be managed and manipulated at the client side.If something like that stays in the server there is no easy way of accessing it or manipulating it. Though we are fetching it from server by the time we get it there might be high chance that it is stale value because the value might have changed to new on the server.This is called ***distributed systems problem***.

* Web application are distributed where servers and multiple other clients are need to be sync they all can manipulate the same thing there should be single source of truth for the clients.
* There would be so many entities that could change the state in server and we can not keep track of all of them as server needs to manage and manipulate the state.
* So we can not treat the same as state in the client it should be ***cache***.Cache is nothing but a map which is key value pair.The key is identity with associated value.Identity places a important role here as we are not getting value based on key.
* If we are trying to access a value from server we need to fetch it using distant key which is nothing but url.
* After fetching data from server the cache could store it for some time and where ever the request happens with the same key then we don't have to fetch it from the server instead we can get it from cache.
* react-query is nothing but a cache-manager and is very optimized for accessing REST API.

***use-query***

A query is a declarative dependency on an asynchronous source of data that is tied to a unique key. A query can be used with any Promise based method to fetch data from a server. 

useQery takes two parameters those are,

* A unique key for the query
* A function that returns a promise that:
    * Resolves the data, or
    * Throws an error

```JS
import {
  QueryClient,
  QueryClientProvider,
  useQueryErrorResetBoundary,
} from '@tanstack/react-query'

const queryClient = new QueryClient()

export const TodoList = () => {
    const {data,isLoading,error} = useQuery(['todos'],()=>{
        return delay(3000).then(()=>{
            throw "Hello world"
        })
    })
    if(isLoading) {
        return <Text>Loading...</Text>
    }
    if(error) {
        return <Text>{error}</Text>
    }
    return (
        <pre>{JSON.stringify(data,null,2)}</pre>
    )
}

export const App = () => {
    return (
        <QueryClientProvider client={queryClient}>
            <TodoList />
      </QueryClientProvider>
    )
}
```

The unique key you provide is used internally for refetching, caching, and sharing your queries throughout your application.

React Query manages query caching for you based on query keys. Query keys can be as simple as a string, or as complex as an array of many strings and nested objects. 

Imagine you have visited a page and moved to another page when you got back to previous page which you have visited recently might show a spinner instead of loading it immediatly.The same will not happen with useQuery.

If you are fetching data from server using useQuery and loading it in our ui and moved to another page and got back to the previouly visited page data will be shown immediatly without any spinner i.e because useQuery caches the data with respect to key when you made the request with same key then the cache data will be returned.

By default, initialData is treated as totally fresh, as if it were just fetched. This also means that it will affect how it is interpreted by the staleTime option.

If you configure your query observer with initialData, and no ***staleTime*** (the default staleTime: 0), the query will immediately refetch when it mounts.

```JS
function Todos() {
  const result = useQuery(['todos'], () => fetch('/todos'), {
    queries: {
      refetchOnWindowFocus: false,
    },
    initialData: initialTodos,
    staleTime: 60 * 1000 
    cacheTime:10
    refetchOnMount:false
  })
}
```
***cacheTime*** is used when every component is having a perticular key is unmounted, still the data associated with that key will be retained after the component mounted with the same key.By default the cacheTime is 5sec. we can still change it.

When ***refetchOnMount*** was set to false any additional components were prevented from refetching on mount. In version 3 only the component where the option has been set will not refetch on mount.

```JS
  const result = useQuery(['todos'], () => fetch('/todos',()=>axios.get('/api/todos').then(res=>res.data)))
```

```JS
export const TodoItem = ({todo}:any) => {
    return(
        <li>{todo.title}</li>
    )
}

export const TodoListView = ({todos}:any) => {
    return (
        <ul>
            {todos.map(todo=>(
                <TodoItem todo={todo}/>
            ))}
        </ul>
    )
}

export const TodoList = () => {
    const {data,isLoading,error} = useQuery(['todos'], () => fetch('/todos',()=>axios.get('/api/todos').then(res=>res.data)))
    if(isLoading) {
        return <Text>Loading...</Text>
    }
    if(error) {
        return <Text>{error}</Text>
    }
    return (
        <TodoListView todos={data}/>
    )
}
```

Imagine we are fetching multiple thousands of data from server and we need to show render it in ui then there may be two options one is inifinite scrolling another is pagination.

If we had to choose between infinite scrolling and pagination it would always better idea to choose pagination because there are few problems associated with inifite scrolling those are:

* if we are scrolling for long amount of time we need manage that memory well otherwise lot of memory got wasted.
* We need to render all the elements at once though only few of them are visible in ui. 
* Resources,rendering and implementations are problems with infinite scrolling.
  
Though useQuery helps with both inifinite scrolling and pagination but it is much easier to get it right with pagination than infinite scrolling.

***pagination***

We can integrate pagination in useQuery by enabling query option and we can see that previous rendered will remain after disabling buttons.

```JS
  export const TodoList = () => {
  const [page, setPage] = React.useState(1)
  const {data,error,isLoading,isPreviousData} = useQuery(['todos',page], () => fetch('/todos',()=>axios.get(`/api/todos?_limit=50&_page=${page}`).then(res=>res.data),{keepPreviousData:true}))

    if(isLoading) {
        return <Text>Loading...</Text>
    }
    if(error) {
        return <Text>{error}</Text>
    }
    return (
        <Button disabled={isPreviousData} onClick={()=>setPage(page-1)}>Previous</Button>
        <Button disabled={isPreviousData} onClick={()=>setPage(page+1)}>Next</Button>
        <TodoListView todos={data}/>
    )
}
 ```
For each page the query will be different.Those queries are associated with the cache.If we visit the previous rendered page the data we can see how fast the data will load now you might think that it would be stale data but useQuery tries to refetch the data and replaces it.We can fine tune fetching and rendering using useQuery by enabling differnt query configure opyions.