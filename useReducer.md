### useReducer

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

This code is using the `useReducer` hook in a functional React component. `useReducer` is similar to `useState`, but it allows for more complex state management.

The `useReducer` hook takes in three arguments:

* **reducer**: A function that takes in the current state and an action, and returns the next state. This function is called **every time an action is dispatched**.

* **initialArg**: The initial state of the component. This can be an object, array, or any other data type.

* **init**: (optional) A function that returns the initial state, which is only executed when the component is first rendered. It is useful when the initial state requires some computation or data loading from an API.

It returns an array containing the **current state and a dispatch function**. The state is the current state of the component and the dispatch function is used to update the state by dispatching an action, which will be **passed to the reducer function**.

The state and dispatch are destructured from the array returned by the useReducer hook.

An alternative to `useState`. Accepts a reducer of type `(state, action) => newState`, and returns the current state paired with a dispatch method. (If you’re familiar with Redux, you already know how this works.)

* `useReducer` is usually preferable to `useState` when you have **complex state logic** that **involves multiple sub-values** or when the **next state depends on the previous one**
* `useReducer` also lets you optimize performance for components that trigger deep updates because you can **pass dispatch down instead of callbacks**

### Counter example from useState section, rewritten to use a reducer

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

**Note:**  

React guarantees that `dispatch` function identity is stable and won’t change on re-renders. This is why it’s safe to omit from the `useEffect` or `useCallback` dependency list.

### Specifying the initial state

There are two different ways to initialize `useReducer` state. You may choose either one depending on the use case. The simplest way is to pass the initial state as a second argument:

```js
const [state, dispatch] = useReducer(
  reducer,
  {count: initialCount}
);
```

**Note:**  

React doesn’t use the `state = initialState` argument convention popularized by Redux. The initial value sometimes needs to depend on props and so is specified from the Hook call instead. If you feel strongly about this, you can call `useReducer(reducer, undefined, reducer)` to emulate the Redux behavior, but it’s not encouraged.

### Lazy initialization

You can also create the initial state lazily. To do this, you can pass an `init` function as the third argument. The initial state will be set to `init(initialArg)`.

It lets you extract the logic for calculating the initial state outside the reducer. This is also handy for resetting the state later in response to an action:

```js
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

### Bailing out of a dispatch

If you return the same value from a Reducer Hook as the current state, React will bail out without rendering the children or firing effects. (React uses the `Object.is` comparison algorithm.)

Note that React may still need to render that specific component again before bailing out. That shouldn’t be a concern because **React won’t unnecessarily go “deeper” into the tree**. If you’re doing expensive calculations while rendering, you can optimize them with `useMemo`.



