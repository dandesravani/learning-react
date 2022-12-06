```JS
  import { Box, Button, Heading, Text } from '@chakra-ui/react'
import { atom, useAtomValue, useSetAtom } from 'jotai'
import React from 'react'

const counterAtom = atom(0)

const incAtom = atom(null, (get, set, _) => {
  const next = get(counterAtom) + 1
  set(counterAtom, next)
})

const decAtom = atom(null, (get, set, _) => {
  const next = get(counterAtom) - 1
  if (next >= 0) {
    set(counterAtom, next)
  }
})

const CounterView = () => {
  const count = useAtomValue(counterAtom)

  return (
    <Text fontSize="3xl" pl="5">
      {count}
    </Text>
  )
}

const CounterActions = () => {
  const inc = useSetAtom(incAtom)
  const dec = useSetAtom(decAtom)

  return (
    <Box>
      <Button onClick={inc}>+</Button>
      <Button onClick={dec}>-</Button>
    </Box>
  )
}

const Header = () => {
  React.useEffect(() => {
    console.log('header')
  })

  return <Heading textAlign="center">Jotai Course</Heading>
}

export const JotaiCounter = () => (
  <Box>
    <Header />
    <CounterView />
    <CounterActions />
  </Box>
)
```

- Here we have counterAtom which is n ordinary object.
- if there exists many setter functions and state is complicated then its better to crate a wright only atom.
- incAtom is nothing but a wrightonly atom which takes 2 parameters one is null nothing but a wright only second parameter is an function
  which takes 2 parameters i.e get and set functions where get is function which is used to get a value set is function to set a value for any atom.
- CounterView component will get re-rendered only when counter atom changed.
- useSetAtom is used to get the actions like incAtom and decAtom which are writeonly atoms.
- whenever we want to create a reusable components its better to create them as hooks rather than components.

**_Jotai Todo Example_**

```JS
import {
  ChakraProvider,
  Checkbox,
  CloseButton,
  Heading,
  HStack,
  Input,
  Radio,
  RadioGroup,
  VStack,
} from '@chakra-ui/react'
import type { PrimitiveAtom } from 'jotai'
import { atom, Provider, useAtom, useSetAtom } from 'jotai'
import type { FormEvent } from 'react'
import React from 'react'
import { a, useTransition } from '@react-spring/web'

type Todo = {
  title: string
  completed: boolean
}

const filterAtom = atom('all')
const todosAtom = atom<PrimitiveAtom<Todo>[]>([])

const filteredAtom = atom<PrimitiveAtom<Todo>[]>(get => {
  const filter = get(filterAtom)
  const todos = get(todosAtom)
  if (filter === 'all') return todos
  else if (filter === 'completed')
    return todos.filter(atom => get(atom).completed)
  else return todos.filter(atom => !get(atom).completed)
})

type RemoveFn = (item: PrimitiveAtom<Todo>) => void
type TodoItemProps = {
  atom: PrimitiveAtom<Todo>
  remove: RemoveFn
}
const TodoItem = ({ atom, remove }: TodoItemProps) => {
  const [item, setItem] = useAtom(atom)
  const toggleCompleted = () =>
    setItem(props => ({ ...props, completed: !props.completed }))

  return (
    <HStack>
      <Checkbox checked={item.completed} onChange={toggleCompleted}>
        <span style={{ textDecoration: item.completed ? 'line-through' : '' }}>
          {item.title}
        </span>
      </Checkbox>
      <CloseButton onClick={() => remove(atom)} />
    </HStack>
  )
}

const Filter = () => {
  const [filter, set] = useAtom(filterAtom)
  return (
    <RadioGroup onChange={set} value={filter}>
      <Radio value="all"> All </Radio>
      <Radio value="completed"> Completed </Radio>
      <Radio value="incompleted"> Incompleted </Radio>
    </RadioGroup>
  )
}

type FilteredType = {
  remove: RemoveFn
}
const Filtered = (props: FilteredType) => {
  const [todos] = useAtom(filteredAtom)
  const transitions = useTransition(todos, {
    keys: todo => todo.toString(),
    from: { opacity: 0, height: 0 },
    enter: { opacity: 1, height: 40 },
    leave: { opacity: 0, height: 0 },
  })

  return transitions((style, atom) => (
    <a.div className="item" style={style}>
      <TodoItem atom={atom} {...props} />
    </a.div>
  ))
}

const TodoList = () => {
  const setTodos = useSetAtom(todosAtom)

  const remove: RemoveFn = todo =>
    setTodos(prev => prev.filter(item => item !== todo))

  const add = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const title = e.currentTarget.inputTitle.value
    e.currentTarget.inputTitle.value = ''
    setTodos(prev => [...prev, atom<Todo>({ title, completed: false })])
  }

  return (
    <>
      <form onSubmit={add}>
        <Filter />
        <Input name="inputTitle" placeholder="Type ..." />
        <Filtered remove={remove} />
      </form>
    </>
  )
}

export function JotaiTodoApp() {
  return (
    <ChakraProvider>
      <Provider>
        <VStack>
          <Heading as="h1">Jōtai</Heading>
          <TodoList />
        </VStack>
      </Provider>
    </ChakraProvider>
  )
}
```

