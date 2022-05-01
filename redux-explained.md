# Redux Explained

## Intro
During my conversations with junior- to mid level frontend developers, especially react developers, redux is brought up as a topic of discussion. How could it not, since redux and react where seen together in pair for a long time. Still, even today if you hear redux, you will probably think react. I do!
During this conversations redux is being treated like a magic monster that you are not supposed to upset. Every part of it is treated like a magic thing. With complex and dark rules. The knowledge about those parts are only known to the selected and chosen people. The one who knows how to bend those parts to his will, is treated with respect among junior- and mid developers.
I would like to break this magic and trivialize this monster so no developer should be afraid of it.

## Part One: Subscription
You can trigger rerender of a react component from the outside of the component.
Take a look at this code snippet:  
```jsx
let state = 0;
let callback = null;

const interval = setInterval(() => {
  if (typeof callback === "function"){
    callback(state + 1);
  }
}, 1000);

function App(){
  const [value, setValue] = useState();
  callback = setValue;
  
  return <div>
    {value}
  </div>
};
```
Although the purpose of this piece is questionable, it does bring the main point across:
You can force a rerender of a component. There is no logic inside the component and still the value is being updated every second.
You can implement any other logic if you feel like it.

Now lets make this piece more generic:
```jsx
import { useEffect } from "react";

let state = 0;
let callbacks = [];

function subscribe(callback) {
  // I do not provide a unsubscribe function yet, feel free to do it yourself
  callbacks.push(callback)
}

const interval = setInterval(() => {
  // instead of one callback we will have unlimited amount of them 
  callbacks.forEach((callback) => {
    if (typeof callback === "function") {
      callback(state + 1);
    }
  })
}, 1000);

function App() {
  const [value, setValue] = useState();
  
  useEffect(() => {
    subscribe((value) => {
      // this function will be called every 1000ms
      setValue(value)
    })
  }, []);

  return <div>
    {value}
  </div>
};
```
This way an unlimited amount of components can subscribe to our logic and be notified about an update.

## Part Two: actions
The setInterval brings the point across, but its kind of useless. Let's make it a little more useful. Instead of incrementing the
value through timeout, let's increment it as we desire:
```jsx
import { useEffect } from "react";

let state = 0;
let callbacks = [];

function subscribe(callback) {
  callbacks.push(callback)
}

function setState(newStateValue){
  state = newStateValue; 
  callbacks.forEach((callback) => {
      // set new state value and call all listeners with the new value 
    if (typeof callback === "function") {
      callback(state);
    }
  })
}

function increment(){
  setState(state + 1);
}

function App() {
  const [value, setValue] = useState();
  
  useEffect(() => {
    subscribe((value) => {
      // this function will be called on every increment
      setValue(value)
    })
  }, []);

  return <div>
    {value}
    <button onClick={increment}>increment</button>
  </div>
};
```
We can make the state variable as complicated as we want and providing our own logic functions (e.g.: increment) to work on it.
As long as we call the setState function at the end, we are good. All listeners will be notified.

Now instead of writing a lot of loose logic functions and potentially forgetting to call setState, we provide one single function
to handle all the logic we want: "dispatch"
```jsx
import { useEffect } from "react";

let state = 0;
let callbacks = [];

function subscribe(callback) {
  callbacks.push(callback)
}

function setState(newStateValue){
  state = newStateValue; 
  callbacks.forEach((callback) => {
      // set new state value and call all listeners with the new value 
    if (typeof callback === "function") {
      callback(state);
    }
  })
}

function dispatch(action){
  const {type, args} = action;
  let newState = null;
  switch (type){
    case 'increment':{
      newState = state + 1;
    }
    // provide more cases 
  }
  
  // set new state an notify listeners
  setState(newState);
}


function App() {
  const [value, setValue] = useState();
  
  useEffect(() => {
    subscribe((value) => {
      // this function will be called on every increment
      setValue(value)
    })
  }, []);

  return <div>
    {value}
    <button onClick={dispatch({type: 'increment'})}>increment</button>
  </div>
};
```
It would be nice not to think about the setState function at all: "reducer"
```jsx
import { useEffect } from "react";

let state = 0;
let callbacks = [];

function subscribe(callback) {
  callbacks.push(callback)
}

function setState(newStateValue){
  state = newStateValue; 
  callbacks.forEach((callback) => {
      // set new state value and call all listeners with the new value 
    if (typeof callback === "function") {
      callback(state);
    }
  })
}

function reducer(state, action){
  const {type, args} = action;
  switch (type){
    case 'increment':{
      return state + 1;
    }
    default: state;
  }
}

function dispatch(action){
  const newState = reducer(state, action);
  // set new state an notify listeners
  setState(newState);
}


function App() {
  const [value, setValue] = useState();
  
  useEffect(() => {
    subscribe((value) => {
      // this function will be called on every increment
      setValue(value)
    })
  }, []);

  return <div>
    {value}
    <button onClick={dispatch({type: 'increment'})}>increment</button>
  </div>
};
```
Now by calling the dispatch function and providing more cases in the reducer function we do not have to think about
the setState function. I think that's one of the magic things.

