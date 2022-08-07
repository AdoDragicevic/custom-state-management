# custom-state-management
Alternative to redux and context, a custom global state management for React.js

---

I love diving deeper in the technologies I love, and getting a deper understanding of how things work "behind the scenes".
In the code below I created a custom state management system for react.

---

## in our react project src/ we add store/ and configure the base logic for our global state management in store.js

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
