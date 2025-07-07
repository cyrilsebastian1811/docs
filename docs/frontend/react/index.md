# __:material-react: React__

## Key features

- **JSX**: A syntax extension that allows writing HTML in JavaScript.
- **Components**: Reusable and independent building blocks of the UI.
- **Virtual DOM**: A lightweight representation of the real DOM that improves performance by updating only the changed parts.
- **One-Way Data Binding**: Data flows in one direction, making the application more predictable.
- **State and Props**: Enables dynamic and interactive UIs.
- **Lifecycle Methods**: Methods like componentDidMount or useEffect that run during specific phases of a component's lifecycle.

??? note "Virtual DOM"

    The Virtual DOM is a lightweight JavaScript representation of the real DOM. When a React component updates, React calculates the difference between the current Virtual DOM and the new Virtual DOM, and updates only the changed elements in the real DOM, improving performance.

## Setup

- Simplify react setup by using `create-react-app` cli tool.

  ```bash title="create-react-app"
  # install as global dependency
  npm install -g create-react-app

  # using binary to create react apps
  npx create-react-app react-tutorial
  ```

- Use `generate-react-cli` to streamline and automate the process of creating React components, pages, and other related files within a React project.

  ```bash title="generate-react-cli"
  cd react-tutorial
  # install as a project development dependency
  npm install generate-react-cli --save-dev

  # using binary to create components
  npx generate-react-cli component MyComponent

  npx generate-react-cli component MyComponent --type page
  ```

??? info "`generate-react-cli.json` config"

    ```json
      // (1)!
      {
        "usesTypeScript": false,
        "usesStyledComponents": false,
        "usesCssModule": false,
        "cssPreprocessor": "css",
        "testLibrary": "None",
        "component": {
          "default": {
            "path": "src/components",
            "withStyle": false,
            "withTest": false,
            "withStory": false,
            "withLazy": false
          },
          "page": {
            "path": "src/pages",
            "withStyle": false,
            "withTest": false,
            "withStory": false,
            "withLazy": false
          }
        }
      }
    ```

    1.  - `path`: Where to create components (relative to project root)
        - `extension`: File extension: `"js"`, `"jsx"`, `"ts"`, or `"tsx"`
        - `type`: component type
            - `"default"`: A basic functional component (with named export)
            - `"pure"`: A functional component using React.memo() for performance optimization
            - `"class"`: A class-based React component (ES6 class extending React.Component)
        - `withStyle`: If `true`, creates a CSS (or SCSS/module.css) file
        - `style`: `"css"`, `"scss"`, `"module.css"`, `"module.scss"`
        - `withTest`: If `true`, generates a test file
        - `withStory`: If `true`, generates a Storybook file
        - `withLazy`: If `true`, wraps the component in `React.lazy()` with `index.js` as entry point

??? note "`npm` v/s `npx`"

    `npm` is used to:

    - Install packages (locally or globally)
    - Manage dependencies in package.json
    - Run scripts defined in package.json

    `npx` is used to:

    - Run Node.js CLI tools without installing them permanently
    - Run locally installed CLI tools from `node_modules/.bin`

## Component Types

1) Controlled vs. Uncontrolled Components

   - **Controlled**: Components where the state is managed by React using useState or similar.
   - **Uncontrolled**: Components where the state is managed by the DOM itself.

2) Stateful vs. Stateless Components

   - **Stateful**: Components that manage their own state.
   - **Stateless**: Components that do not manage their own state (purely presentational).

## Styling

``` jsx title="dynamic styling"
<input type="text" style={{
  color: !isValid ? 'red' : 'black', 
  height: '80%', backgroundColor: 'red'}}/>

<input type="text" className={`form-control ${!isValid ? 'invalid' : ''}`}/>
```

- React provides, ==scoping== a css file to a specific component. To avoid styling conflicts.

``` css title="MainNavigation.module.css"
.header {
  background: black;
}
```

