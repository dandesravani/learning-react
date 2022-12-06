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

filteredTodosAtom will be computed only when todosAtom or filterAtom are changed which is equivalent to useMemo() here memoization will be done automatically using selectors.
filteredAtom is derived value which is computed when ever filterAtom and todosAtom changes.Atoms are tracked by their identity.In add function we create a single atom but there would be some case where we need to create
multiple atoms in that case it would be better to keep state globally.

**_Disadvantages of manipulating DOM directly:_**

> - Imagine there is no react and we have to change the DOM directly to mutate ui being DOM as mutative library changing multiple things will lead to difficulty in managing state.
> - mutating one thing at one place may lead to mutation in another place which leads to difficulty keeping synchronization between UI and program logic.There is no easy way changing HTML and CSS without effecting the programmer.
> - Make sure that the query selectors are as general as possible and do not break easily.
> - Imagine there are lot of DOM objects as well as functions for manipulating DOM and each object is maintaining its own state.State is distributed among the objects and each object is pointing to another object and each object accessing a global variable then state would be unpredictable.

**_How React solves the problem:_**

- A single page can be divided into multiple components.Props can be distributed to these multiple components eventually lead to building VDOM.
- Imagine each component is a pure function being fed with props each function render a VDOM which leads to more predictable code. Even if a bug occurs we can easily identify the respective component and debug it.
- This mental model is so unrealistic where as keeping every component is pure in order to change UI we need to change the props.
- React solves this by keeping rendering components pure at the same time keeping state explicitly.

## **Reactive Programming:**

**_Reactive variables:_**

> - variables contains a reference to immutable values . we can always get a current value from the variable and also can provide with previous value if its been stored might get a next value.Each one of these values are immutable. Anybody accessing these immutable values cant change these values.
> - Whenever the variable value changes there should a mechanism that we can poll the that change.These read and write values.
> - Those variables which always refers to a value which have an identity which can tell others change in their value with their time are called Reactive variables.
> - Every time we call useState we are creating a reactive variable except the fact that we encapsulate that variable into the component.
> - In external system all the reactive variables are updated using hanlders and useEffect but its better to prefer handlers which are invoked once and more predictable.
> - Jotai have integrated the same system as react where setter functions are called as part of external system.
> - setter functions are involve in wright only atoms. write only atoms are not part of reactive programming. write only atoms are nothing but actions.write only atom writes to an existing atom.
> - In jotai we do not have the access to actual value we get it from get function.That is about writable atom is no way to get value but a way to set it as shown in incAtom.

**_Derived values:_**

- Derived values are values which are dependent on reactive variable.
- Derived values can be obtaining a value from providing reactive variable to a pure function.
- Reactive variable can not contain another reactive variable but derived variable can be derived from another derived variable or reactive variable.
- By keeping reactive variables as minimal as possible and independent with each other and make it as a seperate layer with derived variables to extent solve the problem of inconsistency.
- Rendering React components is computation and is computed based on state.

// Suspense

- At earlier suspense is only for dynamically loaded components/lazily loaded components but now suspense is for asynchrony.Most of the web applications are dependent on asynchorny.Rendering applications using concurrency and asynchrony is done using suspense.
