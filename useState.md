### useState

```js
const [state, setState] = useState(initialState);
```

Returns a stateful value, and a function to update it.

During the initial render, the returned state (`state`) is the same as the value passed as the first argument (`initialState`).

The `setState` function is used to update the state. It accepts a new state value and **enqueues a re-render of the component**.

```js
setState(newState);
```

During subsequent re-renders, the first value returned by useState will always be the most recent state after applying updates.

**Note**: React guarantees that setState function identity is stable and will not change on re-renders. This is why it’s safe to omit from the useEffect or useCallback dependency list.

### Functional updates

```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

* The ”+” and ”-” buttons use the **functional form**, because the updated value is based on the previous value. 
* But the “Reset” button uses the **normal form**, because it always sets the count back to the initial value.

If your update function returns the exact same value as the current state, **the subsequent rerender will be skipped completely**.

**Note:**  

Unlike the `setState` method found in class components, `useState` **does not automatically merge update objects**. You can replicate this behavior by combining the **function updater form with object spread syntax**:

```js
const [state, setState] = useState({});
setState(prevState => {
  // Object.assign would also work
  return {...prevState, ...updatedValues};
});
```

Another option is `useReducer`, which is more suited for managing state objects that contain multiple sub-values.

### Lazy initial state

The `initialState` argument is the state used during the initial render. In subsequent renders, it is disregarded. If the initial state is the result of an expensive computation, you may provide a function instead, which will be executed only on the initial render:

```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

### Bailing out of a state update

If you update a State Hook to the same value as the current state, React will **bail out without rendering the children or firing effects**. (React uses the `Object.is` comparison algorithm.)

Note that React may still need to render that specific component again before bailing out. That shouldn’t be a concern because React won’t unnecessarily go “deeper” into the tree. If you’re doing expensive calculations while rendering, you can optimize them with `useMemo`.

