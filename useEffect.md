### useEffect

Accepts a function that contains imperative, possibly effectful code.

**Imperative programming** is a paradigm of computer programming where the program describes steps that change the state of the computer.

Mutations, subscriptions, timers, logging, and other side effects are not allowed inside the main body of a function component (referred to as **React’s render phase**). Doing so will lead to confusing bugs and inconsistencies in the UI.

Instead, use `useEffect`. The function passed to `useEffect` will run after the render is committed to the screen. Think of effects as an escape hatch from React’s purely functional world into the imperative world.

By default, **effects run after every completed render**, but you can choose to fire them only when certain values have changed.

### Cleaning up an effect

Often, effects create resources that need to be cleaned up before the component leaves the screen, such as a subscription or timer ID. To do this, the function passed to useEffect may return a **clean-up function**. For example, to create a subscription:

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

A **memory leak** is a type of resource leak that occurs when a computer program incorrectly manages memory allocations in a way that memory which is no longer needed is not released. 

The clean-up function runs before the component is removed from the UI to prevent memory leaks. Additionally, if a component renders multiple times (as they typically do), **the previous effect is cleaned up before executing the next effect**. In our example, this means a new subscription is created on every update. 

### Conditionally firing an effect

The default behavior for effects is to fire the effect after every completed render. That way an effect is always recreated if one of its dependencies changes.

However, this may be overkill in some cases, like the subscription example from the previous section. We don’t need to create a new subscription on every update, only if the source prop has changed.

To implement this, pass a second argument to `useEffect` that is the array of values that the effect depends on. Our updated example now looks like this:

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

Now the subscription will only be recreated when `props.source` changes.

**Note:**

If you use this optimization, make sure the array includes **all values from the component scope (such as props and state) that change over time and that are used by the effect**. Otherwise, your code will reference stale values from previous renders. Learn more about how to deal with functions and what to do when the array values change too often.

If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array ([]) as a second argument. This tells React that your effect doesn’t depend on any values from props or state, so it never needs to re-run. This isn’t handled as a special case — it follows directly from how the dependencies array always works.

If you pass an empty array ([]), the props and state inside the effect will always have their initial values. While passing [] as the second argument is closer to the familiar `componentDidMount` and `componentWillUnmount` mental model, there are usually better solutions to avoid re-running effects too often. Also, don’t forget that React defers running useEffect until after the browser has painted, so doing extra work is less of a problem.

We recommend using the `exhaustive-deps` rule as part of our `eslint-plugin-react-hooks` package. It warns when dependencies are specified incorrectly and suggests a fix.

The array of dependencies is not passed as arguments to the effect function. Conceptually, though, that’s what they represent: every value referenced inside the effect function should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
