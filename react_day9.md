**Todo App:**

***state.js*** <br/>
Define Todo using ***zod***

Zod is typeScript-first schema validation with static type inference. Zod is used to validate data that coming from server which provides typesafty.[Zod documentation](https://github.com/colinhacks/zod)

```js
import z from 'zod'

export const Todo  =  z.object({
    id:z.number(),
    title:z.string(),
    completed:z.string()
})

export const TodoState = z.array(Todo)
export type TodoState = Readonly<z.infer<typeof TodoState>>

export type Todo = Readonly<z.infer<typeof Todo>>

// create CreateTodo function type
export const CreateTodo(todo:Omit<Todo,'id'>) = Todo.omit({id:true})
export type CreateTodo = Readonly<z.infer<typeof CreateTodo>>

let nextId=100

export function createTodo(todo:Todo):Todo {
    return {...todo,id:nextId++}
}

// declare type for TodoApp actions
export type TodoAction = 
|{type:"CreateTodo",todo:CreateTodo}
|{type:"DeleteTodo",id:number}
|{type:"ToggleTodo",id:number}
|{type:"EditTodo",todo:Todo}

//todo reducer
export const todoReducer = (state:TodoState,action:TodoAction):TodoState => {
    switch(action.type){
        case 'CreateTodo':
            return {...state,createTodo(action.todo)}
        case "DeleteTodo": 
            return state.filter(t=>t.id===action.id)
        case 'EditTodo':
            return state.map(t=>t.id===action.todo.id ? 
                {...t,...action.todo}:t)
        case "ToggleTodo":
            return state.map(t=>t.id===action.id? 
                {...t,completed:!t.completed}:t)
}
}
```
***what is immer and how it works***

Immer is a small library created to help developers with immutable state based on a copy-on-write mechanism, a technique used to implement a copy operation on modifiable resources.

In Immer, there are 3 main states.

1. Current State: Actual state data
1. Draft State: All the changes will be applied to this state.
1. Next State: This state is produced based on the mutations to the draft state.

State is nothing a different value at different pointing of time.Its not like mutable data in the memory.

**changing TodoApp using immer**

```JS
import z from 'zod'
import {produce} from 'immer'

export const Todo  =  z.object({
    id:z.number(),
    title:z.string(),
    completed:z.string()
})

export const TodoState = z.array(Todo)
export type TodoState = Readonly<z.infer<typeof TodoState>>

export type Todo = Readonly<z.infer<typeof Todo>>

// create CreateTodo function type
export const CreateTodo(todo:Omit<Todo,'id'>) = Todo.omit({id:true})
export type CreateTodo = Readonly<z.infer<typeof CreateTodo>>

let nextId=100

export function createTodo(todo:Todo):Todo {
    return {...todo,id:nextId++}
}

// declare type for TodoApp actions
export type TodoAction = 
|{type:"CreateTodo",todo:CreateTodo}
|{type:"DeleteTodo",id:number}
|{type:"ToggleTodo",id:number}
|{type:"EditTodo",todo:Todo}

//todo reducer
export const todoReducer = (state:TodoState,action:TodoAction):TodoState => {
    switch(action.type){
        case 'CreateTodo':
            return produce(state,draft=>{
                draft.push(createTodo(action.todo))
            })

        case "DeleteTodo": 
            return produce(state,draft=>{
                const idx =  draft.findIndex(t=>t.id===action.id)
                if(idx !== -1){
                    return
                }
                draft.splice(idx,1)
            })

        case 'EditTodo':
            return produce(state,draft =>{
                const todo = draft.find(t=>t.id===action.id)
                if(todo) {
                    todo.title = action.todo.title
                    todo.completed = action.todo.completed
                }
            })

        case "ToggleTodo":
            return produce(state,draft=>{
                const todo = draft.find(t=>t.id===action.id)
                if(todo) {
                    todo.completed = =!todo.completed
                }
            })
}
}
```

>What immer does is creating a snapshot of actual state which is draft we actually mutating that snapshot instead of mutating the actual value.Once it is done the new snapshot will be returned from the reducer.

In the above TodoApp using immer, we are forced to use findIndex and find function everytime we need a specific todo.Instead of using ***TodoState as array we can use map***

```JS
export const TodoState = z.map(z.string(),Todo)
export type TodoState = Readonly<z.infer<typeof TodoState>>

export const todoReducer = (state:TodoState,action:TodoAction):TodoState => {
    switch(action.type){
        case 'CreateTodo':
            return produce(state,draft=>{
                const todo = createTodo(action.todo) 
                draft.set(todo.id,todo)
            })

        case "DeleteTodo": 
            return produce(state,draft=>{
                draft.delete(action.id)
            })

        case 'EditTodo':
            return produce(state,draft =>{
                const todo = draft.get(action.todo.id)
                draft.set(action.todo.id,{...todo,...action.todo})
            })

        case "ToggleTodo":
            return produce(state,draft=>{
                const todo = draft.get(action.id)
                if(todo) {
                    todo.completed = !todo.completed
                }
            })
}
}
```
We need to call ```enableMapSet()``` function from immer inorder to use map in immer.

In arrays and sets, the loop iterates over the values stored, and in maps, the loop iterates over each key-value pair which makes it more readable and easy to use.



