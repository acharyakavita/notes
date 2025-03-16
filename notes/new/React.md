# React

## State vs Props

Props are passed into the component and state is maintained internally

## Refs

Refs provide a way to access DOM nodes or React elements created in the render method. Refs prop has the DOM element passed into the callback function. Can be used by class components and functional ones via closure.

### Examples

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }

  render() {
    return <div ref={this.myRef} />;
  }
}
```

or

```jsx
class UserForm extends React.Component {
  handleSubmit = () => {
    console.log("Input Value is: ", this.input.value);
  };

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input type="text" ref={(input) => (this.input = input)} /> // Access
        DOM input in handle submit
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

## Context

Context provides a way to pass data through the component tree without having to pass props down manually at every level.

```jsx
const ThemeContext = React.createContext("light");

class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

## Controlled vs uncontrolled components

### Controlled

Takes its current value through props and notifies changes through callbacks like `onChange`. A parent component "controls" it by handling the callback and managing its own state and passing the new values as props to the controlled component. You could also call this a "dumb component".

```jsx
<input type="text" value={value} onChange={handleChange} />
```

### Uncontrolled

Stores its own state internally and you can query the DOM using a ref to find its current value

```jsx
<input type="text" defaultValue="foo" ref={inputRef} />
```

## Class components

- use lifecycle methods
- extend `React.Component`
- have `this` binding

### Example

```jsx
import React from 'react';
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { foo: 1 };
  }

  clickHandler() {
    this.setState({
        foo: this.state.foo + 1 // or use callback with state passed in for sync
    });
  }

  render() {
    return (
      <div>{this.state.foo}</div>
      <button onClick={clickHandler}>click me</button>
    );
  }
}

MyComponent.propTypes = {
  thing: PropTypes.string,
  stuff: PropTypes.arrayOf(PropTypes.string).isRequired
};

MyComponent.defaultProps = {
  thing: 'something'
};
```

### State Updates

#### simple, possibly async

```jsx
this.setState({ foo: 3 });
```

#### synchronous

```jsx
this.setState((state, props) => ({
  counter: state.counter + props.increment,
}));
```

### Lifecycle methods

`componentWillMount` - Before rendering and is used for App level configuration in your root component

`componentDidMount` - After first rendering where all AJAX requests, DOM or state updates, and event listener set up should occur

`componentWillReceiveProps` - When particular prop updates to trigger state transitions

`shouldComponentUpdate` - Determines whether the component is updated or not based on returned boolean

`componentWillUpdate` - Run before re-rendering the component when there are props or state changes

`componentDidUpdate` - Used to update the DOM in response to prop or state changes

`componentWillUnmount` - Used to cancel any outgoing network requests, or remove all event listeners associated with the component before unmounting

## Functional components

- simpler compared to class components
- mainly focus on the UI of the application, not on the behavior
- basically just the render function of a class component
- can have state and mimic lifecycle events using Reach Hooks

### Example

```jsx
import React from 'react';
import PropTypes from 'prop-types';

const MyComponent = ({ foo }) => (
  <div>{foo}<div>
);