``` jsx title="MainNavigation.js"
import classes from './MainNavigation.module.css';


const MainNavigation = () => (
  <header className={classes.header}>
    <div>React Meetups</div>
    <nav>
      <ul>
        <li><Link to='/'>All Meetups</Link></li>
        <li><Link to='/new-meetup'>Add New Meetup</Link></li>
        <li><Link to='/favorites'>My Favorites</Link></li>
      </ul>
    </nav>
  </header>
);
```


## Dynamic Rendering

- Pass key while rendering lists for performance and avoid bugs related to states.
- Also, use custom id for list items

``` jsx
function Example() {
    const [states, setStates] = useState(['MI', 'MA', 'VA', 'MD']);

    return (
        <div>
            {/* Conditional Rendering */}
            {states.length === 0 && (<p>No states to show</p>)}

            {/* List Rendering */}
            {states.length > 0 && (states.map((item, index) => <ol key={index}>item</ol>))}

            {/* Alternate Approach */}
            {
                states.length > 0 ?
                    states.map((item, index) => (
                      <li key={index}>item</li>
                    )) : (<p>No states to show</p>)
            }
        </div>
    );
}
```


## Events

- All HTML elements used in React are a wrapper on standard HTML elements. Thus, we must use component properties instead of HTML attributes(e.g use className="" instead of class="").
- React events use camelCase (e.g., onClick, onChange) and accept a function reference — not a string like in plain HTML.

| Event Name | Trigger | Element Type |
| --- | --- | --- |
| `onClick` | Element is clicked | Any |
| `onChange` | Input value changes | `<input>`, `<select>`, `<textarea>` |
| `onSubmit` | Form is submitted | `<form>` |
| `onMouseEnter` | Mouse enters an element | Any |
| `onMouseLeave` | Mouse leaves an element | Any |
| `onFocus` | Element receives focus | Input/form |
| `onBlur` | Element loses focus | Input/form |
| `onKeyDown` | Key is pressed | Any |
| `onKeyUp` | Key is released | Any |

```jsx title="examples"
FormComponent = () => {
    const submitHandler = (event) => {
        event.preventDefault();
    }

    const confirmHandler = (event) => {
        event.preventDefault();
    }

    return (
        <form onSubmit={submitHandler}>
            <button type="submit">Add</button>
            <button onClick={confirmHandler}>Confirm</button>
        </form>
    );
}
```

## Props

- props are used to pass data to child components.
- `children` is a default property in React used to pass nested content. Generally used in wrapper components.

=== "Functional Component"

    ``` jsx title="App.js"
    // App.js
    const App = () => {
      return (
        <div>
          <h1>My Todos</h1>
          // children prop passed by default
          <div className="">{props.children}</div>
          <Todo title="Learn React" />
        </div>
      );
    }
    ```

    ``` jsx title="Todo.js"
    const Todo = (props) => {
      const deleteHandler = (event) => {
        console.log(event);
      }

      return (
        <div className='card'>
          <h2>{props.title}</h2>
          <div className='actions'>
            <button className='btn' onClick={deleteHandler}>Delete</button>
          </div>
        </div>
      );
    };
    ```

=== "Class Component"

    ```jsx title="Counter.js"
    class Counter extends React.Component {
      constructor(props) {
        super(props);
      }

      render() {
        // children prop passed by default
        return <div}>{this.props.children}</div>;
      }
    }
    ```


## State

- State's are immutable. i.e updating the state directly may not trigger react re-render or lifecycle hooks.

### Parent-to-Child communication

