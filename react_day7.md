**Data view of Programming:**
* Most often than not programming is all about data i.e Manipulating data, Tranforming data,Creating data.Interms of Database we keep data in the database retirve data from database manipulate and transform that through web servers/ micro services.
* often we dont pass in methods/ operations/   services we only pass data. The only way two services can communicate is through is data.
* An entire application is nothing but transforming data and data manipulations. Web browsers are work with data that obtain from server.
* Most often an application can be seen through either a code view or a data view. The more understanding we have on data view the more polymorphic our code / application can be. Eventually its all about code manipulation / data transformation not code manipulation.

**Why shared mutable state is a problem:**

>if two or more objects points to each other and imaging changing one object might effects the objects they were referencing to.

``` JS 
const pt = {x:1,y:2}
const rect = {top:pt,bottom:pt}

const pt.x =10
```
>Changing pt might chance the objects that it is pointing to i.e top,bottom and pt itself.

* Its difficult to predict the code if it shared and mutated.

Inorder to aviod shared mutation problem we need to follow some rules

* Aviod sharing by copying data.
* Prevent mutations by making data immutable.
* Aviod mutations by updating non-destructively.

**React being a single ownership model:**

```JS
const Counter = () => {
    const [count,set] = React.useState(0)
return (
    <span>
    <button onclick={()=>set(count+1)}>+</button>
    <div>{count}</div>
    <button onclick={()=>set(count-1)}>-</button>
    </span>
)
}

export const App = () => (
    <div>
    <Counter/>
    <h1>Hello world</h1>
    </div>
)
```
> After calling the App component React will try tranverse the whole element tree while it encounters the counter then it will render it. same with header component.

* Everytime set state called the component will be re-rendered and count value will be updtaed.

* In React each component own its own state which is called ***signgle ownership model***. In the above example Counter owns its own state nobody can change it and no chance of sharing it.There are no multiple pointers to it also.

* In most of the value based programming languages function will be created after calling it and after the value goes to unused the function memory will be garbage collected where in React the counter functions will be long lived. Though state is not shared among multiple other objects it is mutable and functions like counter can be long lasting in memory. 

 