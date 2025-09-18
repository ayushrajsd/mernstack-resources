# React Interview Topics and Questions

## Q1. What is the Virtual DOM in React, and how does it improve performance?

**Problem Statement:**  
 Interviewers often test whether you understand React’s performance model. The Virtual DOM is frequently asked because it underpins React’s efficiency. Many candidates answer superficially (“it’s a copy of the DOM”), but interviewers look for depth in reasoning.

Intuition / Approach:

* Direct DOM manipulation is expensive (e.g., repainting, reflowing).  
* React uses a Virtual DOM (VDOM) — a lightweight in-memory tree representation of the real DOM.  
* On updates, React compares the new VDOM with the previous one using a diffing algorithm (reconciliation).  
* Only the minimal required changes are applied to the real DOM.  
* This makes UI updates fast and efficient.

Solution (Step-by-step):

1. React keeps a virtual representation of the UI in memory.  
2. When state or props change, React creates a new VDOM tree.  
3. It compares (diffs) the new tree with the old one.  
4. React finds the minimal number of operations needed.  
5. It batches and applies those updates to the actual DOM.

```js
function Counter() {
 const [count, setCount] = React.useState(0);
 return (
   <div>
     <p>Count: {count}</p>
     <button onClick={() => setCount(count + 1)}>Increment</button>
   </div>
 );
}
```

* Each time you click, React **rebuilds a new VDOM tree**.  
* Instead of repainting the whole UI, React finds that **only the `<p>` node’s text needs updating**, not the `<button>` or `<div>`.  
* That selective update is what improves performance.

## Q2. What is the difference between Props and State in React?

**Problem Statement:**  
 This is a **high-frequency fundamental** because many advanced concepts (hooks, optimization, context) build upon props and state. Interviewers often test whether you understand **data flow** and **mutability** differences.

**Intuition / Approach:**

* **Props:** External inputs passed **from parent to child** (immutable by the child).  
* **State:** Internal, managed **inside the component** (mutable via `setState` or hooks).  
* Props make components **reusable**; State makes components **interactive**.  
* Props → “function parameters”  
* State → “local memory / variables that change over time”

**Solution (Step-by-step):**

1. **Props**  
   * Passed down from parent.  
   * Read-only inside the child.  
   * Used for configuration.  
2. **State**  
   * Managed locally by the component.  
   * Can be updated with `useState` or `setState`.  
   * Drives re-rendering.

**Example:**

```js
// Parent
function App() {
 return <Welcome name="Rahul" />;
}
// Child
function Welcome({ name }) {
 const [count, setCount] = React.useState(0);
 return (
   <div>
     <h1>Hello, {name}</h1> {/* prop */}
     <p>You clicked {count} times</p> {/* state */}
     <button onClick={() => setCount(count + 1)}>Click</button>
   </div>
 );
}
```

* `name="Rahul"` → **prop** (external, fixed unless parent changes).  
* `count` → **state** (internal, changes via `setCount`).

**Mental Model:**

* Think of props as **inputs to a pure function**.  
* Think of state as **memory inside the function that evolves with time**.

## Q3. Explain `useState` in React. How does it work under the hood?

**Problem Statement:**  
 Interviewers frequently ask this because state management is central to React. A shallow answer like “it manages state” won’t cut it. You need to show you understand **initialization, re-rendering, and rules of hooks**.

**Intuition / Approach:**

* `useState` lets you add local state to functional components.  
* React remembers the state value **between re-renders**.  
* When you call the updater function (`setState`), React **schedules a re-render** with the new state.  
* React guarantees that state updates are **batched** and **asynchronous** in event handlers, not immediately reflected.  
* The hook must be called **at the top level** (same order every render).

**Solution (Step-by-step):**

```js
import React, { useState } from "react";
function Counter() {
 const [count, setCount] = useState(0); // initialize with 0
 const increment = () => {
   setCount(prev => prev + 1); // functional update to avoid stale state
 };
 return (
   <div>
     <p>Count is: {count}</p>
     <button onClick={increment}>Increment</button>
   </div>
 );
}
```

1. `useState(0)` initializes `count` with `0`.  
2. React internally keeps a **state cell** per hook call.  
3. On clicking the button, `setCount` schedules a re-render.  
4. React runs `Counter` again, but this time `count` comes from React’s internal memory, not from `0`.  
5. Using `prev => prev + 1` avoids **stale closures** when multiple updates happen quickly.

**Mental Model:**  
 Think of `useState` as React attaching a “sticky note” (the value) to the component instance. When you re-run the function, React peels off the note and sticks it back in with the updated value.

## Q4. Explain `useEffect`. How is it different from lifecycle methods in class components?

**Problem Statement:**  
 This is one of the most **high-frequency interview questions**. Many developers confuse `useEffect` timing, dependencies, and cleanup. Interviewers often test if you can **map lifecycle concepts** to `useEffect`.

**Intuition / Approach:**

* `useEffect` lets you run **side effects** (things outside rendering) in function components.  
* Examples: data fetching, subscriptions, DOM manipulations, timers.  
* By default, it runs after every render, but with dependencies you can control it.  
* It can return a cleanup function to avoid memory leaks.  
* Maps roughly to `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` in classes.

```js
import React, { useState, useEffect } from "react";
function Timer() {
 const [count, setCount] = useState(0);
 useEffect(() => {
   console.log("Effect runs");
   const id = setInterval(() => setCount(c => c + 1), 1000);
   return () => {
     console.log("Cleanup");
     clearInterval(id);
   };
 }, []); // empty dependency = run only on mount/unmount
 return <p>Timer: {count}</p>;
}
```

1. On first render, `useEffect` runs after the DOM is painted.  
2. `setInterval` starts incrementing `count`.  
3. When the component unmounts, the cleanup function clears the interval.  
4. With `[]`, it behaves like `componentDidMount` + `componentWillUnmount`.  
5. With `[count]`, it would re-run on every `count` change (like `componentDidUpdate`).

**Mental Model:**  
 Think of `useEffect` as React’s way of saying: *After I finish painting this frame, I’ll run the side jobs you asked for. And if you gave me a cleanup, I’ll run that before the next effect or unmount.*

## Q5 Why are keys important in React lists, and what happens if you don’t use them correctly?

**Problem Statement:**  
 When rendering lists in React, interviewers often ask about **keys** because incorrect usage leads to **real bugs** that developers commonly face (like input values jumping, or UI elements not updating correctly). Many answers stop at “keys help React identify items,” but you must explain **how wrong keys break reconciliation**.

**Intuition / Approach:**

* React maintains a Virtual DOM and compares the **old list** with the **new list** on re-render.  
* Keys tell React *which item in the list corresponds to which previous item*.  
* If keys are missing or unstable (like array indexes), React may:  
  * Reuse the wrong DOM nodes.  
  * Attach previous state to the wrong item.  
  * Cause subtle UI inconsistencies.