=== "Functional Component"

    - React monitors the state by references and not values. Thus for changes to take effect `useState`.

    ``` jsx title="Todo.js"
    const Todo = (props) => {
      // const [ stateVariable, updateFunc ] = useState(initalValue);
      const [modalVisible, setModalVisibility] = useState(false);

      const deleteHandler = (event) => {
        setModalVisibility(true);
      }

      const closeModal = (event) => {
        setModalVisibility(false);
      }

      return (
        <div className='card'>
          <h2>{props.title}</h2>
          <div className='actions'>
            <button className='btn' onClick={deleteHandler}>Delete</button>
          </div>
          { modalVisible && <Modal onCancel={closeModal} onConfirm={closeModal}/> }
          { modalVisible && <Backdrop onCancel={closeModal}/> }
        </div>
      );
    };
    ```

=== "Class Component"

    ```jsx title="Counter.js"
    class Counter extends React.Component {
      constructor(props) {
        super(props);
        this.state = { count: 0 };
      }

      increment = () => {
        this.setState({ count: this.state.count + 1 });
      };

      render() {
        return <button onClick={this.increment}>{this.state.count}</button>;
      }
    }
    ```


### Child-to-Parent communication

In React, child components can communicate with parent components via callback functions passed as props.

```jsx
// De-Structuring
// {prop1, prop2, prop2} = props
const Child = ({updateState}) => {
  return <button onClick={updateState('Child')}></button>
}

const Parent = () => {
  const [name, setName] = useState('Parent');

  handleUpdateState = (value) => {
    setName(value);
  }

  return <Child updateState={handleUpdateState}></Child>
}
```


## Component Lifecycles

React components go through different lifecycle phases during their existence. These phases are particularly relevant for class components but can also be mapped to functional components using hooks like useEffect.

=== "Class Components"

    === "__Mounting Phase__"
        The mounting phase occurs when a component is being created and inserted into the DOM.

        ??? info "`#!jsx constructor()`"
            - Invoked before the component is mounted.
            - Used for initializing state and binding event handlers.

            ```jsx
            constructor(props) {
              super(props);
              this.state = { counter: 0 };
            }
            ```

        ??? info "`#!jsx static getDerivedStateFromProps()`"
            - Invoked before render.
            - Allows the component to update its state based on props.

            ```jsx
            static getDerivedStateFromProps(props, state) {
              if (props.initialCount !== state.counter) {
                return { counter: props.initialCount };
              }
              return null;
            }
            ```

        ??? info "`#!jsx render()`"
            - The only required method in a class component.
            - Describes the UI structure.

            ```jsx
            render() {
              return <h1>{this.state.counter}</h1>;
            }
            ```

        ??? info "`#!jsx componentDidMount()`"
            - Invoked after the component is mounted in the DOM.
            - Ideal for side effects like API calls, subscriptions, or setting up event listeners.

            ```jsx
            componentDidMount() {
              fetch('/api/data').then(response => response.json());
            }
            ```

    === "__Updating Phase__"
        The updating phase occurs when a component is re-rendered due to changes in state or props.
        ??? info "`#!jsx static getDerivedStateFromProps()`"
            - Invoked when props change. It can update the state based on new props (rarely used).

        ??? info "`#!jsx shouldComponentUpdate()`"
            - Invoked before rendering. Determines if a re-render is needed.
            - Returns a boolean (true by default).

            ```jsx
            shouldComponentUpdate(nextProps, nextState) {
              return nextState.counter !== this.state.counter;
            }
            ```

        ??? info "`#!jsx render()`"
            - Re-renders the component.

        ??? info "`#!jsx getSnapshotBeforeUpdate()`"
            - Called just before the DOM is updated.
            - Allows capturing information about the DOM (e.g., scroll position).

            ```jsx
            getSnapshotBeforeUpdate(prevProps, prevState) {
              return { scrollTop: document.documentElement.scrollTop };
            }
            ```

        ??? info "`#!jsx componentDidUpdate()`"
            - Invoked after the DOM is updated.
            - Useful for performing side effects after updates, like making network requests.

            ```jsx
            componentDidUpdate(prevProps, prevState, snapshot) {
              if (prevState.counter !== this.state.counter) {
                console.log('Counter updated!');
              }
            }
            ```

    === "__Unmounting Phase__"
        This phase occurs when a component is removed from the DOM.
        ??? info "`#!jsx componentWillUnmount()`"
            - Invoked immediately before a component is unmounted and destroyed.
            - Used for cleanup tasks like removing event listeners, canceling network requests, or clearing timers.

            ```jsx
            componentWillUnmount() {
              clearInterval(this.timer);
            }
            ```

    === "__Error Handling Phase__"
        This phase is triggered when an error occurs during rendering, lifecycle methods, or constructors of child components.
        ??? info "`#!jsx static getDerivedStateFromError()`"
            - Invoked after an error is thrown.
            - Updates state to display an error UI.

            ```jsx
            static getDerivedStateFromError(error) {
              return { hasError: true };
            }
            ```

        ??? info "`#!jsx componentDidCatch()`"

            - Invoked after an error is thrown in a child component.
            - Used for logging error information
            - It can wrap around components to handle the error globally/aggregate error handling for set of components.
            - ==No equivalent for functional components==

            ```jsx
            componentDidCatch(error, info) {
              logErrorToService(error, info);
            }
            ```

            ```jsx title="Use case"
            class ErrorBoundry extends Component {
                constructor() {
                    super();
                    this.state = { hasError: false };
                }

                componentDidCatch(error) {
                    console.log(error);
                    this.setState({ hasError: true });
                }

                render() {
                    return (this.state.hasError ? (<p>Something went wrong</p>) : this.props.children);
                }
            }

            function ChildComponent {
                handleClick = () => {
                    throw new Error("Error from child Component");
                }

                return(
                    <button onClick={handleClick}>Throw error</button>
                );
            }


            return (
                <ErrorBoundry>
                    // ChildComponent thow
                    <ChildComponent ...props>
                </ErrorBoundry>
            );
            ```