```JS
import { Draft } from 'immer'
import { useSetAtom, WritableAtom } from 'jotai'
import React from 'react'

export function useAction<Value, P extends any[]>(
  atom: WritableAtom<Value, Value | ((draft: Draft<Value>) => void)>,
  fn: (draft: Draft<Value>, ...args: P) => void,
) {
  const set = useSetAtom(atom)

  return React.useCallback((...args: P) => {
    set(draft => {
      fn(draft, ...args)
    })
  }, [])
}
```

- Here useAction takes two parameters one is action as draft and another one is atom from where we can extract the atom that atom is being
  passed to function we can make what ever changes to that atom and set as shown.

  ```JS
  import { Draft } from 'immer'
  import { atom, useAtomValue, useSetAtom } from 'jotai'
  import { atomWithImmer } from 'jotai-immer'
  import {
  createTodo,
  CreateTodo,
  Filter,
  initialState,
  State,
  Todo,
  } from '../todo'
  import { useAction } from './common'
  ```

const filterAtom = atom<Filter>('All')

export const useFilterValue = () => {
return useAtomValue(filterAtom)
}

export const useUpdateFilter = () => {
return useSetAtom(filterAtom)
}

const todosAtom = atomWithImmer(initialState.todos)
const useTodoAction = <P extends any[]>(
fn: (draft: Draft<State['todos']>, ...args: P) => void,
) => useAction(todosAtom, fn)

export const useCreate = () =>
useTodoAction((draft, todo: CreateTodo) => {
const created = createTodo(todo)
draft.set(created.id, created)
})

export const useDelete = () =>
useTodoAction((draft, id: number) => {
draft.delete(id)
})

export const useEdit = () =>
useTodoAction((draft, todo: Todo) => {
const editTodo = draft.get(todo.id)
draft.set(todo.id, { ...editTodo, ...todo })
})

export const useToggle = () =>
useTodoAction((draft, id: number) => {
const toggleTodo = draft.get(id)
if (toggleTodo) {
toggleTodo.completed = !toggleTodo.completed
}
})

const filteredTodosAtom = atom(get => {
const todoList = Array.from(get(todosAtom).values())
const filter = get(filterAtom)

return filter === 'All'
? todoList
: filter === 'Completed'
? todoList.filter(t => t.completed)
: todoList.filter(t => !t.completed)
})

export const useFilteredTodos = () => {
return useAtomValue(filteredTodosAtom)
}

- The above are the multiple actions that are implemented using atomWithImmer where we can implement mutable actions.
- filteredTodosAtom will be computed only when todosAtom or filterAtom are changed which is equivalent to useMemo() here memoization will be done automatically using selectors.
- In programming, memoization is an optimization technique that makes applications more efficient and hence faster. It does this by storing computation results in cache, and retrieving that same information from the cache the next time it's needed instead of computing it again.
- In simpler words, it consists of storing in cache the output of a function, and making the function check if each required computation is in the cache before computing it.
- We can make TodoList as performant as we can without using useMemo and useCallback.
