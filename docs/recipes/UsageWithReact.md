# Using Redux with React

This is a quick recipe of a way of integrating Redux into your React project using a traditional flux flow.

## Installation

As of redux 1.0, the React integration and components have been removed and added to an isolated repository, called `react-redux`.  First install this package via `npm install --save react-redux`.

## Constants

While there are other options to using constants, they are surely the easiest to get started.  Define them and export them so that our actions and stores can access them.

```
export const FETCH_MESSAGES = "FETCH_MESSAGES";
```

## Actions

Actions are functions that get dispatched to the redux stores.  They are called in your components whenever you need to fetch or act on data (i.e., an API GET or POST).  They can be synchronous or asynchronous, depending on what you're doing.  Here is an example of using our constants, and fetching all the message from an API.  `axios` is just a promised based library for making HTTP requests.

```
import axios from "axios";
import { FETCH_MESSAGES } from "./constants";

export function fetchMessages() {
  return dispatch => (
    axios.get("http://website.com/api/messages")
      .then(response => (
        dispatch({
          type: FETCH_MESSAGES,
          data: response.data
        })
      )
      .catch(err => console.error(err))
    )
}
```

The key point is that, because we are making a promised based request, rather than simply returning the `{ type: FETCH_MESSAGES, data: response.data }`, we are `dispatch`ing it when the data has actually been fetched.

If we were making a synchronous action, the code would look like the following:

```
import { FETCH_MESSAGES } from "./constants";

export function fetchMessages() {
  return {
    type: FETCH_MESSAGES,
    data: "Message here"
  }
}
```

## Stores

Stores are just reducers (functions) that take in dispatched data and provide you an opportunity to consolidate the data how you like.  They have an initial state that you have to define.  Again, in accordance with using constants, the simplest stores have a switch case structure.

```
import { FETCH_MESSAGES } from "./constants";

export default function messages(state = [], action) {
  switch (action.type) {
  case FETCH_MESSAGES:
    return action.data;
  default:
    return state;
  }
}
```

Make sure that you are never mutating the state as it is being returned.  Ideally, you are using some sort of immutable data type so that this would never happen.  For example, the prior example would like like the following using `immutable`.

```
import Immtuable from "immutable";
import { FETCH_MESSAGES } from "./constants";

export default function messages(state = Immutable.List(), action) {
  switch (action.type) {
  case FETCH_MESSAGES:
    return Immutable.fromJS(action.data).toList();
  default:
    return state;
  }
}
```

When your stores are ready, export them in a single file so that they are easier to access (index.js):

```
export { default as messages } from "./messages";
```

## Middleware

As of 1.0, redux no longer includes any middleware.  Middleware has been left open for you to determine what you need for your app.  In our example, we simple need some middleware to handle our promised based actions.  For sake of simplicity, I suggest using the `redux-thunk` middleware.  Install this package so we can use it for initialization.

## Provider

So, now that the stores and actions are setup, we are finally ready for React integration.  At the entry point of the app, we have to initialize redux and provide it for the children components in context.

```
import React from "react";

import { createStore, combineReducers, applyMiddleware } from "redux";
import { Provider } from "react-redux";
import * as stores from "./stores/index";
import thunk from "redux-thunk";

import App from "./App"; // Your entry point component

const createStoreWithMiddleware = applyMiddleware(thunk)(createStore);
const reducer = combineReducers(stores);
const store = createStoreWithMiddleware(reducer);

export default class AppHandler extends React.Component {
  render() {
    return (
      <Provider store={ store }>
        {
          () => <App />
        }
      </Provider>
    );
  }
}

```

This initializes, combines our stores, and passes down redux for usage in the children components.

## Connector

The Connector component has been provided as a wrapping component so that your children components can listen to changes in their respective stores.  It passes down the store's states as props so that your dumb components can simply have the data and actions they need via props.

In any child component of the `<Provider />`, you can do the following:

```
import React from "react";
import { Connector } from "redux-react";
import { bindActionCreators } from "redux";
import { fetchMessages } from "./actions";

import ChildComponent from "./ChildComponent";

// You can optionally specify `select` for finer-grained subscriptions
// and retrieval. Only when the return value is shallowly different,
// will the child component be updated.
function select(state) {
  return { messages: state.messages };
}

export default class App extends React.Component {
  render() {
    return (
      <Connector select={select}>
        {({ counter, dispatch }) =>
          /* Yes this is child as a function. */
          <ChildComponent messages={ messages } 
                   {...bindActionCreators(fetchMessages, dispatch)} />
        }
      </Connector>
    );
  }
}
```

Now, `ChildComponent` is listening to all updates from the `messages` store.  The component has in their `props` `messages` and `fetchMessages`.  So you can fire the action to fetch all the messages in that component, and listen to the updates the action causes.  The component will re-render as necessary like a state update.

## @connect

## Using context as access to the store

## Usage with react-router
