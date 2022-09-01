***useImmer***

```JS
export const Counter = () => {
    const [state,set] = useImmer({count:0})
    const onInc = () => {
        set(count=>count+1)
    }
    const onDec = () => {
        set(count=>count-1)
    }
    retrun (
        <Box>
            <Button onClick={onInc}>+</Button>
            <Text>{state.count}</Text>
            <Button onClick={onDec}>-</Button>
        </Box>
    )
}
```

> Using useImmer we can treat state as draft and set functions as produce.Without understanding concepts like immutability,purity,state and state transistions it would a bad idea to use useImmer.after having knowledge of these and understanding idea of redux then going and choosing useImmer is fine.Beacuse mutation is really bad idea acquiring these practices while in the begining of learning programming would lead to write bad code.

```JS
export const Counter = () => {
    const [state,set] = useState({count:0})
    const onInc = () => {
        set({count:state.count+1})
    }
    const onDec = () => {
        set({count:state.count-1})
    }
    retrun (
        <Box>
            <Button onClick={onInc}>+</Button>
            <Text>{state.count}</Text>
            <Button onClick={onDec}>-</Button>
        </Box>
    )
}
```

In the above case whatever useState returns that state is immutable where as identity changes in the next render.During the render what ever the state we have will not change event handlers retains state that is immutable (state) which leads to event handlers pointing to state which they are exactly supposed to point to.This defines immutability of code.But functions like setInterval captures the value from future which leads to mutation.

Inorder to enhance performance and optimization we use hook like useCallback,useEffect and react memo.Using them in the right place could be so beneficial but at the same time using them where they are not supposed could lead to problems.Suppose useCallback may reduce few re-renders what if it might not rendering something which it is supposed to.Rendering is something that comes naturally to react as we are controlling rendering few manually using useCallback and useEffect we need to be careful.We cannot sacrifice simplicity and correctness for the sake of performance.Inorder to have better understanding of it refer [premature optimization](https://medium.com/@ryanflorence/react-inline-functions-and-performance-bdff784f5578).

If the component does not make any side effects and any DOM manipulations only worried about state then we can simplify the code even more using custom reducerHook.

```JS
import {reducerHook} from './reducer'

const useCounter = () => reducerHook(
    {count:0}, 
    {
        onInc(state) {
         state.count++
        },
        onDec(state){
         state.count--
        }
    }
)

export const Counter = () => {
    const [state,actions] = useCounter()
    
    retrun (
        <Box>
            <Button onClick={()=>actions.onInc}>+</Button>
            <Text>{state.count}</Text>
            <Button onClick={()=>actions.onDec}>-</Button>
        </Box>
    )
}

```

***TodoApp***

***state.ts***
```JS
import { providerHook } from '@reducer'
import { createTodo, Filter, initialState, Todo } from '../todo'

export const { Provider, actions, useAction, useValue, useSelect } =
  providerHook(initialState, {
    createTodo(draft, payload: Todo) {
      const created = createTodo(payload)
      draft.todos.set(created.id, created)
    },

    deleteTodo(draft, payload: number) {
      draft.todos.delete(payload)
    },

    editTodo(draft, payload: Todo) {
      const editTodo = draft.todos.get(payload.id)
      draft.todos.set(payload.id, { ...editTodo, ...payload })
    },

    toggleTodo(draft, payload: number) {
      const toggleTodo = draft.todos.get(payload)
      if (toggleTodo) {
        toggleTodo.completed = !toggleTodo.completed
      }
    },
    setFilter(draft, payload: Filter) {
      draft.filter = payload
    },
  })
  ```
Here providerHooks uses reducerProvider which can typed with any specific context so it better to avoid create context at global level and provide it in reducerProvider.It returns back not only actions but dispatchable actions.providerHook wraps all the handlers nicely and provides them in actions.


***TodoList.tsx***
```JS
import {
  Button,
  ButtonGroup,
  Checkbox,
  Divider,
  Flex,
  Radio,
  RadioGroup,
  Stack,
  Table,
  Tbody,
  Td,
  Th,
  Thead,
  Tr,
} from '@chakra-ui/react'
import React from 'react'
import { Todo } from '../todo'
import { filteredTodosSelector, filterSelector } from './selectors'
import { actions, useAction, useSelect } from './state'
import { TodoForm } from './TodoForm'

export type TodoItemProps = Readonly<Todo>

export const TodoItem = React.memo(
  ({ completed, id, title }: TodoItemProps) => {
    const { deleteTodo, toggleTodo } = actions

    const onDelete = useAction(() => deleteTodo(id))
    const onToggle = useAction(() => toggleTodo(id))

    console.count('todo-item: ')

    return (
      <Tr>
        <Td>{title}</Td>
        <Td>
          <Checkbox isChecked={completed} onChange={onToggle} />
        </Td>
        <Td>
          <ButtonGroup>
            <Button>Edit</Button>
            <Button onClick={onDelete}>Delete</Button>
          </ButtonGroup>
        </Td>
      </Tr>
    )
  },
)

```

Here actions are retrived from useProvider and each action should be used with useAction for easy linting and code reviewing process instaed we can aviod using useAction and use them directly without it.

As we are rendering quiet a few todos React.memo will not gives great difference as we are retriving millions of todos then it will come handy.As the main concept behind rendering component is synching value with ui does not do much with actions so here we are letting component to bring its own action all by it self without passing them as props.


***TodoList***
```JS
export const TodoList = () => {
  const [show, set] = React.useState(false)

  const filter = useSelect(filterSelector)
  const filtered = useSelect(filteredTodosSelector)

  const onSetFilter = useAction(actions.setFilter)

  return (
    <>
      <RadioGroup onChange={onSetFilter} value={filter}>
        <Stack direction="row">
          <Radio value="All">All</Radio>
          <Radio value="Completed">Completed</Radio>
          <Radio value="Incomplete">Incomplete</Radio>
        </Stack>
      </RadioGroup>

      <Divider orientation="horizontal" />

      <Flex direction="column">
        {show ? (
          <TodoForm />
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
              <Th>Actions</Th>
            </Tr>
          </Thead>
          <Tbody>
            {filtered.map(todo => (
              <TodoItem key={todo.id} {...todo} />
            ))}
          </Tbody>
        </Table>
      </Flex>
    </>
  )
}
```

Everytime we need a value we need to make use of useSelect.useSelect takes a function and return a value only when that returned value changes the component will re-render otherwise it will not.This will be extremely useful if we are using global state or provider based state.Making useage of useAction for actions and making usage of useSelect for state can make code more correctness and less prone errors.