```js
function TodoList({ todos }) {
 return (
   <ul>
     {todos.map((todo, index) => (
       <li key={index}>
         <input defaultValue={todo.text} />
       </li>
     ))}
   </ul>
 );
}
```

Suppose the list is:

`[ {id:1, text:"Buy milk"}, {id:2, text:"Go gym"} ]`

Now you remove the first item. New list:

`[ {id:2, text:"Go gym"} ]`

If you used `index` as key:

* First render → keys: `[0, 1]`.  
* Second render → keys: `[0]`.

React thinks:

* Old item with key `0` is the same as new item with key `0`.  
* So it **reuses the first `<li>` DOM node** (which had “Buy milk”) and just updates its text,  
* But the `<input>` might still carry the old typed-in value (“Milk”) even though the new todo is “Go gym.”

This is a **bug caused by wrong keys** — React preserved the wrong element.

Correct:

```js
function TodoList({ todos }) {
 return (
   <ul>
     {todos.map(todo => (
       <li key={todo.id}>
         <input defaultValue={todo.text} />
       </li>
     ))}
   </ul>
 );
}
```

## Q6. What causes a React component to re-render, and how can you prevent unnecessary re-renders?

**Problem Statement:**  
 This tests your understanding of React’s rendering model. Many candidates confuse “re-render” with “DOM update.” Interviewers want to check if you know **when React re-runs the component function** and **how to optimize it**.

**Intuition / Approach:**  
 A React component re-renders when:

1. Its **state changes** (via `setState` or `useState`).  
2. Its **props change** (from parent re-render).  
3. Its **parent re-renders** (unless wrapped in memoization).  
4. Context values it consumes change.

But re-render ≠ DOM update. After re-render, React still diffs with VDOM and may skip actual DOM changes.

You can prevent unnecessary re-renders using:

* `React.memo` for pure functional components.  
* `useMemo` and `useCallback` to memoize values/functions.  
* Splitting large components into smaller one

```js
const Child = React.memo(function Child({ value }) {
 console.log("Child render");
 return <p>Value: {value}</p>;
});

function Parent() {
 const [count, setCount] = React.useState(0);
 return (
   <div>
     <button onClick={() => setCount(c => c + 1)}>Increment</button>
     <Child value="fixed" /> {/* Won’t re-render because props don’t change */}
   </div>
 );
}
```

Without `React.memo`, `Child` would re-render every time `Parent` re-renders.

With `React.memo`, React skips re-rendering `Child` since its `props` didn’t change.

For functions/objects passed as props, you may also need `useCallback` or `useMemo`.

## 

## Q7. What is `useContext` in React, and when should you use it?

**Problem Statement:**  
 This is frequently asked because developers often misuse context for all state, or fail to understand its performance implications. Interviewers want to check if you know **what problems it solves** and **when not to use it**.

**Intuition / Approach:**

* Props drilling (passing props down multiple levels) makes code verbose.  
* `useContext` allows **sharing state or values globally** without drilling.  
* Best for **theme, authentication, language settings, global data**.  
* But overusing it for frequent updates can cause **performance issues** (every consumer re-renders on change).

```js
// 1. Create a context
const ThemeContext = React.createContext("light");
function App() {
 return (
   <ThemeContext.Provider value="dark">
     <Toolbar />
   </ThemeContext.Provider>
 );
}

function Toolbar() {
 return <Button />;
}

function Button() {
 const theme = React.useContext(ThemeContext);
 return <button className={theme}>Click</button>;
}
```

1. `createContext` defines the context.  
2. `Provider` wraps components and supplies a value.  
3. Any component can call `useContext` to read the current value.

**Mental Model:**

* Use `useContext` when multiple components **need access to the same value** without manually passing it down.  
* Don’t use it for **frequently changing values** (like typing text), as it will cause unnecessary re-renders in all consumers.

## Q8. What is `useReducer` in React, and how does it compare to `useState`?

**Problem Statement:**  
 This question checks whether you understand **when state becomes complex enough** to move beyond `useState`. Many developers default to `useState`, but interviewers expect you to know when `useReducer` is better.

**Intuition / Approach:**

* `useState` works best for simple local state.  
* When state has **multiple sub-values** or **complicated transitions**, `useReducer` helps organize logic.  
* Instead of multiple `setState` calls, you centralize updates in a reducer function.  
* Conceptually similar to Redux (action → reducer → new state).

```js
function reducer(state, action) {
 switch (action.type) {
   case "increment":
     return { count: state.count + 1 };
   case "decrement":
     return { count: state.count - 1 };
   default:
     return state;
 }
}

function Counter() {
 const [state, dispatch] = React.useReducer(reducer, { count: 0 });
 return (
   <div>
     <p>Count: {state.count}</p>
     <button onClick={() => dispatch({ type: "increment" })}>+</button>
     <button onClick={() => dispatch({ type: "decrement" })}>-</button>
   </div>
 );
}
```

1. `reducer` defines state transitions based on action type.  
2. `useReducer` initializes state (`{ count: 0 }`).  
3. `dispatch` triggers updates through the reducer.  
4. Component re-renders with the new state.

**Mental Model:**

* Use `useState` for **simple toggles, counters, form inputs**.  
* Use `useReducer` when:  
  * State depends on **previous state** heavily.  
  * State logic is **complex with multiple actions**.  
  * You want a **predictable state transition flow** (like Redux but local).

## 

## Q9. What is `useMemo` in React, and when should you use it?

**Problem Statement:**  
 Interviewers ask about `useMemo` to check if you know how to avoid **expensive recalculations**. Many candidates misuse it for everything, so they want to see if you understand its **true purpose** and **trade-offs**.

**Intuition / Approach:**

* On every render, functions in React are re-executed.  
* If you have an **expensive calculation**, you don’t want it to run unless its inputs actually change.  
* `useMemo` caches (memoizes) the result of a calculation and recomputes it only when dependencies change.

```js
function ExpensiveCalculation({ num }) {
 const compute = (n) => {
   console.log("Computing...");
   let result = 0;
   for (let i = 0; i < 1e7; i++) result += n;
   return result;
 };

 const result = React.useMemo(() => compute(num), [num]);
 return <p>Result: {result}</p>;
}

```

1. Without `useMemo`, `compute(num)` runs on every render, even if `num` didn’t change.  
2. With `useMemo`, React only recomputes when `num` changes.  
3. This improves performance when calculations are **CPU-heavy**.

**Mental Model:**

* Use `useMemo` to memoize **values**.  
* Generalize: *If the calculation is expensive and the inputs didn’t change → reuse the cached value.*

## Q10. What is `useCallback` in React, and how is it different from `useMemo`?