=== "Functional Components"
    For functional components, React hooks like useEffect can mimic lifecycle behaviors:

    ??? danger "On every render"

        - Be careful not to change the state in this case.
        - Changing state cause re-rendering, so on and so forth. Thus entering a loop.

        ```jsx
        useEffect(() => {
          setLocalState1();         // ❌ will create a loop
          console.log('Component updated'); 
        });
        ```

    ??? info "Mounting Phase"

        ``` jsx
        useEffect(() => {
          console.log('Component mounted');
        }, []);
        ```

    ??? warning "When state/prop changes"

        - React tracks changes using reference equality (===)
            - `user.name` is a primitive string, so React can tell when it changes.
            - Including `user.name` is usually sufficient, not `user`.
        - However,
            - If `user` is recreated (new reference) on every render, then `[user.name]` won’t detect the change — but `[user]` will.
            - If `user.name` is nested (e.g. `user.profile.name`) and `user.profile` changes, React won’t catch that unless you include the deeper object.

        ``` jsx title="simple states"
        useEffect(() => {
          console.log(count);                   // state: count
        }, [count]);
        ```

        ``` jsx title="complex states"
        useEffect(() => {
          console.log(user.profile.name);       // state: user = { profile: {name: } }
        }, [user.profile.name]);
        ```

    ??? info "Unmounting Phase"

        ```jsx
        useEffect(() => {
          return () => {
            console.log('Component will unmount');
          };
        }, []);
        ```


## Hooks

They let you use state and other React features without writing a class(i.e functional components)

### State Hook

