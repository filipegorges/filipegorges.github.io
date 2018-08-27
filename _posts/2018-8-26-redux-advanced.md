On this post, I'll refer to some more advanced Redux features and techniques.

### Main concepts

* Middlewares
* Redux DevTools
* Action Creators
* Redux Thunk

## Middlwares

Middlewares are basically wrappers, which add functionalities without interrupting the process to which they are applied to.
Within this example, we'll add a logger functionality to our Redux store:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import registerServiceWorker from './registerServiceWorker';

// Include the 'applyMiddleware' function from Redux
import { createStore, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';

import App from './App';
import reducer from './store/reducers/reducer';

// Define the middleware: it'll receive the store,
// which will return a function 'next', which then
// returns a function 'action'; Within it, you'll
// be able to set your middleware logic.
const logger = store => {
    return next => {
        return action => {
            console.log('[MiddleWare] Dispatching', action);
            
            const result = next(action);
            console.log('[Middleware] next state', store.getState());
            
            return result;
        }
    }
};

// Add the 'applyMiddleware' function to 'createStore'
const store = createStore(reducer, applyMiddleware(logger));

const app = (
    <Provider store={store}>
        <App />
    </Provider>
);

ReactDOM.render(app, document.getElementById('root'));
registerServiceWorker();
```

Whenever we dispatch an action, we'll be notified via console by it:

![Logger Middleware](/assets/images/log_middlware.PNG)

## Redux DevTools

Redux DevTools is a Google Chrome extension which allows you to debug Redux applications, with features 
like time-travelling, state/dispatch diff and test generation.

![Redux DevTools Extension](/assets/images/redux-devtools.PNG)

Once added to your Google Chrome browser, you must set up your application to be able to connect to it:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import registerServiceWorker from './registerServiceWorker';

// Include the 'compose' from Redux
import { createStore, compose } from 'redux';
import { Provider } from 'react-redux';

import App from './App';
import reducer from './store/reducers/reducer';

// Set up the necessary enhancer (required by DevTools)
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

// Add the enhancer to 'createStore'
const store = createStore(reducer, composeEnhancers());

const app = (
    <Provider store={store}>
        <App />
    </Provider>
);

ReactDOM.render(app, document.getElementById('root'));
registerServiceWorker();
```

Then on your Chrome browser, access de console (hit F12), go to the newly added 'Redux' tab, and you'll see:

![Redux DevTools Extension Enabled](/assets/images/redux-devtools-enabled.PNG)

## Action Creators

Action Creators are simply functions that return Actions, which are useful for handling asynchronous code.
Below is a comparison before and after adjusting actions to become Action Creators:

**Before**

`actions.js` file:

```javascript
export const INCREMENT = 'INCREMENT';
export const ADD = 'ADD';
export const STORE = 'STORE';
export const DELETE = 'DELETE';
```

`someComponent.js` file:

```javascript
import * as actionTypes from './actions';

// ... some implementation

// this function will map dispatch functionallities
// to prop references you can use within your component
const mapDispatchToProps = dispatch => {
    return {
        onIncrementProperty: () => dispatch({ type: actionTypes.INCREMENT }),
        onAddToProperty: (value) => dispatch({ type: actionTypes.ADD, value: value }),
        onStoreProperty: (propObj) => dispatch({ type: actionTypes.STORE, id: propObj.id, value: propObj.value }),
        onRemoveProperty: (propObj) => dispatch({ type: actionTypes.REMOVE, id: propObj.id }),
    };
};
```

**After**

`actionTypes.js` file:

```javascript
export const INCREMENT = 'INCREMENT';
export const ADD = 'ADD';
export const STORE_RESULT = 'STORE_RESULT';
export const DELETE_RESULT = 'DELETE_RESULT';
```

`increment.js` file:

```javascript
import * as actionTypes from './actionTypes';

export const increment = () => {
    return {
        type: INCREMENT,
    };
};

export const add = (value) => {
    return {
        type: ADD,
        value: value,
    };
};
```

`storage.js` file:

```javascript
import * as actionTypes from './actionTypes';

export const storeResult = (result) => {
    return {
        type: STORE_RESULT,
        result: result,
    };
};

export const deleteResult = (id) => {
    return {
        type: DELETE_RESULT,
        id: id,
    };
};
```

`index.js` file:

```javascript
export {
    add,
    increment,
} from './increment';

export {
    storeResult,
    deleteResult,
} from './storage';
```

`someComponent.js` file:

```javascript
import * as actionCreators from '../../store/actions/index';

const mapDispatchToProps = dispatch => {
    return {
        onIncrementCounter: () => dispatch(actionCreators.increment()),
        onAddCounter: (value) => dispatch(actionCreators.add(value)),
        onStoreResult: (result) => dispatch(actionCreators.storeResult(result)),
        onDeleteResult: (id) => dispatch(actionCreators.deleteResult(id)),
    };
};
```

## Redux Thunk
It's a middleware which enables Action Creators to return a 
function which will dispatch an Action at some point in time, that is, **execute asynchronous code**.

```bash
npm i --save redux-thunk
```

To apply the middleware thunk, it's the same process we saw at the middleware section, 
adding the `thunk` import from the `redux-thunk` package:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import registerServiceWorker from './registerServiceWorker';

// Include the 'applyMiddleware' function from Redux
import { createStore, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';
import thunk from 'redux-thunk';

import App from './App';
import reducer from './store/reducers/reducer';

// Add the 'applyMiddleware' function to 'createStore'.
// You may also apply multiple middlewares, by separating
// them with a comma, e.g.: applyMiddleware(logger, thunk)
const store = createStore(reducer, applyMiddleware(thunk));

const app = (
    <Provider store={store}>
        <App />
    </Provider>
);

ReactDOM.render(app, document.getElementById('root'));
registerServiceWorker();
```

With that, it's now possible to issue async request to APIs, for example:

```javascript
import * as actionTypes from './actionTypes';

// axios is a promise based HTTP client for the browser and node.js
import axios from 'axios';

export const getResults = () => {
  // this is provided by 'redux-thunk'
  return dispatch => {
      axios.get('https://some-destination.com/results.json')
          .then(response => { // If request succeeds
              dispatch(storeResult(response.data));
          })
          .catch(error => { // If it fails
              dispatch(storeResultFailed());
          });
  }
}

export const storeResult = (result) => {
    return {
        type: actionTypes.STORE_RESULT,
        result: result,
    };
};

export const storeResultFailed = () => {
    return {
        type: actionTypes.STORE_RESULT_FAILED,
    };
};
```

now, within the reducer, we'd have:

```javascript
import * as actionTypes from '../actions/actionTypes';

const initialState = {
    results: [],
}

const reducer = (state = initialState, action) => {
    switch(action.type) {
        case actionTypes.STORE_RESULT: return storeResult(state, action);
        case actionTypes.STORE_RESULT_FAILED: return storeResultFailed(state, action);
        default: return state;
    }
}

const storeResult(state, action) {
    return {
        ...state,
        ...action.result,
    }
}

const storeResultFailed(state, action) {
    return {
        ...state,
        error: true,
    }
}
```

then, within your component code:

```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as resultActions from '../../store/actions/index';

class Results extends Component {
    componentDidMount() {
        this.props.getResults();
    }
    
    // ... component implementation
    
    const mapStateToProps = state => {
        return {
            results: state.results,
        };
    };
    
    const mapDispatchToProps = dispatch => {
        return {
            onGetResults: () => dispatch(resultActions.getResults()),
        };
    };
}

```

### Links
[Redux Thunk project](https://github.com/reduxjs/redux-thunk)
[Redux DevTools project](https://github.com/zalmoxisus/redux-devtools-extension)
[Redux DevTools browser plugin](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)