MyComponent.propTypes = {
  foo: PropTypes.string
};
```

## Hooks

Hooks enable the extraction and reuse of stateful logic that is common across multiple components without the burden of higher order components or render props. Hooks allow us to easily manipulate the state of our functional component without needing to convert them into class components.

### State

State lets a component “remember” information like user input, both `useState` and `useReducer` are examples.

#### useState

Lets you add a state variable to your component. Takes in `initialState` value or an initializer function (pure, no arguments) and store the value it returns.

```jsx
import { useState } from 'react';
..
const [state, setState] = useState(initialState);
```

##### useReducer

Lets you add a reducer to your component. Takes in a `reducer` function that is pure, should take the state and action as arguments that specifies how the state gets updated and returns the next state. Also takes in `initialArg` which is the value the initial state is calculated from. Lastly the optional `init` initializer function that should return the initial state but if not specified the initial state is set to `initialArg`. Otherwise, the initial state is set to the result of calling `init(initialArg)`. Returns the `current state` of this state variable and a `dispatch` function that lets you change it in response to interaction.

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  if (action.type === "incremented_age") {
    return {
      age: state.age + 1,
    };
  }
  throw Error("Unknown action.");
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button
        onClick={() => {
          dispatch({ type: "incremented_age" });
        }}
      >
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

### Context

Context lets a component receive information from distant parents without passing it as props. For example, your app’s top-level component can pass the current UI theme to all components below, no matter how deep.

#### useContext

```jsx
import { createContext, useContext } from "react";

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = "panel-" + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = "button-" + theme;
  return <button className={className}>{children}</button>;
}
```

### Ref

Refs let a component hold some information that isn’t used for rendering, like a DOM node or a timeout ID. Unlike with state, updating a ref does not re-render your component. Refs are an “escape hatch” from the React paradigm. They are useful when you need to work with non-React systems, such as the built-in browser APIs.

#### useRef

```jsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  ..
  const interval = intervalRef.current;
  ..
  inputRef.current = 1;
```

### Effects

Lets a component connect to and synchronize with external systems. This includes dealing with network, browser DOM, animations, widgets written using a different UI library, and other non-React code.

#### useEffect

Lets you synchronize a component with an external system. This only runs on the client and takes in a set up function, dependencies and if a function is returned then it is run on cleanup. If the dependencies array is empty, the effect runs only once after the initial render.

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

### Performance

A common way to optimize re-rendering performance is to skip unnecessary work. For example, you can tell React to reuse a cached calculation or to skip a re-render if the data has not changed since the previous render.

#### useMemo

Lets you cache the result of a calculation between re-renders. Takes in a calculation function and depencies that will use the cached value from the calculation function if unchanged.

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  ..
}
```

#### useCallback

Lets you cache a function definition between re-renders. It takes in a function definition and dependencies and returns the same function for a given set of dependencies (these are not discarded and are all cached).

```jsx
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  ..
```

#### useTransition

Lets you update the state without blocking the UI. Takes in a `scope` function that updates some state by calling one or more `set` functions. Returns `isPending` flag that tells you whether there is a pending transition, `startTransition` function that lets you mark a state update as a transition.

```jsx
import { useState, useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  ..
}
```

#### useDeferredValue

Lets you defer updating a part of the UI. Takes in a `value` that you want to defer which is returned during initial render. Subsequent re-renders will use the old value and then try to re-render in the background with the new value.

```jsx
import { useState, useDeferredValue } from "react";

export default function App() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

## HOCs

A higher-order component (HOC) is a function that takes a component and returns a new component. It's meant for logic reuse leveraging composition. They can perform state abstraction, props manipulation, render high-jacking.

### Example

```jsx
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log("Current props: ", this.props);
      console.log("Previous props: ", prevProps);
    }

    render() {
      // Filter out extra props that are specific to this HOC and shouldn't be passed through
      const { extraProp, ...passThroughProps } = this.props;

      // Inject props into the wrapped component. These are usually state values or instance methods
      const injectedProp = someStateOrInstanceMethod;

      // Pass props to wrapped component
      return (
        <WrappedComponent injectedProp={injectedProp} {...passThroughProps} />
      );
    }
  };
}
```

also

```jsx
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

## Handling Events

React events are named using camelCase, rather than lowercase. With JSX you pass a function as the event handler, rather than a string.

```jsx
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log("The link was clicked.");
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```

### Input Events

```jsx
function NameForm() {
  const [value, setValue] = useState("");

  function handleChange(e) {
    setValue(e.target.value);
  }

  function handleSubmit(e) {
    alert("A name was submitted: " + value);
    e.preventDefault();
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input type="text" value={value} onChange={handleChange} />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```
