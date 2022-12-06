**Suspense**

- Imagine their exists a parent component and it contains two children components and each component fetch data using useFetch
  concurrent react can choose to render both of them at a time.Imagine one component fetching data for profile and another component fetching data for todos to that associated profile
  in the synchrony world fetching todos first and then profile would be bad idea.
- In the above mentioned scenerio it would be better to embed one component into another so that fetch for profile component would happens first and
  after completing it fetch for second component will be happens and get rendered as they both does not start parllely.
- If we do not want to render both child components serially then extract both useFecths from child components keep them in parent component so that the parent will decide
  which component to render.
- Some advantages of server-side rendering include: Faster load time. A server-side rendered application speeds up page loading when the user suffers from a slow internet connection.
  Thus it greatly improves the whole user experience.
- One of the disadvantage of SSR is May cause slower page rendering. Server-side rendering is ideal for static HTML site generation, but in more complex applications, frequent server requests and full page reloads can result in overall slower page rendering.
- Few applications might only need SSR. Some of the application need client side rendering i.e why useQuery provides initial data which fetched from server where we can fetch data
  with few intervals.
- Fetching data in the parent component might not a good idea for complex applications imagine we have multiple child components and few of them rendered
  conditionally. Since keeping useFetch in parent component fetches data for all the children without needing it.
- For the above situations **_suspense_** allows pause the renderings for react. Interraptable renederings is provided by the react and it can
  be accessed through suspense.
- Rendering supposed to be pure and fast. React able to do both.If we instruct react to resume rendering then it stops current task and starts rendering immediatly.
- Components can be asynchrouns so that they can be called and paused and later can be resumed and returned a promise.This is what react try to achieve with suspense.

```JS
  <Box flexGrow={1}>
    <Suspense fallback={<Heading>Loading...</Heading>}>
          <TodoListComp />
    </Suspense>
  </Box>
```

- suspense make sure to aware other components that it has something got to do and wants to pause.If one of the child needs to load something might need to pause then
  fallback component will be shown instaed of child compoenents.