**Problem Statement:**  
 `useCallback` is often confused with `useMemo`. Interviewers want to see if you understand it’s about **functions as dependencies/props**, not just caching results.

**Intuition / Approach:**

* In React, functions are **recreated on every render**.  
* Passing a new function down as a prop can trigger **unnecessary re-renders** in memoized children.  
* `useCallback` memoizes the function itself, ensuring its reference stays the same unless dependencies change.

## Q10. What is `useCallback` in React, and how is it different from `useMemo`?

**Problem Statement:**  
 `useCallback` is often confused with `useMemo`. Interviewers want to see if you understand it’s about **functions as dependencies/props**, not just caching results.

**Intuition / Approach:**

* In React, functions are **recreated on every render**.  
* Passing a new function down as a prop can trigger **unnecessary re-renders** in memoized children.  
* `useCallback` memoizes the function itself, ensuring its reference stays the same unless dependencies change.

```js
// // React.memo is a higher-order component that memoizes a component.

// It prevents re-renders if the props haven't changed.

const Child = React.memo(({ onClick }) => {
 console.log("Child rendered");
 return <button onClick={onClick}>Click</button>;
});

function Parent() {
 const [count, setCount] = React.useState(0);
// Without useCallback: new function reference each render.
// With useCallback: stable reference until dependencies change.
 const handleClick = React.useCallback(() => {
   console.log("Clicked");
 }, []);

 return (
   <div>
     <p>Count: {count}</p>
     <button onClick={() => setCount(c => c + 1)}>Increment</button>
     <Child onClick={handleClick} />
   </div>
 );
}
```

1. Without `useCallback`, `handleClick` would be a new function each render → causing `Child` to re-render unnecessarily.  
2. With `useCallback`, `handleClick` keeps the same reference until dependencies change.  
3. `Child` is wrapped in `React.memo`, which normally skips re-renders if props don’t change.  
4. This helps optimize performance when functions are passed as props to memoized children.

**Mental Model:**

* `useMemo` → memoize **values**.  
* `useCallback` → memoize **functions**.  
* Generalize: *If a child re-renders because of changing function references, wrap the function in `useCallback`.*

## Q11. What is `React.memo` and when should you use it?

**Problem Statement:**  
 Interviewers often ask about `React.memo` because developers misuse it, either wrapping every component or not knowing when it helps. The goal is to test whether you understand that `React.memo` is about **skipping unnecessary re-renders** when props don’t change.

**Intuition / Approach:**

* `React.memo` is a higher-order component (HOC).  
* It wraps a component and memoizes its rendered output.  
* If the props are the same as the last render, React reuses the previous rendered result instead of re-rendering.  
* Useful when:  
  * Component is **pure** (output depends only on props).  
  * Component is expensive to re-render.  
* Not always beneficial: for small components, memoization overhead can outweigh gains.

```js
const Child = React.memo(({ value }) => {
 console.log("Child rendered");
 return <p>{value}</p>
});

function Parent() {
 const [count, setCount] = React.useState(0);
 return (
   <div>
     <button onClick={() => setCount(c => c + 1)}>Increment</button>
     <Child value="fixed" />
   </div>
 );
}
```

1. Without `React.memo`, `Child` re-renders every time `Parent` re-renders.  
2. With `React.memo`, `Child` skips re-rendering because its props (`"fixed"`) never change.  
3. If props do change, React re-renders `Child`.

**Mental Model:**

* Generalize: Use it for **pure components with stable props** to avoid wasted renders.

## Q12. How does React decide what to update on the DOM (Reconciliation)?

**Problem Statement:**  
 This is one of the most important React concepts interviewers test. They want to know if you understand React’s **diffing algorithm** — how React decides which parts of the UI to update instead of repainting everything.

**Intuition / Approach:**

* On every render, React builds a new Virtual DOM tree.  
* It compares the new tree with the previous one using **reconciliation**.  
* Diffing rules:  
  1. Different types of elements → React destroys the old and builds new.  
  2. Same type → React reuses the DOM node and updates changed attributes.  
  3. Lists → React uses **keys** to match old vs new children efficiently.  
* This process makes updates efficient and predictable.

```js
function Example({ isLoggedIn }) {
 return (
   <div>
     {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in.</p>}
   </div>
 );
}
```

If `isLoggedIn` changes from `true → false`:

* React compares old `<p>Welcome back!</p>` with new `<p>Please log in.</p>`.  
* Since both are `<p>` elements, React **keeps the same `<p>` node** and just updates its text.  
* No full DOM replacement.

If you instead swapped `<p>` with `<h1>`, React would **remove the `<p>` and insert a new `<h1>`**, because the element types differ.

For lists, React uses **keys** to match children (see Q5). Without correct keys, React may reuse nodes incorrectly.

**Mental Model:**

* React’s reconciliation can be generalized as:  
  * *Same type → update in place, different type → destroy and rebuild, lists → match by key.*  
* Generalize this pattern for reasoning about why certain UI updates are efficient or buggy.

## Q14. Render a Todo List with add and delete functionality.

**Problem Statement:**  
 This is a **step-up machine coding problem**. It tests list rendering, use of keys, controlled inputs, and state updates. A common extension in interviews is to ask: *“What happens if you use index as key here?”* (link back to Q5).

**Intuition / Approach:**

* Use `useState` to manage the list of todos.  
* Use a controlled input for adding new todos.  
* Render the list with proper `key`.  
* Implement delete functionality.

```js
import React, { useState } from "react";
function TodoApp() {
 const [todos, setTodos] = useState([]);
 const [text, setText] = useState("");
 const addTodo = () => {
   if (!text.trim()) return;
   const newTodo = { id: Date.now(), text };
   setTodos(prev => [...prev, newTodo]);
   setText("");
 };

 const deleteTodo = (id) => {
   setTodos(prev => prev.filter(todo => todo.id !== id));
 };

 return (
   <div>
     <input
       value={text}
       onChange={(e) => setText(e.target.value)}
       placeholder="Add todo"
     />

     <button onClick={addTodo}>Add</button>
     <ul>
       {todos.map(todo => (
         <li key={todo.id}>
           {todo.text}
           <button onClick={() => deleteTodo(todo.id)}>Delete</button>
         </li>
       ))}
     </ul>
   </div>
 );
}
```

1. `text` manages input field value.  
2. `addTodo` adds new items to the array using `id` (not index).  
3. `deleteTodo` removes items using `filter`.  
4. React re-renders the list efficiently using keys.

**Mental Model:**

* For lists, always use **stable unique keys** (e.g., id).  
* Updates to arrays in React should be done immutably (`map`, `filter`, spread).

## 

## Q15. Build a Controlled Form with Validation

**Problem Statement:**  
 This is a frequent React interview problem because it tests whether you understand **controlled components** (inputs controlled by React state instead of DOM). Interviewers may extend by asking you to add validations, show error messages, or prevent submission until valid.

