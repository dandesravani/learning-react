**Attaching state to UI**

> We can make use of producer once instead of using it multiple places where state needs to manipulated.

```JS
import z from 'zod'
import {produce} from 'immer'

export const Todo  =  z.object({
    id:z.number(),
    title:z.string(),
    completed:z.string()
})

export const State = z.map(z.string(),Todo)
export type State = Readonly<z.infer<typeof TodoState>>

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

export const todoReducer = produce((draft:State,action:TodoAction):TodoState => {
    switch(action.type){
        case 'CreateTodo':
                const todo = createTodo(action.todo) 
                draft.set(todo.id,todo)
                break

        case "DeleteTodo": 
                draft.delete(action.id)
                break

        case 'EditTodo':
                const todo = draft.get(action.todo.id)
                draft.set(action.todo.id,{...todo,...action.todo})

        case "ToggleTodo":
                const todo = draft.get(action.id)
                if(todo) {
                    todo.completed = !todo.completed
                }
            
}
})
```
***TodoItem.tsx***

```JS
import {
  Button,
  ButtonGroup,
  Checkbox,
  Divider,
  Flex,
  Table,
  TableContainer,
  Tbody,
  Td,
  Th,
  Thead,
  Tr,
} from '@chakra-ui/react'
import React from 'react'
import { Todo } from '../state'
import { AddTodoForm } from './CreateTodoForm'
import { todos } from './data'
import { state } from './state'

type TodoItemProps = {
  readonly todo: Todo
  // onCreate(todo: Omit<Todo, 'id'>): void
  onDelete(id: Todo['id']): void
  // onUpdateTitle(id: Todo['id'], todo: Todo): void
  onToggle(id: Todo['id']): void
}

export const TodoItem = ({
  todo,
  // onCreate,
  onDelete,
  onToggle,
}: // onUpdateTitle,
TodoItemProps) => {
  return (
    <Tr>
      <Td>{todo.title}</Td>
      <Td>
        <Checkbox
          isChecked={todo.completed}
          onChange={() => onToggle(todo.id)}
        />
      </Td>
      <Td>
        <ButtonGroup>
          <Button>Edit</Button>
          <Button onClick={() => onDelete(todo.id)}>Delete</Button>
        </ButtonGroup>
      </Td>
    </Tr>
  )
}
```

***TodoList.tsx***

```js
type TodoListProps = {
  todoList:readonly Todo[]
  onDelete(id: Todo['id']): void
  onToggle(id: Todo['id']): void
//   onCreate(todo: Omit<Todo, 'id'>): void
}

export const TodoList = ({ todoList,onDelete, onToggle, onCreate }: TodoListProps) => {
  const [show, set] = React.useState(false)
  return (
    // <Flex direction="column">
    //   {show ? (
    //     <AddTodoForm onCreate={onCreate} onShow={set} />
    //   ) : (
    //     <Button alignSelf="flex-end" m="20px" onClick={() => set(!show)}>
    //       Add
    //     </Button>
    //   )}

    //   <Divider orientation="horizontal" />
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
              onDelete={onDelete}
              onToggle={onToggle}
            />
          ))}
        </Tbody>
      </Table>
    </Flex>
  )
}

export const initialState:State = todoList.reduce((acc,v)=> {acc.set(v.id,v)
return acc},new Map())
```
> In React a component should understandable even if it is isolated.It should be reusable. To make it one component needs to take props instead of not taking anything.

While writing a React components there are few steps that needs to be followed.
* Always start with a static structure of the component.
* Once it is done, make sure the component is pure for same set of props it should return same view.
* Then make the components stateful where ever the state is required.
* Components should render and that render needs to be pure.Inorder to achieve that we should represent state as transition between values i.e implementation of transition function so that no values are changed.
* Hooks allows us to connect that derived value with reactive variable.one of the hook is useReducer.


***App.tsx***

