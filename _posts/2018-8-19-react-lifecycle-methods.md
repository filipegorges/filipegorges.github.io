This post addresses the different lifecycle methods present in React, their order of execution, do's and dont's.

### Syllabu's:
**side effects**: a side effect [is anything that affects something outside the scope of the function being executed](https://www.reddit.com/r/reactjs/comments/8avfej/what_does_side_effects_mean_in_react/), in React term's, that would be, for example, fetching data, triggering an animation, saving/updating cache, etc.

**state**: a container's property which carries information on the current state of its inner properties.

**props**: props are properties passed down from parent components to children components, they can be either an object or primitve, or a reference to a function.

![React's official cheat sheet](https://github.com/filipegorges/filipegorges.github.io/tree/master/assets/images/react-lifecycle-hooks.png)

## Lifecycle methods


Method | Do's | Dont's
--- | --- | ---
`constructor()` | state initialization, event handler binding. | cause side-effects.
`render()` | return JSX, arrays, fragments, portals, strings, numbers, booleans and null. | modify state.
`componentDidMount()` | cause side effects. | if using subscriptions, must remove them with `componentWillUnmount()`.
`componentDidUpdate(prevProps, prevState)` | conditionally cause side effects. | calling `setState()` unconditionally may trigger infinite loop.
`componentWillUnmount()` | perform cleanups: cancel network requests, close subscriptions. | call `setState()`, the component will never re-render.
`shouldComponentUpdate(nextProps, nextState)` | conditionally warn React that re-rendering on update is unnecessary for the component (for performance optimizations). | use it for flow-control, "manually" preventing React from updating the component.
`static getDerivedStateFromProps(props, state)` | when state depends on changes in props, conditionally return object to update state or null. | cause side effects, reset state on prop changes, re-compute data.
`getSnapshotBeforeUpdate(prevProps, prevState)` | capture information from the DOM (scroll position, for example). Return value will be passed to `componentDidUpdate()`. | use for anything else but for DOM information capture.
`componentDidCatch()` | catch unexpected exceptions/errors from children components. (Usefull for 404's in routing). | use for control flow.

## Legacy lifecycle methods
These methods will be deprecated in future versions of React (specifically, after version 17), if you currently use them, consider reading the migration [guide](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html).

* componentWillMount()
* componentWillReceiveProps()
* componentWillUpdate()

