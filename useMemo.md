### useMemo

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Returns a **memoized value**.

**Pass a “create” function and an array of dependencies**. `useMemo` will only recompute the memoized value when one of the dependencies has changed. This **optimization helps to avoid expensive calculations on every render**.

Remember that the function passed to `useMemo` runs during **rendering**. Don’t do anything there that you wouldn’t normally do while rendering. For example, **side effects** belong in `useEffect`, not `useMemo`.

If no array is provided, a new value will be computed on every render.