```jsx
const Example = () => {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

- `useState` returns a pair, the current state value and a function that lets you update it.
- Similar to `this.setState` in a class, except it doesn’t merge the old and new state together.
- Hence, if current state depends on previous state.

```jsx 
function Example() {
    const [userInput, setUserInput] = useState({
        title: '',
        firstName: '',
        lastName: ''
    });

    handleClick = () => {
        // 1. Not recommended
        setUserInput({
            ...userInput,
            lastName: 'test'
        });

        // 2. Recommended
        // The 1st approach may return stale data as react schedule's state updates
        setUserInput((prevState) => {
            return {
                ...prevState,
                lastName: 'test'
            };
        })
    };

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => handleClick()}>
                Click me
            </button>
        </div>
    );
}
```

### Ref's Hook

- Used to reference a HTML tag
- Suitable, to read input values from forms once(on submission). Rather than monitoring every e event.

```jsx
import React, { useRef } from 'react';
const Example = () => {
  const nameInputRef = useRef();

  handleClick = () => {
    // The node element can be found in the current property
    console.log(nameInputRef.current.value);
  }

  return (
    <div>
      <input type="text" ref={nameInputRef} />
      <button onClick={handleClick}></button>
    </div>
  );
}
```

### Effects hook

- It serves the same purpose as componentDidMount, componentDidUpdate, and componentWillUnmount in React classes, but unified into a single API. ex:  `useEffect(() => {...}, [dependencies])}`

- It runs after every component evaluation,, if the specified dependencies changes

```jsx
import { useEffect, useState } from 'react';

let myTimer;

const MyComponent = (props) => {
  const [timerIsActive, setTimerIsActive] = useState(false);

  const { timerDuration } = props; // using destructuring to pull out specific properties

  useEffect(() => {
    if (!timerIsActive) {
      setTimerIsActive(true);
      myTimer = setTimeout(() => {
        setTimerIsActive(false);
      }, timerDuration);
    }
  }, [timerIsActive, timerDuration]);
};

# Runs every time component renders
useEffect(() => {});

# Runs only once, right after component mounts
useEffect(() => {}, []);

# Runs whenever dependency changes
useEffect(() => {}, [...dependencies]);
```

<figure markdown="span">
  ![Component Lifecycle](./img/component-lifescycle.png){ width="600" }
  <figcaption>Class-based Component Lifecycle</figcaption>
</figure>


## Fragment

- Using Fragments to Avoid Unnecessary Wrappers

```jsx
import React, { Fragment } from 'react';
function Example() {
    return (
        // avoid unnecessary div's
        <Fragment>
            <AddUser onAddUser={}></AddUser>
            <UserList items={}></UserList>
        </Fragment>
    );
}
```


## Portal

- A portal provides a way to render children into a DOM node that exists outside the DOM hierarchy of the parent component.

```jsx
function Modal({ isOpen, onClose, children }) {
    if (!isOpen) return null;

    // Create a portal to render modal content outside the parent component
    return ReactDOM.createPortal(
        <div style={modalStyles}>
            <div style={overlayStyles}>
                <div style={contentStyles}>
                    {children}
                    <button onClick={onClose}>Close</button>
                </div>
            </div>
        </div>,
        document.getElementById('portal-root') // Mount modal content in a DOM node outside the root
    );
}

function App() {
    const [isModalOpen, setModalOpen] = React.useState(false);

    return (
        <div>
            <h1>React Portal Example</h1>
            <button onClick={() => setModalOpen(true)}>Open Modal</button>
            <Modal isOpen={isModalOpen} onClose={() => setModalOpen(false)}>
                <h2>Modal Content</h2>
                <p>This content is rendered outside the main DOM tree using a portal.</p>
            </Modal>
        </div>
    );
}
```


## Redux

Provides application wide immutable store. Best used for maintaining app level state, similar to context

```jsx
import { createStore, combineReducers } from 'redux';

const counterReducer = (state = { counter: 0, showCounter: true }, action) => {
  switch (action.type) {
    case "INCREMENT": return {
      counter: state.counter + 1,
      showCounter
    }
    case "DECREMENT": return {
      counter: state.counter - 1,
      showCounter
    }
    case "INCREASE": return {
      counter: (state.counter + action.amt), showCounter
    }
    case "TOGGLE": return {
      counter: state.counter,
      showCounter: !state.showCounter
    }
  }
  return state;
}

const store = createStore(combineReducers({
  counter: counterReducer
}))

