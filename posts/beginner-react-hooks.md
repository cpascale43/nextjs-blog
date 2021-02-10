---
title: "Replace Redux with React Hooks"
date: "2020-06-17"
---

In React, you may have come across what's known as the "prop drilling" problem. This is what happens when you pass props down from the top of your component tree to the bottom. It gets tedious! Redux is a state management library commonly used with React which lets us avoid this.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/qgu0nis9lf9e95s3ul2m.png)

However, the Context API was released in [React 16.3](https://reactjs.org/docs/hooks-overview.html#state-hook):

> Context provides a way to pass data through the component tree without having to pass props down manually at every level.

Huh? Does this mean I don't need Redux anymore? Let's refactor an app that uses Redux and see what happens.

## Setup

We're going to start from [this repo](https://github.com/cpascale43/BeginnerReactHooks). Go ahead and fork & clone to your local machine.

## useState & React Context

If you looked at our app and thought, wow, that's a lot of code for a todo-list app...you're right! It is. We don't need to use action creators, dispatch, or connect.

The two Hooks we'll be using are `useState` and `useContext`. From the [React docs] (https://reactjs.org/docs/hooks-overview.html#state-hook):

> Hooks are functions that let you “hook into” React state and lifecycle features from function components. Hooks don’t work inside classes — they let you use React without classes.

`useState` allows you to create and update state within functional components. It takes one argument - the initial state - and returns two values: a state value, which you can name whatever you want, and a function that lets you update this value.

Meaning, something like this...

```javascript
const [input, setInput] = useState("");
```

...is equivalent to this (assuming you call `this.setState` somewhere else):

```javascript
  constructor(props) {
    super(props);
    this.state = {
      input: "",
    };
  }
```

You may have heard of `useState` already, but if Context is new, it basically allows you to use global state across components.

This is important because once you declare your state using `useState`, you'll need to lift it up to become global state using React Context. You'll do this in your components via a second Hook: `useContext`.

#### Are you with me so far?

- Context API
- useState
- useContext

## Getting started with React Context

The first step in our journey is to create our Context object. We'll do this using the createContext function provided by React.

In `client/context.js`, create your Context object.

```javascript
// Create context object
export const ListContext = createContext();
```

**To make this Context available to all of our components, we have to use a Context provider.** According to the React docs, "every Context object comes with a Provider React component that allows consuming components to subscribe to context changes."

This means that whatever we pass as a value to our provider will be passed to components that are descendants of this provider.

#### client/context.js

```javascript
import React, { useState, createContext } from "react";

// Create Context object
export const ListContext = createContext();

// Create a provider for components to consume and subscribe to changes
export const ListContextProvider = (props) => {
  const [tasks, setTasks] = useState([]);

  return (
    <ListContext.Provider value={[tasks, setTasks]}>
      {props.children}
    </ListContext.Provider>
  );
};
```

Look closely at our provider. It looks like a regular React component, right? If we wrap all of our components in this provider, they will be able to access the global state.

## Refactor components

We're going to transform class components into functional components, create local state using `useState`, and hook into the global state via `useContext`.

### AddItem.js

Let's get Redux out of the way. Delete `connect`, `addItem`, `mapDispatchToProps`, and set your default export to the `AddItem` component.

Change the class component to a functional component, and delete the constructor. Set the local state to an empty string, like this:

```javascript
const [input, setInput] = useState("");
```

Now, refactor `handlekey`:

```javascript
const handleKey = (evt) => {
  if (input === "") return;
  if (evt.key === "Enter") {
    setInput("");
  }
};
```

Replace any reference to `this.state.input` with simply, `input`. Similarly, any function call involving `this.setState` should now call `setInput`.

Next, see if you can log out the value of `input` to your console from within `handleKey`.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/1ymwoje7nhje6zx29a8s.png)

Congratulations! You've successfully implemented `useState`. Your `AddItem` component now has its own local state based on the input. This is awesome. Pats on the back.

Our next task is to tackle `useContext`. Let's go for it!

Import `useContext` into the top of your file, and place it below `useState`.

```javascript
const [input, setInput] = useState("");
const [tasks, setTasks] = useContext(ListContext);
```

