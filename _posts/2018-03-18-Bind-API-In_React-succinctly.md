---
layout: post
title: "Bind API in React succinctly"
author: "Jituan Lin"
categories: frontEnd
tags: [Programming,FrontEnd,React,Functional-Programming]
image:
  feature: Bind-API-in-React-succinctly.jpeg
  teaser: Bind-API-in-React-succinctly.jpeg
  credit: Death to Stock Photo
  creditlink: ""
description: "Welcome to Jituan's Blog!"  
---

## Bind some API in React one line
In React, for communicate with server and then render in browser,we need do something like step by step list bellow:
1. Write a function that initiate a request, in ES6, it likely to return a Promise then will fulfil when server response.
2. Call it from React component, and set the flag by *setState* function  for indicate the request is emit and the result is *pendding*.
3. Chian the operate of handle result with *Promise.then* function, and then set the flag to *isFulfil* or *isError* for indicate the request state.
4. Write *Render* function that render result according to state flag and data.
5. Call the function in *componentDidMount* or whenever you want.

## We need do it more succinctly
As mentioned above, for most API we will write code according to the above steps, why we not write a function that automatically do that for us? Just don't repeate by yourself! Futher more, use flag to indicate state is so ugly, inspired by *Scala* [Try](http://www.scala-lang.org/api/2.9.3/scala/util/Try.html]) class, we could simulation [pattern match](https://docs.scala-lang.org/tour/pattern-matching.html) to do more succinctly. With the helper function, we could bind API in React one line:
```js
this.beStateful(fetchAnswerApi, 'result', true)
```

### First, let us simplify the request state express way
#### We use *State* class to represent the *http* result.
```js
class State {
    caseOf({
               Success: fnS,
               Failure: fnF,
               Pending: fnP
           }) {
        switch (this.constructor.name) {
            case 'Success':
                return fnS(this.data)
            case 'Failure':
                return fnF(this)
            case 'Pending':
                return fnP()
            default:
                throw new Error(`[pattern match error]: ${this.constructor.name} is not sub class of State`)
        }
    }
}
```
In above code, the *State* class has a method *caseOf*, we will use it for handle *http* result of different state like this:
```js
result.caseOf({
                Success: data => <span>{data}</span>,
                Failure: ({status, message}) => <p>[{status}]:{message}</p>,
                Pending: () => <bold>pending</bold>
             })
 ```
And then, we use three subclass *Success*, *Failure*, *Pending* for represent different *http* result. In *Ocaml* programming language or other language support *algebraic data types*, we could express their relationship as:
```Ocaml
type State = Success| Failure| Pending
```
The implemention(in *Javascript*):

```js
export class Failure extends State {
    constructor(option) {
        super()
        _.forEach(
            option,
            (value, key) => this[key] = value
        )
    }
}

export class Success extends State {
    constructor(data) {
        super()
        this.data = data
    }
}

export class Pending extends State {
}
```

#### Bind API in React with *State* class.
Now, we could use *State* class to represent *http* result, but how we use it for bind API in React? 
1. We convert a normal API function to a function that do somethig listed below:
    - When the *http* request emit, set the result to *Pending* by *React* *setState* method.
    - When the server response, set the result to *Failure* or *Success* according to response result.
2. Bind the converted function in React.
For implement it, we cound write a function for convert, then call it in *React* component *constructor*. 
**Note: for do this we need use *bind* method for let the *this* refer to current *React* instance.**
Or more succinctly, we extends the *React* *Component* for add this convert function:
```js
export class Component extends React.Component {
    //the convert method
    beStateful(api, propName, isEmitAfterMounted = false) {
        const setProp = val => this.setState({
            [propName]: val
        })
        this.state = {...this.state, [propName]: new Pending()}
        const method = (...args) => {
            R.apply(
                api,
                args
            )
                .then(
                    result => {
                        if (result.status > 200) return setProp(new Failure(result))
                        return setProp(new Success(result.data))
                    }
                )
        }
        // bind in to React
        this[`fetch${propName}`] = method
        // Should call it when mounted?
        if (isEmitAfterMounted) {
            if (api.length === 0) {
                throw new Error('[bind auto emit api(after mounted) failure]:the bind api has arguments')
            }
            const originComponentDidMount = this.componentDidMount
            this.componentDidMount = () => {
                originComponentDidMount && originComponentDidMount()
                method()
            }

        }
        return method
    }

}
```
### Finaly, use it like this:
```js
 const fetchAnswerApi = willError => new Promise(
     (res, rej) => setTimeout(
         () => willError ? res({status: 404, message: 'OH NO!'}) : res({status: 200, data: 42}),
         1000
     )
 )
 
 class ExampleComp extends Component {
     constructor(props) {
         super(props)
         this.beStateful(fetchAnswerApi, 'answer', true)
     }
 
     render() {
         return (
             <div>
                 {
                     this.state.answer.caseOf({
                         Success: data => <span>{data}</span>,
                         Failure: ({status, message}) => <p>[{status}]:{message}</p>,
                         Pending: () => <bold>pending</bold>
                     })
                 }
 
             </div>
         )
     }
```
Just only one function call to bind a API to React:
```
this.beStateful(fetchAnswerApi, 'answer', true)
```

That is all, thanks.


