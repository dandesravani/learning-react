**Valtio:**

Valtio turns the object you pass it into a self-aware proxy.

```JS
import {proxy} from 'valtio'

const state = proxy({
    count:0
})
 ```
You can make changes to it in the same way you would to a normal js-object.

```JS
const inc = () => {
    state.count++
}

const dec = () => {
    state.count--
}
```

Create a local snapshot that catches changes. Rule of thumb: read from snapshots in render function, otherwise use the source. The component will only re-render when the parts of the state you access have changed, it is render-optimized.

```JS
export const Counter = () => {
    const snap = useSnapshot(state)
    return (
        <Box>
            <Button onClick={inc}>+</Button>
            <Text>{snap.count}</Text>
            <Button onClick={dec}>-</Button>
        </Box>
    )
}
```
* Valtio follows Reactive programming model where the object taken by proxy will be a reactive variable we can change the state of object directly but if we need to use a value then we need to use through useSnapshot.
* We can create as many Reactive variables as we want to and changing one variable won't be effecting  another.

```JS
import {proxy} from 'valtio'

const state2 = proxy({
    count:0
})

const inc2 = () => {
    state.count++
}

const dec2 = () => {
    state.count--
}
```
```JS
export const App = () => {
    return(
        <>
            <Counter1/>
            <Counter2/>
        </>
    )
}
```
The Above where changing state of one counter variable won't be effecting the state of another counter variable.        

**TodoApp using valtio**

***state.tsx***

```JS
import {proxyMap} from 'valtio/utils'

//MutTodo will be Todo without readonly
export const state = proxyMap<number,MutTodo>(initialState)

export function createTodo(todo:CreateTodo) {
    const created = create(todo)
    state.set(created.id,created)
}

export function deleteTodo(id:number) {
    state.delete(id)
}

export function editTodo(todo:Todo) {
    const editTodo = state.get(todo.id)
    state.set(todo.id,{...editTodo,...todo})
}

export function toggleTodo(id:number) {
    const toggleTodo = state.get(id)
    if(toggleTodo) {
        toggleTodo.completed =  !toggleTodo.completed
    }
}
```

>use useSnapshot from valtio to make use of state where ever it is needed.

* In Redux there always exists a Single state variable where in a case one component state does not depend on another why do keep it one variable.
* where in the case of Valtio we are allowed to create as many variables as we want to which leads to the advantange of maintaing state to component where it is needed.Where as useReducer allows us maintain state of bunch of components together.

> There is reactive variable it can have different values at different point of time.Every time value changes the derived values also need to be changed.All the components which are dependent upon reactive variables need to be updated.

How do we achieve the above scenerio using Redux, we will have one pointer it will be updated we won't mutate anything.whenever we need a new value we need to derive a new value based on existed value and can work with it.

There will be an action where in this action makes change in state from lets say A to B.Because change in B lot of other things can be updated.

Valtio works exact same way. where in it does change the mutate the actual state it clones the initial state and lets mutate it. Once the function is being called it will return that clone object as new state.

Everytime a derived value works with reactive variable it needs work with a value.That is what useSnapshot provides with.So that we can work with a derived value not the actual state.

> In valtio we can keep the state and state transitions are external to the react tree which is one of the advantage.

**React context**

Imagine if we want to transfer props from parent to child component irrespective of its position in the component tree we need to send props from parent to all the successive children components.This is called ***prop drilling.***

The testing and debugging of the each component will be easy using prop drilling technique.

The problem with this approach is that most of the components through which this data is passed have no actual need for this data.They are simply used as mediums for transporting this data to its destination component.

Inorder to avoid this prop drilling problem we can use useContext.

***context.tsx***

The Context API basically lets you broadcast your state/data to multiple components by wrapping them with a context provider. It then passes this state to the context provider using its value attribute. The children components can then tap into this provider using a context consumer or the useContext Hook when needed, and access the state provided by the context provider.

```JS
 export type ReducerContext = {
    state:State
    dispatch:Function
 }

const Context = React.createContext<ReducerContext | undefined>(undefined)

export const ReducerProvider = ({children}:any) => {
    const [state,dispatch] = React.useReducer(todoReducer,initialState)

    return (
        <Context.provider value={{state,dispatch}}>{children}</Context.provider>
    )
}

const useState = () =>{
    const ctx = React.useContext(Context)
    invariant(ctx!==undefined,"use ReducerProvider")

    return ctx.state
}

const useDispatch = () =>{
    const ctx = React.useContext(Context)
    invariant(ctx!==undefined,"use ReducerProvider")

    return ctx.dispatch
}
```

***App.tsx***

```JS
import {ReducerProvider} from './Context'

export const TodoApp = () => {
    return (
        <ReducerProvider>
            <TodoList/>
        </ReducerProvider>
    )
}
```

***TodoList.tsx***

```JS
import {useState} from './Context'

export const TodoList = () => {
 const state =  useState()
 const todoList = Array.from(state.values())

  return (
    <Flex direction="column">
      <Table variant="simple">
        <Thead>
          <Tr>
            <Th>Title</Th>
            <Th>Completed</Th>
          </Tr>
        </Thead>
        <Tbody>
          {todoList.map(todo => (
            <TodoItem
              key={todo.id}
              todo={todo}
            />
          ))}
        </Tbody>
      </Table>
    </Flex>
  )
}
```

***TodoItem.tsx***

```JS
import {useDispatch} from './Context'

type TodoItemProps = {
  readonly todo: Todo
}

export const TodoItem = ({
  todo,
}: 
TodoItemProps) => {
  const dispatch = useDispatch()
  return (
    <Tr>
      <Td>{todo.title}</Td>
      <Td>
        <Checkbox
          isChecked={todo.completed}
          onChange={() =>dispatch({type:"ToggleTodo",id:todo.id}) }
        />
      </Td>
      <Td>
        <ButtonGroup>
          <Button>Edit</Button>
          <Button onClick={() => dispatch({type:"DeleteTodo",id:todo.id})}>Delete</Button>
        </ButtonGroup>
      </Td>
    </Tr>
  )
}

```