## Part Three: initial state
At this point it would be nice to be able to work with a more complicated state than a number.
See the defaultState in the reducer, state = undefined and the initial call of the dispatch function.
```jsx
import { useEffect } from "react";

let state = undefined;
let callbacks = [];

function subscribe(callback) {
  callbacks.push(callback)
}

function setState(newStateValue){
  state = newStateValue; 
  callbacks.forEach((callback) => {
      // set new state value and call all listeners with the new value 
    if (typeof callback === "function") {
      callback(state);
    }
  })
}

const defaultState = {
  counter: {
    currrentValue: 0, 
  }
}
function reducer(state = defaultState, action){
  const {type, args} = action;
  switch (type){
    case 'increment':{
      return {
        ...state,
        counter: {
          value: state.counter.value + 1
        }
      }
    }
    default: state;
  }
}

function dispatch(action){
  const newState = reducer(state, action);
  // set new state an notify listeners
  setState(newState);
}

dispatch({type: '__initState__'})

function App() {
  const [value, setValue] = useState();
  
  useEffect(() => {
    subscribe((value) => {
      // this function will be called on every increment
      setValue(value)
    })
  }, []);

  return <div>
    {value}
    <button onClick={dispatch({type: 'increment'})}>increment</button>
  </div>
};
```
By calling dispatch once initially, state of type undefined will be passed to the reducer, and it will default to the passed
default state. And notify all listeners about the new state resulting in one initial rerender.

## Part Four: createStore
Not to keep all the function hanging in the global scope, lets wrap it in to a function: "createStore"
```jsx
import { useEffect } from "react";

function createStore(reducer){
  let state = undefined;
  let callbacks = [];

  function subscribe(callback) {
    callbacks.push(callback)
  }

  function setState(newStateValue){
    state = newStateValue;
    callbacks.forEach((callback) => {
      // set new state value and call all listeners with the new value 
      if (typeof callback === "function") {
        callback(state);
      }
    })
  }
  
  function dispatch(action){
    const newState = reducer(state, action);
    // set new state an notify listeners
    setState(newState);
  }

  dispatch({type: '__initState__'})
  
  return {
    dispatch,
    subscribe
  }
}

function reducer(state = defaultState, action){
  const {type, args} = action;
  switch (type){
    case 'increment':{
      return {
        ...state,
        counter: {
          value: state.counter.value + 1
        }
      }
    }
    default: state;
  }
}

const defaultState = {
  counter: {
    currrentValue: 0,
  }
}
const store = createStore(reducer);

function App() {
  const [value, setValue] = useState();
  
  useEffect(() => {
    store.subscribe((value) => {
      // this function will be called on every increment
      setValue(value)
    })
  }, []);

  return <div>
    {value}
    <button onClick={store.dispatch({type: 'increment'})}>increment</button>
  </div>
};
```
Imagine we move this function in to a separate file or even better to one npm package.
The only thing the user would get is the createStore function. As long as the user does not look in to the createStore function
everything works like magic.
Depending on the project size, the user would probably never see and know about the createStore
function. He would from time to time write some new cases in the reducer, which is in its own file, separated from 
everything else and that's about it. This fact adds more mystery to the magic. But as you can see its really simple.