**Intuition / Approach:**

* Use `useState` to store form input values.  
* Bind `value` and `onChange` for controlled inputs.  
* On submit, validate inputs and show error if invalid.  
* Keep form updates immutable.

```js
import React, { useState } from "react";

function SignupForm() {
 const [form, setForm] = useState({ name: "", email: "" });
 const [error, setError] = useState("");
 const handleChange = (e) => {
   const { name, value } = e.target;
   setForm((prev) => ({ ...prev, [name]: value }));
 };

 const handleSubmit = (e) => {
   e.preventDefault();
   if (!form.name.trim() || !form.email.trim()) {
     setError("All fields are required");
     return;
   }

   if (!/S+@S+.S+/.test(form.email)) {
     setError("Invalid email address");
     return;
   }

   setError("");
   console.log("Form submitted:", form);
 };

 return (
   <form onSubmit={handleSubmit}>
     <input
       name="name"
       value={form.name}
       onChange={handleChange}
       placeholder="Name"
     />

     <input
       name="email"
       value={form.email}
       onChange={handleChange}
       placeholder="Email"
     />

     <button type="submit">Submit</button>

     {error && <p style={{ color: "red" }}>{error}</p>}
   </form>
 );
}
```

1. Each input is controlled by `form` state (`value` comes from state, updates via `onChange`).  
2. Validation checks for empty fields and valid email format.  
3. If validation fails, error is shown and submission is blocked.

**Mental Model:**

* Controlled components = React is **the single source of truth** for form values.  
* Pattern generalizes to: *always bind `value` + `onChange` for predictable form state.*

## Q16. Build a Star Rating Widget Component

**Problem Statement:**  
 This is a popular interview problem because it tests rendering lists, event handling, and conditional styling. It can also be extended (hover effects, resetting, controlled vs uncontrolled mode).

**Intuition / Approach:**

* Render stars (array map).  
* Track selected rating in state.  
* On click, update the rating.  
* Conditionally highlight stars based on current rating.

```js
import React, { useState } from "react";

function StarRating({ totalStars = 5 }) {
 const [rating, setRating] = useState(0);
 return (
   <div>
     {[...new Array(totalStars)].map((_, index) => (
       <span
         key={index}
         onClick={() => setRating(index + 1)}
         style={{
           cursor: "pointer",
           color: index < rating ? "gold" : "gray",
           fontSize: "24px"
         }}
       >
         ★
       </span>
     ))}

     <p>Rating: {rating}</p>
   </div>
 );
}
```

**Step-by-step:**

1. `[...new Array(totalStars)]` → creates an array with `totalStars` slots.  
2. `map` loops over it and renders a star for each index.  
3. `onClick` sets rating to `index + 1`.  
4. Stars with `index < rating` are gold, others are gray.

**Mental Model:**

* Think: *render N items → track state → style based on state.*  
* This same pattern works for checkboxes, tabs, pagination, etc.

## Q17. What are Error Boundaries in React and why are they needed?

**Problem Statement:**  
Interviewers ask this to check if you know how React handles **runtime errors in components**. Many candidates confuse error boundaries with `try–catch`. The key is to show you understand they are **React-specific components** that catch render-time errors.

---

### Intuition / Approach
- If a React component throws an error, by default the **entire React component tree unmounts**.  
- Error Boundaries are special components that **catch errors in their child component tree** during:  
  * Rendering  
  * Lifecycle methods  
  * Constructors  
- They let you gracefully show a **fallback UI** instead of crashing the whole app.  

---

### Example: Implementing an Error Boundary

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true }; // Update state so fallback UI is shown
  }

  componentDidCatch(error, info) {
    console.log("Error logged:", error, info); // log error details
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong.</h2>;
    }
    return this.props.children;
  }
}

// Usage
function BuggyComponent() {
  throw new Error("Crashed!");
}