// REDUX IIMPLEMENTATION
// Subscribing to a state
const counterSubscriber = () => {
  const state = store.getState();
  console.log(state);
}

store.subscribe(counterSubscriber);

store.dispatch({ type: "INCREMENT" })


// REACT-REDUX IIMPLEMENTATION
import { Provider, useSelector, useDispatch } from 'react-redux';

// Add provider, such that all nested components has access to the store
// Hence generally added to App(Root) component
<Provider store={store}>
  <App />
</Provider>

const Counter = () => {
  // get's and subscribe's to the state
  const counter = useSelector(state => state.counter);
  const showCounter = useSelector(state => state.showCounter);
  const dispatch = useDispatch();

  const incrementHandler = () => {
    dispatch({ type: "INCREMENT" });
  }

  const increaseHandler = () => {
    dispatch({ type: "INCREASE", amt: 5 });
  }

  const decrementHandler = () => {
    dispatch({ type: "DECREMENT" });
  }

  const toggleHandler = () => {
    dispatch({ type: "TOGGLE" });
  }

  return (
    <div>
      Counter
      {showCounter && <span>{counter}</span>}
      <button onClick={incrementHandler}>Increment</button>
      <button onClick={increaseHandler}>Increase by 5</button>
      <button onClick={decrementHandler}>Decrement</button>
      <button onClick={toggleHandler}>Toggle Counter</button>
    </div>
  );
}

export default Counter;
```

### Limitations of react-redux

- Could lead to misspell or use of wrong action identifiers when working on a project with multiple actions and multiple developer contribution. Could be avoided by spelling action identifiers correctly
- Since states are immutable. We return a cloned state with the necessary updates, and as the state get complex so does the reducer. This could lead to unmanageable code
- No modularization of reducers

!!! note

    redux/toolkit is another library that address the above mentioned short comings of react-redux

```jsx
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { counter: 0, showCounter: true },
  reducers: {
    increment(state) {
      // possible becuase Immer package automatically intercepts this
      // and return a cloned verions of the state with the below update
      state.counter++;
    },
    decrement() {
      state.counter--;
    },
    increase(state, action) {
      state.counter = state.counter + action.amt;
    },
    toggle(state) {
      state.showCounter = !state.showCounter;
    }
  }
})

const store = configureStore({
  reducer: {
    // counterReducer key -> counter
    counter: counterSlice.reducer
  }
})

import { Provider, useSelector, useDispatch } from 'react-redux';

// Add provider, such that all nested components has access to the store
// hence generally added to App(Root) component
<Provider store={store}>
  <App />
</Provider>

const Counter = () => {
  // get's and subscribe to the state
  // state.<store_reducer_key>.<properties>
  const counter = useSelector(state => state.counter.counter);
  const showCounter = useSelector(state => state.counter.showCounter);
  const dispatch = useDispatch();

  const incrementHandler = () => {
    dispatch(counterSlice.actions.increment());
  }

  const increaseHandler = () => {
    dispatch(counterSlice.actions.increase({ amt: 5 })); // {type: SOME_UNIQUE_IDENTIFIER, amt: 5}
    // dispatch(counterSlice.actions.increase(5)); // defaults {type: SOME_UNIQUE_IDENTIFIER, payload: 5}
  }

  const decrementHandler = () => {
    dispatch(counterSlice.actions.decrement());
  }

  const toggleHandler = () => {
    dispatch(counterSlice.actions.toggle());
  }

  return (
    <div>
      Counter
      {showCounter && <span>{counter}</span>}
      <button onClick={incrementHandler}>Increment</button>
      <button onClick={increaseHandler}>Increase by 5</button>
      <button onClick={decrementHandler}>Decrement</button>
      <button onClick={toggleHandler}>Toggle Counter</button>
    </div>
  );
}

