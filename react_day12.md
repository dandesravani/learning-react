**State management**

* State management will be the complex part in web development.
* React is simple when it comes to rendering. Rendering should be pure. where most often than not rendering pure for pure components where as it is always just JSX with no complexity.
* Complexity lies in rendering components which are having side effects or the components with event handlers.Getting a value from outside world is difficult but integrating it in our state management is simple.

There are multiple options that provides state management:
1. Using Reactive variables / Reactive libraries 
    ex: jotai,recoil,valtio
2. Caching library
    ex: swr,tanstack-query
3. rxjs/ observable-hooks
4. useReducer
5. useState
6. xstate

Reactive variables are nothing but atoms from closure.If the state is way more complex then we can use xstate.

***How do we know which one to use in which secenerio***
* Honestly there will not be any best practices to follow in order to pick certain state management tool. This could vary from secenerio to secenerio. We really need to have a good understanding of what are different state management tools,what are the problems in our code etc.
No tool is perfect for every job.

>If you feel pressured to do things “the Redux way”, it may be a sign that you or your teammates are taking it too seriously. It’s just one of the tools in your toolbox, an experiment gone wild.<br/>
-***Dan Abramov***

* React is similar to c++ value based model.The only differentiation is if function gets called in c++ it starts and once it is done then it ends erased from the memory where as in case of components in react ther are going to stay in memory for longer amount of time we need to unmount them explicitly.
* The advantage of c++ value based model is its always have single owner where as the case in ***Redux***.
* Redux keeps all of the state in globally where state is being one singular atom.

The cases when a global store is really needed aren’t very common, but they still exist:

* When we need to share data between components that do not necessarily belong to the same subtree, but the data they share cannot be considered application data.
* When we need to persist some of the component’s state even after the component has left the screen.

Based on above scenarios we can not even conclude that we should use useState or local state always by considering following conditions.

* Imagine if we are fetching data from server and manipulating a value that comes from server will not be considered as state.
* Once the data is fetched ,component is unmounted and mounted again why do we need to fetch it again. So what we need in such situations is a ***caching library***.
  
In case of ***useReducer*** it is always gets its state explicitly it does cpature its state from closure. The more out of the rendering the better it is.

We lift up state to a common ancestor of components that need it so they can all share in the state. This allow us to move easily share state among all these components that need rely upon it. In this case keep state at global level will not be necessary using ***useState*** at its nearest parents component will do the job.

As we mentioned earlier they will not be any best practices and no tool is perfect for job always try to understand the problem from programmer perspective irrespective programming language you use and try to have a better idea on each and every tool to do the job.



