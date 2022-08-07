# custom-state-management
Alternative to redux and context, a custom global state management for React.js

---

I love diving deeper in the technologies I love, and getting a deper understanding of how things work "behind the scenes".
In the code below I created a custom state management system for react.

---

## 1. in our react project we add the below code, responsible for configuring the global store logic (e.g. in src/store/store.js)

```js
import { useState, useEffect } from "react";


let globalState = {};

let listeners = [];

let actions = {};


export const useStore = (shouldListen = true) => {
  // each component using this hook has acces to same state obj
  const [_, setState] = useState(globalState);

  useEffect( () => {
    
    // some components will use this hook only to dispatch actions. For them we don't add their
    // setState to the listeners array
    if (shouldListen) {
      // each component using this hook gets its own setState, we add it to the listeners array
      // in order to keep track of all components that need to rerender
      listeners.push(setState);
    }

    // when component using this hook unmounts, remove its setState from listeners array
    return () => {
      if (shouldListen) {
        listeners.filter(listener => listener !== setState);
      }
    }
  }, [shouldListen]);

  const dispatch = (actionName, payload) => {

    const newState = actions[actionName](globalState, payload);

    globalState = { ...globalState, ...newState };
    
    // when component using this hook updates the state, we update the state for all components consuming
    // this hook (we keep track of those components indirectly, i.e. by storing their setState functions
    // in the listeners array)
    listeners.forEach(listener => listener(globalState));
  }

  return [globalState, dispatch];
};


export const initStore = (userActions, initialState) => {
  // even if we create multiple stores, they are still merged into 1 store
  if (initialState) {
    globalState = { ...globalState, ...initialState };
  }
  actions = { ...actions, ...userActions };
};

```


## 2. We create a function for creating a custom store

Blueprint for the store creation function; we can create multiple stores, they will be merged into 1 store

``` js
function createSomeStore() {
  const actions = {};
  const state = {};
  initStore({}, {});
}
```


## 3. We instanciate the store in index.js; no wrapper component required

``` js
// store is created here and accessible through the entire app
// we don't need to wrap it around <App />
createSomeStore();
```


### EXAMPLE:

## Creating a custom store function, based on step 2
``` js
export const createCounterStore = () => {

  const state = { count: 0 };

  const actions = {

    INCREMENT(state, amount) {
      return { count: state.count + amount };
    },
    
    DECREMENT(state, amount) {
      return { count: state.count - amount };
    },
    
    RESET() {
      return { count: 0 };
    }

  };

  initStore(actions, state);
}
```

## Creating a custom component that will consume the useStore hook
``` js
import { useStore } from "../store/store";

const Counter = () => {

  const [ state, dispatch ] = useStore();
  const { count } = state;

  const handleIncrement = () => {
    dispatch("INCREMENT", 4);
  };

  const handleDecrement = () => {
    dispatch("DECREMENT", 1);
  }

  const handleReset = () => {
    dispatch("RESET");
  }

  return (
    <div>
      <h2>{count}</h2>
      <button onClick={handleDecrement}>Decrement by 1</button>
      <button onClick={handleReset}>Reset</button>
      <button onClick={handleIncrement}>Increment by 4</button>
    </div>
  )
};

export default Counter;
```


## Using our custom component multiple times in <App />

```js
import Counter from "./components/Counter";

function App() {
  return (
    <div className="App">
      <Counter />
      <Counter />
      <Counter />
    </div>
  );
}

export default App;

```

## Instanciating store in Index.js

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

import { createCounterStore } from './store/store-expamples';

// store is created here and accessible through the entire app
// we don't need to wrap it around <App />
createCounterStore();

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```