## Part Five: utils
Couple things are still missing though. For example the combineReducers function:
```js
function combineReducers(reducersMap /* {[reducerKey: string], reducerFunction: (state, action) => state} */){
  return (state, action) => {
    return Object.entries(reducersMap).reduce((finalState, [reducerKey, reducerFunction]) => {
      return {
        ...finalState,
        // for every key in the reducersMap, take the state value by the key and pass it to the
        // reducer under the same key.
        [reducerKey]: reducerFunction(state?.[reducerKey], action)
      }
    }, {})
  }
}

const counterReducer = (state = {value: 0}, action) => {
  // any logic goes here
  return state
};

const rootReducer = combineReducers({
  counter: counterReducer,
  
  // You also can do this. (if you really need to)
  user: combineReducers({
    name: nameReducer,
    surname: combineReducers(/* surnameReducer */(surname, action) => surname)
  }),
})

const store = createStore(rootReducer);
```
There are more helper functions like this. I will not go in to all of them.

## Part Six: enhancer
So far there is one more thing missing. It's the enhancer function with is passed optional and is passed as the second argument
to the createStore function:
```js
function createStore(reducer, enhancer){
  // [...] 
  let state = undefined;
  if (typeof enhancer === "function"){
    // if enhancer is passed, deligate the creation of the store to this function
    return enhancer(createStore)(reducer, state)
  }
  // [...] 
}

// [...]
const logEnhancer = (createStore) => {
  return (reducer, state) => {
    // use the default createStore to create a normal store
    const store = createStore(reducer, state);
    // return a store object
    return {
      ...store,
      // override the normal dispatch function 
      dispatch(action){
        console.log(action);
        // deligate the call
        return store.dispatch(action);
      }
    }
  }
}

const store = createStore(reducer, logEnhancer)
```
The enhancer function can be understood as an "extend" in the OOP but in the functional world.
It takes the original object (a function in this case), overrides and/or extends it with its own functions.

This is the basic idea of redux. A lot of the original implementation contains error and concurrency handling. I do not provide it here.
Otherwise, it's much harder to understand what is happening.


## Part Seven: react-redux
Although I have started with a react component, redux has nothing to do with react.
I can't emphasise this point hard enough. Some developers fight me hard on this. You can use redux everywhere you see fit.
Its just that redux got popular with react. Now a lot of developers associate redux with react only.

There is a separate package called "react-redux" which connects react with redux in a nice way:

The hooks first.
```jsx
import { createContext, useContext } from "react";

const ReduxContext = createContext({});

const useStore = () => useContext(ReduxContext);
const useDispatch = () => {
  const store = useContext(ReduxContext);
  return (action) => store.dispatch(action); 
}
const useSelector = (selector) => {
  const store = useContext(ReduxContext);
  return selector(store.getState());
}

const Provider = ({ store, children }) => {
  return <ReduxContext.Provider value={store}>{children}</ReduxContext.Provider>
};

// [...]
const store = createStore(reducer);

// [...]
<Provider store={store}>
  <App />
</Provider>
```
In the nutshell this the implementation of the react-redux hooks. Error handling and most importantly memoization is missing here.
The base idea is kind of straight forward and does not need an explanation.

The connect function:
```jsx
const connect = (mapState, mapDispatch) => {
  return (Component) => {
    return (props) => {
      const store = useStore(); 
      // memoization and batching is missing here!
      const mappedState = mapState(store.getState());
      const mappedDispatch = mapDispatch(store.dispatch);
      
      <Component {...props} {...mappedState} {...mappedDispatch}/>
    }
  }
}
// [...]
const mapState = (state) => {
  return {
    name: state.name
  }
}
const mapDispatch = (dispatch) => {
  return {
    setName: (name) => dispatch({type: 'setName', args: [name]})
  }
}
const withUserName = connect(mapState, mapDispatch);
const WithUserStateComponent = withUserName(MyComponent); 
```

That's kind of it. Nothing more. I hope that I could de-mystify redux a little more for you.
Now you should be able to create your own state management, if you want. Or bring any other logic in to react.
Simply provide a subscribe function and notify react, when it should rerender. 




