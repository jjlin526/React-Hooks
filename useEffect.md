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

### Timing of effects

Unlike `componentDidMount` and `componentDidUpdate`, the function passed to `useEffect` fires **after layout and paint**, during a **deferred event**. This makes it suitable for the many common side effects, like setting up subscriptions and event handlers, because most types of work shouldn’t block the browser from updating the screen.

However, not all effects can be deferred. For example, a DOM mutation that is visible to the user must fire synchronously before the next paint so that the user does not perceive a visual inconsistency. (The distinction is conceptually similar to passive versus active event listeners.) For these types of effects, React provides one additional Hook called `useLayoutEffect`. It has the same signature as `useEffect`, and only differs in when it is fired.

Additionally, starting in React 18, the function passed to `useEffect` will fire synchronously before layout and paint when it’s the result of a **discrete user input** such as a click, or when it’s the result of an **update wrapped in flushSync**. This behavior allows the result of the effect to be observed by the event system, or by the caller of `flushSync`.

**Note:**  

This **only affects the timing of when the function passed to `useEffect` is called** - updates scheduled inside these effects are still deferred. This is different than `useLayoutEffect`, which fires the function and processes the updates inside of it immediately.

Even in cases where `useEffect` is deferred until after the browser has painted, it’s guaranteed to fire before any new renders. React will always flush a previous render’s effects before starting a new update.

### componentDidMount and componentDidUpdate vs. useEffect

`componentDidMount` and `componentDidUpdate` are lifecycle methods in React class components that are called at specific points during the component's lifecycle. `componentDidMount` is called immediately after the component is added to the DOM and `componentDidUpdate` is called every time a component updates, which includes re-rendering.

On the other hand, `useEffect` is a Hook in React functional components that allows you to synchronize a component with an external system or some other state or props. The `useEffect` runs after the layout and paint. It’s better suited for side effects like subscriptions and event handlers, because it allows the browser to update the screen before running the effect.

This behavior is different from `componentDidMount` and `componentDidUpdate` because `useEffect` is more flexible and it allows you to control when the effect should run by providing a list of dependencies, unlike `componentDidMount` and `componentDidUpdate` which are called automatically when the component mounts or updates.

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
