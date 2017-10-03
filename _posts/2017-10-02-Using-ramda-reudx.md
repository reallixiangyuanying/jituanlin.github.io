---
layout: post
title: "Using ramda-redux to simplify Redux code"
author: "Jituan Lin"
categories: Function Programming
tags: [documentation,sample]
image:
  feature: ramda-redux-normal.jpg
  teaser: ramda-redux-teaser.jpg
  credit:
  creditlink:
---

## What is **ramda-redux**  

[ramda-redux](https://github.com/jituanlin/ramda-redux) is a library for reduce **Redux** code.

github: [ramda-redux](https://github.com/jituanlin/ramda-redux)

npm: [ramda-redux](https://www.npmjs.com/package/ramda-redux)

sample:[ramda-redux-sample](https://github.com/jituanlin/ramda-redux-sample)

## How **ramda-redux** reduce *Redux* template code:
With **ramda-redux** 
  ```js
  //./reducer/index.js
  // create a reducer for createStore function
  import R from 'ramda'
  import {Pattern,Reducer,Matcher} from 'ramda-redux'

  const addNumberPattern = Pattern(
  	'addNumber',
  	(state, payload) => R.over(
  		R.lensProp('n'),
  		R.inc,
  		state
  	)
  )

  const reducer = Reducer(
  	{n: 0},
  	Matcher(
  		addNumberPattern
  	)
  )

  const store = createStore(reducer)
  ```

  ```js
  //./component/addNumber.js
  // create a React component
  import React from 'react'
  import {connect} from 'react-redux'
  import R from 'ramda'
  import {Action} from 'ramda-redux'
  import {Button} from 'antd'

  const Link = ({n, addNumber}) =>
  	<div style={{marginLeft:'200px',marginTop:'200px'}}>
  		<p>{n}</p>
  		<Button onClick={addNumber}        type="primary">addNumber</Button>
  	</div>

  export default connect(
  	R.pick(['n']),
  	dispatch => (
  		{
  			addNumber: () => dispatch(Action('addNumber'))
  		}
  	)
  )(Link)
  ``` 

## The *ramda-redux* way of organizing code

0. Using `Pattern` factory function to create some instances of **Pattern**.

1. Passing this instances(pattern) to `Matcher` factory function.

2. Passing the instance of `Matcher` to Reducer factory function, the return is a function that satisfy for `Redux.createStore`.

3. Using `connect` same as traditional `Redux` way


## *ramda-redux* way VS traditional way of *Redux* 

* Using `Pattern` for handle action from `store.dispatch(someAction)` rather than the Javascript keyword `switch`. In this way, we could combine `reducer` without `combineReducers`,more concise. Consider follow code:
  ```js
  //./redux/pattern/increase.js
  export default Pattern(
    'increase',
    (state,{incremental})=>R.over(
      R.lensProp('number'),
      R.add(incremental),
      state
    )
  )

  //.redux/pattern/reduce.js
  export default Pattern(
    'reduce',
    (state,{reduction})=>R.over(
      R.lensProp('number'),
      R.add(reduction),
      state
    )
  )

  //.redux/index.js
  import increasePattern from './pattern/increase'
  import reducePattern from './pattern/reduce'  

  export default Reducer(
    {n:0},
    Matcher(
      increasePattern,
      reducePattern
    )
  )
  ```

* Using `Action` factory function to **new** a action, and specify the `action` must has unite structure that:
  ```js
    {
      type: 'someType',
      payload:someData
    }
  ```
So we cound use **ramda** pure function generate a new state conveniently.

* **ramda-redux** API more friendly using **ramda** pure function to generate a new state.

  
  