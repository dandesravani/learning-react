```JS
const isProd = process.env.NODE_ENV === 'production'

//These default options will gonna apply to all the queries
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: isProd, 
      retry: isProd ? 3 : 0,
      staleTime: isProd ? 0 : 5 * 60 * 1000,
      useErrorBoundary: true,
      suspense: true,
    },
  },
})

const queryClientAtom = atom(queryClient)

const Fallback = ({ error, resetErrorBoundary }: any) => {
  return (
    <Heading color="red">
      {JSON.stringify(error, null, 2)}
      <Button onClick={() => resetErrorBoundary()}>Try again</Button>
    </Heading>
  )
}

export const TodoApp = () => {
  const { reset } = useQueryErrorResetBoundary()

  return (
    <ErrorBoundary onReset={reset} FallbackComponent={Fallback}>
      <QueryClientProvider client={queryClient}>
        <Provider initialValues={[[queryClientAtom, queryClient]]}>
          <TodoList />
        </Provider>
      </QueryClientProvider>
    </ErrorBoundary>
  )
}
```

* ***refetchOnWindowFocus*** is set to boolean value if it in production mode then only refetching happens.In development mode when you switch between vscode and browser or browser to vscode unnecessary fetching won't happen.
* If it in production mode then only retries happen otherwise ***retry*** will be zero.
* With in given time to ***staleTime*** it will not try to refetch until then it will consider that the cache is latest and not stale.
* setting ***useErrorBoundary*** true will make the component throw error instead returning that respective error and handling it.so that we don't have to worry about error that useQuery returns.
* setting ***suspense*** true allows useQuery to suspends rendering of a component until the data is available.So it does not need to return back with property of isLoading boolean.using useErrorBoundary and suspense allows us not to worry about error and isLoading.
* Here we are using ***react-error-boundary*** for handling errors that react-query throws.The Fallback function will be called when ever any kind of exception happens in any of the rendering of the component. 
* JSON Server is a Node Module that you can use to create demo rest json webservice in less than a minute. All you need is a JSON file for sample data.More info [here](https://www.npmjs.com/package/json-server)

```JS
const useTodoList = () => {
  const [limit] = React.useState(15) //limited Todos per page 
  const [filter, setFilter] = React.useState<Filter>('All')
  const [page, setPage] = React.useState(1) //current page is being maintained here

  const { data } = useTodos()

  const filtered = React.useMemo(
    () => (data === undefined ? undefined : filteredTodos(data, filter)),

    [data, filter],
  )

  const todoList = React.useMemo(
    () => (filtered === undefined ? undefined : paged(filtered, page, limit)),
    [filtered, page, limit],
  )

  const { deleteTodo, toggleTodo } = useTodoMutations()

  const pc = pageCount(filtered?.length || 0, limit)

  React.useEffect(() => {
    if (page > pc) {
      setPage(pc)
    }
  }, [page, pc])

  const onFilterChange = React.useCallback(
    (s: string) => setFilter(s as Filter),
    [],
  )

  return {
    todoList,
    pageCount: pc,
    page,
    onPageChange: setPage,

    filter,
    onFilterChange,
    onDelete: deleteTodo.mutate,
    onToggle: toggleTodo.mutate,
  } as const
}

export const TodoList = () => {
  const {
    todoList,
    pageCount,

    filter,
    page,

    ...actions
  } = useTodoList()

  invariant(todoList !== undefined, 'todoList is undefined')

  return (
    <Flex direction="column" h="100vh" p="5">
      <FilterView filter={filter} onFilterChange={actions.onFilterChange} />

      <Box flexGrow={1}>
        <TodoListView
          todoList={todoList}
          onDelete={actions.onDelete}
          onToggle={actions.onToggle}
        />
      </Box>

      <Pagination
        current={page}
        pageCount={pageCount}
        onPageChange={actions.onPageChange}
      />
    </Flex>
  )
}
```

***React.useMemo*** takes a function and a dependency array.The value being returned from callback function is derived value and being values in dependency array are reactive variables.

Everytime reactive variables change the callback function is called and new value will be returned back.If these values doesnot change useMemo returning back the same value as it is already stored.If we are having computation which is costly its better to use useMemo otherwise rendering will not be cheap.

Unlike useCallback and useEffect, useMemo is much safer to use because it willnot capture value from closure.It only takes pure function as captured values are already in dependency array. 

Unlike queries, mutations are typically used to create/update/delete data or perform server side-effects. For this purpose, React Query exports a ***useMutation*** hook.

useMutation hook needs to be passed function which performs the mutation(delete/put/post) along with that we need to pass an object which consists of properties like onSuccess,onError and onSetteled similart to try,catch and finally.

If query was successful then useQuery can ***invalidate queries*** inorder to fetch the data with exact mutations we applied on it.Once the fetching is successful useQuery makes the query invalid and tries to re-renders the component and fetches the data with reflected changes.