```JS
 export const App = () => {
    const [todoList,dispatch] = useReducer(todoReducer,initialState)
    const handleDelete = (id:number) => {
        dispatch({type:"DeleteTodo",id})
    }
    const handleToggle = (id:number) => {
        dispatch({type:"ToggleTodo",id})
    }
    return (
        <TodoList todoList={Array.from(todoList.values())} onDelete={handleDelete} onToggle={handleToggle}/>
    )
 }
 ```

**State mangement using valtio:**

***state.tsx***

```JS
import { proxy } from 'valtio'
import { todos } from './data'
import { Todo } from '../state'

let nextId = 100

export const state = proxy({
  todos,
  onCreate: (todo: Todo) => {
    state.todos.push({ ...todo, id: nextId++ })
  },
  onToggle: (id: number) => {
    const foundIdx = state.todos.findIndex(t => t.id === id)
    const foundTodo = state.todos[foundIdx]
    if (foundTodo) {
      foundTodo.completed = !foundTodo.completed
    }
  },
  onDelete: (id: number) => {
    const todoIdx = state.todos.findIndex(t => t.id === id)
    if (todoIdx !== -1) {
      state.todos.splice(todoIdx, 1)
    }
  },
  onUpdateTitle: (id: number, title: string) => {
    const found = state.todos[id]
    if (found) {
      found.title = title
    }
  },
})
```

***TodoList.tsx***
```JS
import {
  Button,
  ButtonGroup,
  Checkbox,
  Divider,
  Flex,
  Table,
  TableContainer,
  Tbody,
  Td,
  Th,
  Thead,
  Tr,
} from '@chakra-ui/react'
import React from 'react'
import { useSnapshot } from 'valtio'
import { Todo } from '../state'
import { AddTodoForm } from './CreateTodoForm'
import { todos } from './data'
import { state } from './state'

type TodoItemProps = {
  readonly todo: Todo
  // onCreate(todo: Omit<Todo, 'id'>): void
  onDelete(id: Todo['id']): void
  // onUpdateTitle(id: Todo['id'], todo: Todo): void
  onToggle(id: Todo['id']): void
}

export const TodoItem = ({
  todo,
  // onCreate,
  onDelete,
  onToggle,
}: // onUpdateTitle,
TodoItemProps) => {
  return (
    <Tr>
      <Td>{todo.title}</Td>
      <Td>
        <Checkbox
          isChecked={todo.completed}
          onChange={() => onToggle(todo.id)}
        />
      </Td>
      <Td>
        <ButtonGroup>
          <Button>Edit</Button>
          <Button onClick={() => onDelete(todo.id)}>Delete</Button>
        </ButtonGroup>
      </Td>
    </Tr>
  )
}

type TodoListProps = {
  onDelete(id: Todo['id']): void
  onToggle(id: Todo['id']): void
  onCreate(todo: Omit<Todo, 'id'>): void
}

export const TodoList = ({ onDelete, onToggle, onCreate }: TodoListProps) => {
  const [show, set] = React.useState(false)
  const snap = useSnapshot(state)
  return (
    <Flex direction="column">
      {show ? (
        <AddTodoForm onCreate={onCreate} onShow={set} />
      ) : (
        <Button alignSelf="flex-end" m="20px" onClick={() => set(!show)}>
          Add
        </Button>
      )}

      <Divider orientation="horizontal" />
      <Table variant="simple">
        <Thead>
          <Tr>
            <Th>Title</Th>
            <Th>Completed</Th>
          </Tr>
        </Thead>
        <Tbody>
          {snap.todos.map(todo => (
            <TodoItem
              key={todo.id}
              todo={todo}
              onDelete={onDelete}
              onToggle={onToggle}
            />
          ))}
        </Tbody>
      </Table>
    </Flex>
  )
}
```

***TodoApp.tsx***

```JS
import React from 'react'
import { TodoList } from './TodoList'
import { state } from './state'
import { useSnapshot } from 'valtio'

export const TodoPage = () => {
  const snap = useSnapshot(state)
  console.log(snap)

  return (
    <TodoList
      onDelete={snap.onDelete}
      onToggle={snap.onToggle}
      onCreate={snap.onCreate}
    />
  )
}
```




