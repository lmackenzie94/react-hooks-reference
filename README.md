# React Hooks Reference

1. [Overview](#overview)
2. [useState](#usestate)
3. [useEffect](#useeffect)
4. [useLayoutEffect](#uselayouteffect)
5. [useRef](#useref)
6. [useMemo](#usememo)
7. [useCallback)(#usecallback)
8. [Rules of Hooks](#rules)

<br><br>

<a name="overview"></a>

### Overview

> Hooks are simply functions that allow us to *hook* into certain, possibly stateful, React functionality. 

- release with React 16.8
- allows us to use functional components for everything
- reusable & 100% backwards compatible

<br><br>

<a name="usestate"></a>

### useState()
- State can be **any type** (recall class-based state had to be an object).
- **Values are preserved** between re-renders.
- Can be used multiple times to **manage multiple states**.
- **Always returns an array with exactly two elements** – your current state and a function to update that state.
- When you update state, your **previous state is fully replaced**. This differs from class-based state that would *merge* your new and previous state.
- React **automatically passes the previous state** to your setter function. This is useful when we're updating state that relies on the current value.

**EXAMPLE:**

```javascript
function Counter() {
  // Initialize 'count' with a value of 0
  // Use array destructuring to quickly assign our state and its
  // setter function to variables
  const [count, setCount] = useState(0)
  
  const increment = () => {
    // Recall React automatically passes the previous state value
    // We can call it anything but 'prevCount' makes sense here
    // This is more reliable than 'setCount(count + 1)'
    setCount(prevCount => prevCount + 1)
  }
  
  return (
    <div>
      <p>Current count: {count}</p>  
      <button onClick={increment}>Add 1</button>
    </div>
  )
}

export default Counter;
```

<br><br>

<a name="useeffect"></a>

### useEffect()

> Used to **manage side effects** (i.e. anything that affects something outside of the function being executed)

*In short, useEffect tells React that your component needs to do something after render. React remembers the function you passed and calls it later after performing the DOM updates. The function we pass is our effect. Some common use cases include data fetching, API requests, timers, event handlers, and subscriptions.* 
- By default, it **executes after every render cycle**, including the first render.
- Combines **componentDidMount, componentDidUpdate, and componentWillUnmount** into a single API.
- Passing an empty array as the second argument makes it act like componentDidMount (i.e. only runs after the first render).
- Adding dependencies to the array will tell React to only run the useEffect on the first render and when one of the specified dependencies has changed.
- Can be used multiple times within the same component.
- React defers running useEffect until after the browser has painted, so doing extra work is less of a problem because it won’t block the browser.

> TIP: Use multiple effects to separate concerns

**BASIC EXAMPLE:**

```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count])

// [count] tells React to skip applying the effect if the value of 'count' hasn't changed between re-renders.
```

> The dependency array should contain any values (data, variables, etc) defined within your component that are used inside the useEffect. However, the "setter" function returned from useState ('setCount' from the example above) can always be omitted as React ensures it won't change.

**CLEANUP EXAMPLE:**
- important to prevent memory leaks.
- we want to unsubscribe, remove event listeners, etc. after the component has unmounted.
- additionally, without cleaning up we could end up with multiple subscriptions, event listeners, etc. b/c a new one gets created every re-render.

```javascript
function Form() {
  useEffect(() => {
    function handleResize() {
      // do stuff
    }
    window.addEventListener('resize', handleResize)
    
    // this runs before the effect gets called again AND when the component unmounts.
    return () => {
      window.removeEventListener('resize', handleResize)
    }
  })
}
```

**Notes on the dependency array:**
- make sure the array includes all values from the component scope (such as props and state) that change over time and that are used by the effect.
    - if you don't, your code will reference stale values from previous renders.
- to run an effect and clean it up only once (on mount and unmount), you can pass an empty array ([]) as a second argument. This tells React that your effect doesn’t depend on any values from props or state, so it never needs to re-run.
    - passing an empty array means the props and state inside the effect will always have their initial values.