function App() {
  return (
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

---

### Key Points
- Only **class components** can be error boundaries.  
- They catch errors from their **children**, not themselves.  
- They **don’t catch**:
  * Errors in event handlers  
  * Errors in async code (e.g., `setTimeout`)  
  * Errors during server-side rendering  


## Q18. What is React Suspense and how does it work?

**Problem Statement:**  
 This is a common advanced interview question. Interviewers test if you understand how React handles **asynchronous rendering** — particularly code-splitting and lazy loading.

**Intuition / Approach:**

* Suspense lets you **pause rendering** of a component tree until some condition (like code or data) is ready.  
* While waiting, it shows a **fallback UI**.  
* Used with:  
  * `React.lazy()` → lazy-load components.  
  * Concurrent features (e.g., future data fetching APIs).

```js
const LazyComponent = React.lazy(() => import("./LazyComponent"));

function App() {
 return (
   <div>
     <h1>App Start</h1>
     <React.Suspense fallback={<p>Loading...</p>}>
       <LazyComponent />
     </React.Suspense>
   </div>
 );
}
```

`React.lazy` dynamically imports `LazyComponent`.

While the import is loading, Suspense renders its `fallback`.

Once ready, it replaces the fallback with the component.

## Q19. What is `useRef` in React, and when should you use it?

**Problem Statement:**  
 Interviewers ask this to see if you understand **refs vs state**. Many learners mistakenly use state for things that don’t need to trigger re-renders.

**Intuition / Approach:**

* `useRef` gives you a mutable object (`.current`) that persists across renders.  
* Updating `.current` does **not trigger a re-render**.  
* Common use cases:  
  1. Accessing DOM nodes directly.  
  2. Storing values across renders without re-rendering.  
  3. Storing timers/intervals.

```js
import React, { useRef, useEffect } from "react";

function FocusInput() {
 const inputRef = useRef(null);
 useEffect(() => {
   inputRef.current.focus(); // directly focus DOM element
 }, []);

 return <input ref={inputRef} placeholder="Type here" />;
}
```

1. `useRef(null)` creates a ref object.  
2. Attach it to an element via `ref={inputRef}`.  
3. After mount, `inputRef.current` points to the DOM node.  
4. You can imperatively call DOM methods like `.focus()`.

**Mental Model:**

* Use `state` when changes should re-render the UI.  
* Use `ref` when you just need a value to persist across renders without re-rendering.

## Q20. What is `forwardRef` in React and why do we need it?

Problem Statement:  
 This tests whether you understand how refs flow in React. By default, refs can’t be passed through components — `forwardRef` is the way to expose a child’s ref to its parent.

Intuition / Approach:

* Normally, `ref` works only on DOM elements or class components.  
* If you wrap a component (like `Input`), parent refs won’t automatically attach to the child’s DOM node.  
* `forwardRef` lets you explicitly pass refs through components.

```js
import React, { useRef, forwardRef } from "react";

const CustomInput = forwardRef((props, ref) => (
 <input ref={ref} {...props} />
));

function App() {
 const inputRef = useRef();
 return (
   <div>
     <CustomInput ref={inputRef} placeholder="Enter text" />
     <button onClick={() => inputRef.current.focus()}>Focus Input</button>
   </div>
 );
}
```

`CustomInput` is wrapped in `forwardRef`.

Parent passes `ref` to `CustomInput`.

`forwardRef` ensures the `ref` points to the `<input>` inside, not the `CustomInput` component.

Now parent can directly call `.focus()` on the child input.

## Q21. Build a Stopwatch Component with Start, Stop, and Reset

**Problem Statement:**  
 This problem tests if you know how to:

1. Use `useState` to track elapsed time.  
2. Use `useRef` to store an interval ID so that it persists across renders without triggering re-renders.  
3. Manage side effects and cleanup correctly.

**Intuition / Approach:**

* Use `useState` for `time` (seconds).  
* Use `useRef` to hold the interval ID (doesn’t need to re-render when updated).  
* `start` → create an interval if one doesn’t already exist.  
* `stop` → clear interval and reset ref.  
* `reset` → clear interval and reset time to 0.

```js
import React, { useState, useRef } from "react";

function Stopwatch() {
 const [time, setTime] = useState(0);
 const intervalRef = useRef(null);
 const start = () => {
   if (intervalRef.current !== null) return; // prevent multiple intervals
   intervalRef.current = setInterval(() => {
     setTime(prev => prev + 1);
   }, 1000);
 };

 const stop = () => {
   clearInterval(intervalRef.current);
   intervalRef.current = null;
 };

 const reset = () => {
   clearInterval(intervalRef.current);
   intervalRef.current = null;
   setTime(0);
 };

 return (
   <div>
     <h2>Time: {time}s</h2>
     <button onClick={start}>Start</button>
     <button onClick={stop}>Stop</button>
     <button onClick={reset}>Reset</button>
   </div>
 );
}
```

**Key Points:**

1. `time` is state → triggers re-render each second.  
2. `intervalRef.current` stores the interval ID → no re-renders when it changes.  
3. `start` ensures only one interval runs at a time.  
4. `stop` and `reset` both clear the interval.

**Mental Model:**

* Use `state` for values that should re-render the UI.  
* Use `ref` for values that should **persist across renders but not cause re-renders**

## Q23. Build a Debounced Search Input

**Problem Statement:**  
 Implement a search input where the API is called only after the user **stops typing for 500ms**. This prevents excessive API calls on every keystroke.

**Intuition / Approach:**

* Use `useState` to track the input value.  
* Use `useEffect` to set up a `setTimeout` whenever the value changes.  
* Use `useRef` to store the timeout ID so it can be cleared on new keystrokes.  
* Call the “API” only when typing has paused for the debounce delay.

```js
import React, { useState, useEffect, useRef } from "react";

function DebouncedSearch() {
 const [query, setQuery] = useState("");
 const timeoutRef = useRef(null);

 useEffect(() => {
   if (query === "") return;
   // clear previous timeout if user types again
   if (timeoutRef.current) {
     clearTimeout(timeoutRef.current);
   }

   timeoutRef.current = setTimeout(() => {
     console.log("API call with query:", query);
     // Here you would call your API with `query`
   }, 500);

   // cleanup on unmount
   return () => clearTimeout(timeoutRef.current);
 }, [query]);

 return (
   <div>
     <input
       type="text"
       placeholder="Search..."
       value={query}
       onChange={(e) => setQuery(e.target.value)}
     />
   </div>
 );
}
```

**Key Points:**

1. `query` is state, so UI updates as user types.  
2. Timeout ID is stored in `useRef` so it persists across renders but doesn’t trigger re-renders.  
3. `useEffect` resets the timer if typing continues, ensuring only the last input triggers the “API call.”

## Q24. Build a Tabbed Component

**Problem Statement:**  
 You need to build a component that shows different content when different tabs are clicked. This tests whether you can manage **active state** and conditionally render content.

**Intuition / Approach:**

* Use `useState` to track the active tab index.  
* Render tab headers in a row.  
* Highlight the active tab and show only its content.

```js
import React, { useState } from "react";

function Tabs() {
 const [active, setActive] = useState(0);
 const tabs = ["Home", "Profile", "Settings"];
 const content = [
   "Welcome to the Home tab",
   "This is your Profile tab",
   "Here are the Settings"
 ];

 return (
   <div>
     {/* Tab headers */}
     <div style={{ display: "flex", gap: "10px" }}>
       {tabs.map((tab, index) => (
         <button
           key={index}
           onClick={() => setActive(index)}
           style={{
             fontWeight: active === index ? "bold" : "normal"
           }}
         >
           {tab}
         </button>
       ))}
     </div>
     {/* Tab content */}
     <div style={{ marginTop: "10px" }}>
       <p>{content[active]}</p>
     </div>
   </div>
 );
}
```

**Key Points:**

1. Active state determines which content is shown.  
2. Conditional styling highlights the active tab.  
3. The logic scales to any number of tabs.

## Q25. Build an Accordion Component

**Problem Statement:**  
 Build an accordion where clicking a section title toggles its visibility. This tests **conditional rendering** and **list state management**.

**Intuition / Approach:**

* Use `useState` to track the currently open index (or `null` if none).  
* When a header is clicked, toggle open/close.  
* Render content conditionally based on active index

```js
import React, { useState } from "react";

function Accordion() {
 const [activeIndex, setActiveIndex] = useState(null);
 const items = [
   { title: "Section 1", content: "Content of section 1" },
   { title: "Section 2", content: "Content of section 2" },
   { title: "Section 3", content: "Content of section 3" }
 ];

 const toggle = (index) => {
   setActiveIndex(prev => (prev === index ? null : index));
 };

 return (
   <div>
     {items.map((item, index) => (
       <div key={index}>
         <h3 onClick={() => toggle(index)} style={{ cursor: "pointer" }}>
           {item.title}
         </h3>
         {activeIndex === index && <p>{item.content}</p>}
       </div>
     ))}
   </div>
 );
}
```

**Key Points:**

1. State tracks which item is expanded.  
2. Clicking toggles the same item open/close.  
3. Only one section can be open at a time (easy extension → allow multiple open).

## Q26. Build a File Explorer Component

**Problem Statement:**  
 You are asked to build a File Explorer-like UI. The structure includes folders and files. Folders can be expanded/collapsed to show or hide their children. This problem tests:

1. Recursive rendering of nested data.  
2. State management for expand/collapse.  
3. Handling dynamic UI interactions.

**Intuition / Approach:**

* Represent the file system as a **nested JSON object**.  
* Each folder should have its own local expand/collapse state.  
* Recursively render children when a folder is expanded.

```js
import React, { useState } from "react";

const data = {
 name: "root",
 isFolder: true,
 children: [
   {
     name: "src",
     isFolder: true,
     children: [
       { name: "index.js", isFolder: false },
       { name: "App.js", isFolder: false }
     ]
   },
   {
     name: "public",
     isFolder: true,
     children: [
       { name: "index.html", isFolder: false }
     ]
   },
   { name: "package.json", isFolder: false }
 ]
};

function Explorer({ node }) {
 const [expanded, setExpanded] = useState(false);
 if (!node.isFolder) {
   return <p style={{ marginLeft: "20px" }}>📄 {node.name}</p>;
 }

 return (
   <div style={{ marginLeft: "20px" }}>
     <p onClick={() => setExpanded(prev => !prev)} style={{ cursor: "pointer" }}>
       {expanded ? "📂" : "📁"} {node.name}
     </p>

     {expanded &&
       node.children.map((child, index) => (
         <Explorer key={index} node={child} />
       ))}

   </div>
 );
}

function FileExplorerApp() {
 return <Explorer node={data} />;
}

export default FileExplorerApp;

```

**How It Works:**

1. `Explorer` component is **recursive** — if the node is a folder, it renders its children by calling `Explorer` again.  
2. Each folder has its own local `expanded` state.  
3. Clicking the folder toggles expansion.  
4. Files are just rendered as plain text.

**Key Points:**

* Data structure drives the UI.  
* Recursion simplifies nested rendering.  
* Each folder’s expand/collapse is **independent**.

## Q27. Build a Job Board Component (Fetch Jobs from Public API)

**Problem Statement:**  
 You are asked to build a Job Board that fetches jobs from the public **Hacker News Jobs API**. This problem tests your ability to:

1. Make multiple API calls.  
2. Manage async data fetching in React.  
3. Handle loading and error states.

Hacker News Job APIs:

1. **Fetch job IDs - [https://hacker-news.firebaseio.com/v0/jobstories.json](https://hacker-news.firebaseio.com/v0/jobstories.json)**  
   1. **Returns an array of job IDs.**  
2. **Fetch job details by ID - [https://hacker-news.firebaseio.com/v0/item/{id}.json](https://hacker-news.firebaseio.com/v0/item/{id}.json)**

Display jobs in a list with title, author, and link.

Show loading and error states.

Implement a “Load More” button to load jobs in batches of 10. The button should hide when there are no more jobs to load.

**Intuition / Approach:**

* Fetch **all job IDs** once and store them.  
* Use `page` state to track which batch is being loaded.  
* Each time the page increases, fetch the corresponding slice of job IDs and their details.  
* Combine new jobs with existing ones.  
* Use a **functional update** for `setJobs` to avoid stale closures.  
* Track `fetchingJobDetails` to prevent double fetches and manage button state.

```js
import React, { useEffect, useRef, useState } from "react";

const PAGE_SIZE = 6;

function JobBoard() {
 const [jobIds, setJobIds] = useState(null);
 const [jobs, setJobs] = useState([]);
 const [page, setPage] = useState(0);
 const [fetchingJobDetails, setFetchingJobDetails] = useState(false);
 const [error, setError] = useState(null);
 const isMounted = useRef(true);

 // Handle component unmount

 useEffect(() => {
   isMounted.current = true;
   return () => {
     isMounted.current = false;
   };
 }, []);

 // Fetch jobs whenever page changes

 useEffect(() => {
   fetchJobs(page);
   // eslint-disable-next-line react-hooks/exhaustive-deps
 }, [page]);

 async function fetchJobIds(currPage) {
   let ids = jobIds;
   if (!ids) {
     try {
       const res = await fetch(
         "https://hacker-news.firebaseio.com/v0/jobstories.json"
       );
       ids = await res.json();
       if (!isMounted.current) return;
       setJobIds(ids);
     } catch (err) {
       setError("Failed to load job IDs: " + err.message);
       return [];
     }
   }

   const start = currPage * PAGE_SIZE;
   const end = start + PAGE_SIZE;
   return ids.slice(start, end);
 }

 async function fetchJobs(currPage) {
   const jobIdsForPage = await fetchJobIds(currPage);
   if (!jobIdsForPage.length) return;
   try {
     setFetchingJobDetails(true);
     const jobsForPage = await Promise.all(
       jobIdsForPage.map((jobId) =>
         fetch(
           `https://hacker-news.firebaseio.com/v0/item/${jobId}.json`
         ).then((res) => res.json())
       )
     );
     if (!isMounted.current) return;

     setJobs((prev) => [...prev, ...jobsForPage]); // functional update
   } catch (err) {
     setError("Failed to load jobs: " + err.message);
   } finally {
     if (isMounted.current) {
       setFetchingJobDetails(false);
     }
   }
 }

 if (error) return <p style={{ color: "red" }}>{error}</p>;

 if (jobIds == null) return <p>Loading job IDs...</p>;

 return (
   <div className="app">
     <h1>Hacker News Job Board</h1>
     <div className="jobs" role="list">
       {jobs.map((job) => (
         <div key={job.id} className="job-posting">
           <h3>{job.title}</h3>
           <p>Posted by: {job.by}</p>
           {job.url && (
             <a href={job.url} target="_blank" rel="noreferrer">
               View Job
             </a>
           )}
         </div>
       ))}
     </div>

     {jobs.length > 0 &&
       page * PAGE_SIZE + PAGE_SIZE < jobIds.length && (
         <button
           className="load-more-button"
           disabled={fetchingJobDetails}
           onClick={() => setPage((prev) => prev + 1)}
         >
           {fetchingJobDetails ? "Loading..." : "Load More"}
         </button>
       )}
   </div>
 );
}