Now, when do you need to make changes to the list of tasks? Probably whenever a user is pressing "Enter," or clicking the "Add Task" button. See if you can figure out where to add this.

```javascript
setTasks([...tasks, input]);
```

Your code may look something like this:

#### components/AddItem.js

```javascript
import React, { useState, useContext } from "react";
import { ListContext } from "../context";

const AddItem = () => {
  const [input, setInput] = useState("");
  const [tasks, setTasks] = useContext(ListContext);

  const handleKey = (evt) => {
    if (input === "") return;
    if (evt.key === "Enter") {
      setTasks([...tasks, input]);
      setInput("");
    }
  };

  return (
    <div className="input-group mb-3">
      <input
        type="text"
        className="form-control"
        placeholder="Tasks go here..."
        aria-label="Your items here"
        aria-describedby="button-addon2"
        value={input}
        onChange={(evt) => setInput(evt.target.value)}
        onKeyDown={handleKey}
      />
      <div className="input-group-append">
        <button
          className="btn btn-outline-primary"
          type="button"
          id="button-addon2"
          onClick={() => {
            if (input === "") return;
            setTasks([...tasks, input]);
            setInput("");
          }}
        >
          Add Task
        </button>
      </div>
    </div>
  );
};

export default AddItem;
```

### List.js

Let's move on to `List.js`. Overall, what we need to do is refactor how tasks are being referenced. Instead of our `connect` function mapping tasks from the global Redux store to List's props, we're going to hook into the context object directly.

Here's how we'll do it:

- Connect to the Context Object with `useContext`
- Create a toggleItem function (there are a lot of ways you could do this!)
- Change `props.items` to `tasks`

Give this one a shot! If you get stuck, here's what I came up with.\*\*

#### components/List.js

```javascript
import React, { useContext } from "react";
import { ListContext } from "../context";

const List = () => {
  const [tasks, setTasks] = useContext(ListContext);

  const toggleItem = (idx) => {
    const deleted = tasks[idx];
    const newTasks = tasks.filter((task) => task !== deleted);
    setTasks(newTasks);
  };

  return (
    <ul>
      {tasks.map((item, idx) => {
        return (
          <div key={idx} className="row p-3">
            <div className="col">
              <li>{item}</li>
            </div>
            <div className="col d-flex justify-content-end">
              <button
                onClick={() => toggleItem(idx)}
                type="button"
                className="btn btn-danger btn-sm"
              >
                Remove
              </button>
            </div>
          </div>
        );
      })}
    </ul>
  );
};

export default List;
```

### App.js

At some point during this exercise, you may have noticed this error message:
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pvm8jm79y9p5htn05ne2.png)

What is this? What does it mean?!

Well, remember when I said this?

> If we wrap all of our components in this Provider, they will be able to access the global state.

We forgot to wrap our app in our provider! Let's go ahead and do it now.

#### client/components/App.js

```javascript
import React from "react";
import AddItem from "./AddItem";
import List from "./List";
import { ListContextProvider } from "../context";

const App = () => (
  <ListContextProvider>
    <div className="container p-5">
      <h1 className="display-3">
        A List<small className="text-muted"> with React Hooks</small>
      </h1>
      <AddItem />
      <div className="card scroll shadow-sm p-3 mb-5 bg-white rounded">
        <List />
      </div>
    </div>
  </ListContextProvider>
);

export default App;
```

We're almost done! Head to `client/index.js` and remove the Redux provider. Feel free to remove Redux dependencies from your `package.json`, and to delete `store.js`.

You should be up and running now! This is awesome.

## What did we learn?

React has a native way for you to manage state via Context and Hooks. We learned:

- How to create a global Context object + Context provider to wrap components in, with a global state.
- `useState` allows functional components to access state.
- `useContext` lets us hook into our global Context object and make changes to our global state from within components.

5 stars for a job well done. If you got stuck at any point, check out the full solution [here](https://github.com/cpascale43/BeginnerReactHooksSolution).

## Happy coding!👋

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/wf259v352edpgfwhws66.png)

### \*You may not always want to use React Context. From the React docs: "If you only want to avoid passing some props through many levels, component composition is often a simpler solution than context." Read more [here](https://reactjs.org/docs/context.html#before-you-use-context).

### \*\*When refactoring, I combined `Item.js` and `List.js`.
