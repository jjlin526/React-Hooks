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