export default JobBoard;
```

**Key Points in This Solution:**

1. **Job IDs fetched only once** → efficient.  
2. **Pagination logic** → slice IDs based on page number.  
3. **Functional state update** in `setJobs` → prevents stale closure bugs.  
4. **Unmount safeguard** → avoids updating state after unmount.  
5. **Load More button** →  
   * Disabled while fetching.  
   * Hides when all jobs are loaded.

## Q28. What are common strategies to optimize performance in a React app?

**Problem Statement:**  
 Interviewers often ask this open-ended question to see whether you can identify performance bottlenecks and apply the right optimizations. A shallow answer like “use React.memo” won’t be enough — they want a **structured overview of strategies**.

**Intuition / Approach:**  
 Optimizations in React can be grouped into three areas:

1. **Preventing unnecessary re-renders**  
   * `React.memo` for pure components.  
   * `useCallback` and `useMemo` for stable function and value references.  
   * Proper use of keys in lists.  
2. **Reducing bundle size / load time**  
   * Code splitting with `React.lazy` + `Suspense`.  
   * Dynamic imports.  
   * Tree shaking and avoiding unused libraries.  
3. **Efficient UI rendering**  
   * Virtualizing long lists with `react-window` or `react-virtualized`.  
   * Debouncing and throttling input handlers.  
   * Offloading expensive work to Web Workers.  
4. **Other good practices**  
   * Caching results from APIs (React Query, SWR).  
   * Avoiding inline object/array creation inside render.  
   * Using `useRef` for values that don’t need to re-render.

**Solution (Summary List):**

* Avoid unnecessary re-renders → `React.memo`, `useCallback`, `useMemo`.  
* Optimize list rendering → virtualization (`react-window`).  
* Optimize bundle → code splitting, lazy loading.  
* Handle async efficiently → caching libraries, debouncing.  
* Optimize heavy work → Web Workers, `useMemo`.

## Q29. Why do we need Redux when we already have Context and useReducer?

**Problem Statement:**  
 A common interview question: if React already has `useReducer` for complex local state and Context for global sharing, why would you use Redux at all?

**Intuition / Approach:**

* **Context + useReducer** works fine for small apps.  
* But for large-scale apps:  
  1. **Re-render issues** – Context makes every consumer re-render when value changes.  
  2. **DevTools/debugging** – Redux has time-travel debugging, action logging, and middleware.  
  3. **Scalability** – Redux organizes state updates via a predictable action → reducer → store flow.  
  4. **Middleware support** – Handles async logic (Redux Thunk, Redux Saga).  
  5. **Cross-cutting concerns** – Easier to handle state that multiple distant components need.

**Solution (Step-by-step):**

* Context + useReducer is like **manual wiring** of global state.  
* Redux adds:  
  * A **centralized store**.  
  * **Middleware** for async/side effects.  
  * **DevTools integration** for debugging.  
  * **Performance optimizations** (only subscribed components re-render).

**Mental Model:**

* Generalize: *Context/useReducer = good for small apps. Redux = designed for complex apps where state management, debugging, and middleware support are critical.*

## Q30. What are the benefits of Redux Toolkit over plain Redux?

**Problem Statement:**  
 Many interviews check if you know the modern Redux way — **Redux Toolkit (RTK)**. Using “plain Redux” with manual reducers and action creators is discouraged now.

**Intuition / Approach:**

* Plain Redux required:  
  1. Writing boilerplate (`actions`, `actionTypes`, `reducers`).  
  2. Manual immutable updates (lots of `...spread`).  
  3. Verbose setup.  
* Redux Toolkit solves this with:  
  1. **`configureStore`** – sets up store with DevTools and middleware automatically.  
  2. **`createSlice`** – generates actions + reducers together.  
  3. **Immer.js under the hood** – lets you write “mutating” logic safely (immutability handled internally).  
  4. **Thunks built-in** – `createAsyncThunk` simplifies async logic.

```js
import { configureStore, createSlice } from "@reduxjs/toolkit";