export default Counter;
```

## Routing

- `BrowserRouter` initializes the `App` component with routing capabilities.
- `Route` componet listens on url events to render the necessary component at the specified location.

```jsx
import { BrowserRouter, Route } from 'react-router-dom';

root.render(
  <BrowserRouter><App /></BrowserRouter>
);

const App = () => {
  return (
    <div>
      // matches all /welcome/*
      <Route path="/welcome"><Welcome /></Route>
      // matches all /products/*
      <Route path="/products"><Products /></Route>
    </div>
  );
}
```


### Switch

- Switch renders one active route at a time.
- In the example below both Products(`/products`) and ProductDetails(`/products/:productId`) matches `/products/p1` url. Thus, `<Products />` will be rendered
- `exact` checks for an exact url match. e.g. `/products/p1` renders the first match(<Products />) followed by the second match(<ProductDetails />).

```jsx 
import { Switch, Route } from 'react-router-dom';

const App = () => {
  return (
    <div>
      <Switch>
        <Route path="/welcome"><Welcome /></Route>
        <Route path="/products" exact><Products /></Route> // matches all /products* paths
        <Route path="/products/:productId"><ProductDetail /></Route> // matches all /products/* paths
      </Switch>
    </div>
  );
}
```


### Link and NavLink

- In order to render the Welcome or Products component, user would have to manually change the url to either `/welcome` or `/products` . But it's common to have links in the app to manipulate the url and sequentially render the appropriate component.
- Both Link and NavLink creates an <a> tag implicitly

```jsx
<nav>
  <ul>
    <li><Link to="/welcome">Welcome</Link></li>
    <li><Link to="/products">Welcome</Link></li>

    {/* NavLink can be used to provide custom styling to active link */}
    <li><NavLink activeClassName="active" to="/orders">Orders</Link></li>
    <li><NavLink activeClassName="active" to="/users">Users</Link></li>
  </ul>
</nav>
```


### Path and Query parameters

```jsx
// dynamic routes 
// <Route path="/products/:productId/:userId?sort=asc"><ProductDetail /></Route>

import { useParams, useLocation } from 'react-router-dom';

const ProductDetail = () => {
  // path params
  const params = useParams();

  //
  const location = useLocation();

  // query params
  const queryParams = new URLSearchParams(location.search);

  return (
    <div>
      <h1>{params.userId}. This is the Product: {params.productId}. sorting order {queryParams.get('sort')}</h1>
    </div>
  );
}
```


### Nested Route

``` jsx
const Welcome = () => {
  return (
    <section>
      <h1>Welcome Page</h1>
      {/* Nested Route. This Route is evaluted if /welcome route is active */}
      {/* /welcome -> Welcome component */}
      {/* /welcome/new-user -> Welcome and NewUser component */}
      <Route path="/welcome/new-user">
        <h1>Welcome, new user!</h1>
      </Route>
    </section>
  );
}
```

You can have Route in other Routes

```jsx
import { Switch, Route } from 'react-router-dom';
const App = () => {
  // Route componet listens to url events to render the necessary component at the specified location
  return (
    <div>
      <Switch>
		{/* without exact, the app would be stuck in an infinite loop(everything matches /) */}
        <Route path="/" exact>
		      <Redirect to="/welcome" />
	 	    </Route>
        <Route path="/welcome"><Welcome /></Route>
        <Route path="/products" exact><Products /></Route> # matches all /products* paths
        <Route path="/products/:productId"><ProductDetail /></Route> {/* matches all /products/* paths */}
      </Switch>
    </div>
  );
}
```

### Programatic Navigation

```jsx
import { useHistory, useNavigate } from "react-router-dom";
const history = useHistory();

history.push("/quotes"); {/* retains the history. i.e can navigate back to current page upon redirect*/}
history.replace("/quotes"); {/* history is lost. i.e can't navigate back to current page upon redirect*/}

{/* https://reactrouter.com/en/main/hooks/use-navigate */}
const navigate = useNavigate();
navigate("/quotes", { replace: true });
```
