# React Hooks Reference

1. [Overview](#overview)
2. [useState](#usestate)
3. [useEffect](#useeffect)
4. [useLayoutEffect](#uselayouteffect)
5. [useContext](#usecontext)
6. [useRef](#useref)
7. [useMemo](#usememo)
8. [useCallback](#usecallback)
9. [Rules of Hooks](#rules)

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

*In short, <code>useEffect</code> tells React that your component needs to do something after render. React remembers the function you passed and calls it later after performing the DOM updates. The function we pass is our effect. Some common use cases include data fetching, API requests, timers, event handlers, and subscriptions.* 
- By default, it **executes after every render cycle**, including the first render.
- Combines **componentDidMount, componentDidUpdate, and componentWillUnmount** into a single API.
- Passing an empty array as the second argument makes it act like componentDidMount (i.e. only runs after the first render).
- Adding dependencies to the array will tell React to only run the <code>useEffect</code> on the first render and when one of the specified dependencies has changed.
- Can be used multiple times within the same component.
- React defers running <code>useEffect</code> until after the browser has painted, so doing extra work is less of a problem because it won’t block the browser.

> TIP: Use multiple effects to separate concerns

**BASIC EXAMPLE:**

```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count])

// [count] tells React to skip applying the effect if the value of 'count' hasn't changed between re-renders.
```

> The dependency array should contain any values (data, variables, etc) defined within your component that are used inside the <code>useEffect</code>. However, the "setter" function returned from <code>useState</code> ('setCount' from the example above) can always be omitted as React ensures it won't change.

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
    
<br><br>

<a name="uselayouteffect"></a>

### useLayoutEffect()

> USE CASE: if your effect is mutating the DOM (via a DOM node ref) **and** the DOM mutation will change the appearance of the DOM node between the time that it is rendered and your effect mutates it. Using <code>useEffect</code> could result in a flicker when your DOM mutations take effect.
> TIP: always use <code>useEffect</code> when possible to avoid blocking visual updates.

- identical to <code>useEffect</code>, but fires *synchronously* after all DOM mutations.
- used to read layout from the DOM and synchronously re-render.
- your code runs immediately after the DOM has been updated, but before the browser has had a chance to "paint" those changes (the user doesn't actually see the updates until after the browser has repainted).
- can be useful if you need to make DOM measurements (like getting the scroll position or other styles for an element) and then make DOM mutations or trigger a synchronous re-render by updating state.

[useEffect() example - render flash](https://codesandbox.io/s/useeffect-flash-on-render-yluoi) **vs.** [useLayoutEffect example - no flash](https://codesandbox.io/s/uselayouteffect-no-flash-ylyyg?file=/src/index.js)

<br><br>

<a name="usecontext"></a>

### useContext()

- **accepts a context object** (i.e. the value returned from <code>React.createContext</code>)
- **returns the current context value** for that context.
    - which is determined by the <code>value</code> prop of the nearest <code><MyContext.Provider></code> above calling component in the tree.
- a component calling <code>useContext</code> will always re-render when the context value changes. If re-rendering the component is expensive, you can optimize it by using memoization.
- only lets you *read* the context and subscribe to its changes. You still need a <code><MyContext.Provider></code> above in the tree to *provide* the value for this context.
  
**EXAMPLE:**

```javascript
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