// Slice = reducer + actions

const counterSlice = createSlice({
 name: "counter",
 initialState: { value: 0 },
 reducers: {
   increment: (state) => { state.value += 1 },  // immer lets us mutate
   decrement: (state) => { state.value -= 1 }
 }
});

export const { increment, decrement } = counterSlice.actions;

const store = configureStore({
 reducer: { counter: counterSlice.reducer }
});
```

* No manual action types.  
* No boilerplate reducers.  
* Immutability handled internally by Immer.  
* Store preconfigured with good defaults.

**Mental Model:**

* Plain Redux = verbose manual setup.  
* RTK = batteries included, less code, more readable, safe immutability.

## 

## Q31. What other state management libraries are available apart from Redux? (Good-to-know for interviews)

**Problem Statement:**  
 Redux is popular, but not the only option. Interviewers may ask this to see if you are aware of the broader ecosystem and can pick the right tool for the right job.

**Intuition / Approach:**  
 Different libraries approach state differently:

1. **Zustand**  
   * Lightweight, simple store based on hooks.  
   * No boilerplate — just functions and hooks.  
   * Selectors prevent unnecessary re-renders.

```js
import create from 'zustand';

const useStore = create(set => ({
 count: 0,
 increment: () => set(state => ({ count: state.count + 1 }))
}));

function Counter() {
 const { count, increment } = useStore();
 return <button onClick={increment}>{count}</button>;
}
```

1. **Recoil (by Facebook/Meta)**  
   * State is split into **atoms** (pieces of state).  
   * Components subscribe to atoms; only re-render if that atom changes.  
   * Great for large apps where state is shared in fine-grained ways.  
2. **MobX**  
   * Uses **observables**.  
   * Reactively updates components when observed state changes.  
   * More implicit than Redux (less boilerplate, but harder to debug sometimes).  
3. **XState**  
   * State machine + statecharts.  
   * Explicit states and transitions → good for workflows (wizards, forms, multi-step flows).

**Solution (Interview Answer):**

* Redux is great for predictable state management, especially with Redux Toolkit.  
* But for smaller apps, **Zustand** or **Recoil** may be simpler and lighter.  
* **MobX** focuses on reactive programming with observables.  
* **XState** is powerful for complex workflows requiring finite states.

**Mental Model:**

* Generalize: *Redux = structured and verbose, great for large apps. Alternatives like Zustand, Recoil, MobX, and XState trade boilerplate for simplicity or special use cases.*

## Q32. Explain how Redux works internally (Action → Reducer → Store → UI update)

**Problem Statement:**  
 Interviewers often test whether you really understand Redux’s **unidirectional data flow**. Many candidates can “use” Redux but can’t explain what happens step by step when a user clicks a button.

**Intuition / Approach:**  
 The flow in Redux is always the same:

1. **UI dispatches an action** – usually triggered by a user event.  
2. **Action reaches the reducer** – a pure function that calculates the next state based on the action type.  
3. **Reducer returns new state** – Redux replaces the old state with the new one.  
4. **Store notifies subscribers** – components that are connected to the store (via `useSelector` or `connect`) re-render with the new state.

```js
// actions are plain JS objects
const increment = () => ({ type: "INCREMENT" });

// reducer is a pure function
function counterReducer(state = { value: 0 }, action) {
 switch (action.type) {
   case "INCREMENT":
     return { value: state.value + 1 };
   default:
     return state;
 }
}

// store holds global state

import { createStore } from "redux";

const store = createStore(counterReducer);

// UI dispatches an action

store.dispatch(increment());

// store calls reducer → updates state

console.log(store.getState()); // { value: 1 }
```

**Step-by-step flow explained:**

1. User clicks “+” → `dispatch({ type: "INCREMENT" })`.  
2. Store passes the current state + action to the reducer.  
3. Reducer returns `{ value: state.value + 1 }`.  
4. Store updates its state.  
5. Components subscribed to the store re-render with the new value.

**Mental Model:**

* Redux always follows:  
   **Action → Reducer → Store → UI update.**  
* Generalize: *Data flows in one direction, making state predictable and easier to debug*

## Q33. What is Concurrent Rendering in React and why is it important?

**Problem Statement:**  
 React introduced concurrent rendering features starting with React 18. Interviewers ask this to test if you understand how React improves performance and user experience by rendering work in a more **interruptible, prioritized** way.

**Intuition / Approach:**

* In older React versions, rendering was **synchronous**: once rendering started, it blocked the main thread until completion.  
* In concurrent rendering, React can **pause, interrupt, and resume rendering**.  
* This allows React to prioritize urgent updates (like typing) over less urgent ones (like rendering a big list).  
* Features built on top of concurrent rendering:  
  1. **Automatic Batching**  
  2. **Transitions** (mark updates as non-urgent)  
  3. **Suspense improvements**

```js
import React, { useState, startTransition } from "react";

function SearchApp() {
 const [query, setQuery] = useState("");
 const [results, setResults] = useState([]);
 function handleChange(e) {
   const value = e.target.value;
   setQuery(value);
   // mark expensive update as non-urgent
   startTransition(() => {
     const filtered = Array(5000)
       .fill("item")
       .map((v, i) => v + i)
       .filter((item) => item.includes(value));
     setResults(filtered);
   });
 }

 return (
   <div>
     <input value={query} onChange={handleChange} />
     <ul>
       {results.map((r, i) => (
         <li key={i}>{r}</li>
       ))}
     </ul>
   </div>
 );
}
```

* Typing in the input is urgent → React updates `query` immediately.  
* Filtering 5000 items is heavy → wrapped in `startTransition`, so React may delay/resume it without blocking typing.

**Key Points:**

* Concurrent rendering makes React apps **more responsive**.  
* It’s not a new API, but an **internal rendering mechanism** that powers features like `startTransition`, `Suspense`, and automatic batching.

## 

## Q34. Explain React’s batching of state updates with an example.

**Problem Statement:**  
 Batching means React groups multiple state updates into a single render to improve performance. Interviewers want to check if you know this, especially since React 18 introduced **automatic batching for all async events** (not just React events).

**Intuition / Approach:**

* Before React 18: batching happened only inside React event handlers.  
* Outside React (like `setTimeout`, `Promise.then`), updates caused multiple renders.  
* React 18 introduced **automatic batching everywhere**.

```js
import React, { useState, useEffect } from "react";

function BatchingDemo() {
 const [count, setCount] = useState(0);
 const [flag, setFlag] = useState(false);
 useEffect(() => {
   setTimeout(() => {
     console.log("Before updates");
     setCount(c => c + 1);
     setFlag(f => !f);
     console.log("After updates");
   }, 1000);
 }, []);

 console.log("Component rendered");

 return (
   <div>
     <p>Count: {count}</p>
     <p>Flag: {flag.toString()}</p>
   </div>
 );
}
```

**What happens in different React versions?**

* **React 17 (before automatic batching):**  
  * First `setCount` triggers a re-render → `"Component rendered"` logs once.  
  * Then `setFlag` triggers another re-render → `"Component rendered"` logs again.  
  * You’ll see **2 renders** for those 2 state updates.  
* **React 18 (with automatic batching):**  
  * React groups both updates into a single render.  
  * `"Component rendered"` logs **only once**.

## 

## Q35. Create a custom hook to determine window size on resizing

**Problem Statement:**  
 Interviewers ask this to test your ability to build **custom hooks** and handle **browser events**. The goal is to write a hook that listens for `resize` events and returns the current `width` and `height` of the browser window.

**Intuition / Approach:**

* Use `useState` to store window dimensions.  
* Use `useEffect` to attach a resize event listener.  
* Clean up the listener on unmount.  
* Return the dimensions from the hook.

```js
import { useState, useEffect } from "react";

// Custom hook

function useWindowSize() {
 const [size, setSize] = useState({
   width: window.innerWidth,
   height: window.innerHeight
 });

 useEffect(() => {
   function handleResize() {
     setSize({
       width: window.innerWidth,
       height: window.innerHeight
     });
   }

   window.addEventListener("resize", handleResize);

   return () => window.removeEventListener("resize", handleResize);

 }, []);

 return size;

}

// Usage

function App() {
 const { width, height } = useWindowSize();
 return (
   <p>
     Window size: {width} x {height}
   </p>
 );
}
```

**Key Points:**

1. Custom hooks start with `use` and can use other hooks internally.  
2. Always clean up event listeners in `useEffect`.  
3. Hook returns data (width/height) that can be used in any component.

**Mental Model:**

* Generalize: *Custom hooks = extract reusable logic that uses hooks.*

## Q36. Build a Table Component with Pagination

**Problem Statement:**  
 This tests your ability to handle **lists, slicing, and UI state management**. You need to create a table that displays rows of data and supports pagination with Next/Prev buttons.

**Intuition / Approach:**

* Use `useState` to track `currentPage`.  
* Slice the data array based on `currentPage` and `pageSize`.  
* Disable “Prev” on the first page and “Next” on the last page.

```js
import React, { useState } from "react";

function TableWithPagination({ data, pageSize = 5 }) {
 const [page, setPage] = useState(0);
 const start = page * pageSize;
 const end = start + pageSize;
 const paginatedData = data.slice(start, end);
 return (
   <div>
     <table border="1" cellPadding="5">
       <thead>
         <tr>
           <th>ID</th>
           <th>Name</th>
         </tr>
       </thead>
       <tbody>
         {paginatedData.map((item) => (
           <tr key={item.id}>
             <td>{item.id}</td>
             <td>{item.name}</td>
           </tr>
         ))}
       </tbody>
     </table>
     <div style={{ marginTop: "10px" }}>
       <button onClick={() => setPage(p => p - 1)} disabled={page === 0}>
         Prev
       </button>
       <span style={{ margin: "0 10px" }}>
         Page {page + 1} of {Math.ceil(data.length / pageSize)}
       </span>
       <button
         onClick={() => setPage(p => p + 1)}
         disabled={end >= data.length}
       >
         Next
       </button>
     </div>
   </div>
 );
}

// Example usage

const sampleData = Array.from({ length: 22 }, (_, i) => ({
 id: i + 1,
 name: `Item ${i + 1}`
}));

export default function App() {

 return <TableWithPagination data={sampleData} pageSize={5} />;

}
```

**Key Points:**

1. `page` state determines which slice of data is displayed.  
2. `Prev` and `Next` buttons update `page`.  
3. Buttons are disabled when on first/last page.  
4. Easily extendable (sorting, searching, API-driven pagination).

**Mental Model:**

* Generalize: *Pagination = slice(start, end) based on page × pageSize.*

