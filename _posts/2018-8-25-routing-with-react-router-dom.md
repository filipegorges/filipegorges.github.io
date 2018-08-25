In order to add routing to your React project, the standard is to use the `react-router-dom` package:

```bash
npm -i --save react-router-dom
```

The information written here was taught to me on Maximilian Schwarzmüller's great course [React 16 - The Complete Guide (incl. React Router 4 & Redux)](https://www.udemy.com/react-the-complete-guide-incl-redux/).

## React Router DOM

The way routing works in React Router DOM, is it doesn't render a new page, but instead substitutes the rendered components/containers, conditionally.

## Main concepts
* `BrowserRouter`
* `Link`
* `NavLink`
* `Route`
* `Switch`
* `Redirect`
* Programmatic Navigation
* Guard
* Handling 404's
* Lazy Loading Routes 

### BrowserRouter 
enables routing behavior on children components.

**Basic syntax**: 

```javascript
<BrowserRouter>
  <SomeComponent />
</BrowserRouter>
```

### Link
substitutes anchors for accessing "pages".

**Basic syntax**: 

`<Link to="/some-path">Some Component</Link>`

**Supported behaviors**: 
  * `exact`: will only access the component informed to `to={}` if the route is an exact match.
  * `to`: can also receive configurations through an object, for example:
    * `pathname`: string containing the absolute path you wish to match (to get the relative one, you may do it like this: `pathname: props.match.url + '/the-path'`)
    * `hash`: string containing a fragment, e.g.: `#about`. Good for scrollspies.
    * `search`: string containing query params.

### NavLink
substitutes anchors for accessing "pages", and adds styling options. 

**Basic syntax**: 

`<NavLink to="/some-path">Some Component</NavLink>`

**Supported behaviors**: same as `Link`, but supports adding styling attributes to it:
  * `activeClassName`: by default, clicking on a `NavLink` will add a `.active` class to the anchor element. This behavior allows you to substitute that class for a different one.
  * `activeStyle`: allows modification of the current style for the `.active` class.

### Route
A `Route` receives an expression to pattern-match, and a destination; Once the pattern is matched, the destination is called and rendered where the `Route` component was declared. 

**Notes**: 
 * React Router will render every `Route` which `path` matches to the request, therefore if you want to render a single `Route` from a group of `Route`s, consider using `Switch` alongside them.
 * When you have multiple `Route` renders at the same screen, and you have a component that should be re-rendered based on pattern-matching activation, you must implement a `componentDidUpdate()` lifecycle method in order to update it, since **React Router does not unmounts Route rendered components**.

**Basic syntax**:
```javascript
// This route will only render 'Posts' if the route path is EXACTLY "/".
<Route path="/" exact component={Posts} />

// This route will render NewPost once it receives a match on "/new-post".
<Route path="/new-post" component={NewPost} />

// This route will match anything that has "/" + "anything else", 
// and will reference "anything else" as "id", within the 'props.match.params' object.
<Route path="/:id" component={Post} />
```

**Supported behaviors**:
  * `path`: receives an expression to pattern-match, in this case, the desired route.
  * `exact`: same as `<Link />` and `<NavLink />`, the `exact` clause will only evaluate to true (and therefore allow rendering), if the pattern matched is an exact match.
  * `component`: receives the imported component as the argument which will be rendered if the pattern-matching evaluates to true.
  * `render`: receives a function to perform an in-line rendering, e.g.: `<Route path="/" render={() => <h1>Home</h1>} />`.

### Switch
A `Switch` receives children `Route` components, and renders the first, and only the first one, which request matches its route rules.

**Warning**: As stated, `Switch` will only render the first matching `Route`, therefore it is important to be observant of the order which the `Routes` are declared.

**Basic syntax**:

```javascript
<Switch>
    <Route path="/" exact component={Posts} />
    <Route path="/new-post" component={NewPost} />
    <Route path="/:id" exact component={FullPost} />
</Switch>
```

### Redirect
The `Redirect` component changes the navigation **from** a given route path, **to** another route path.
On this example, if the user tries to access "/", it'll be redirected to whatever matches "/posts" route, triggering React Router to attempt matching routes again:

**Basic syntax**:

```javascript
<Route path="/posts" component={Posts} />

<Redirect from="/" to="/posts" />
```

### Programmatic Navigation
If you want to navigate to a given route, from within a method (say, at the end of some validation, or Promise resolve), you can do it through the `history` object from `props`:

**Basic syntax**:

```javascript
itemSelectedHandler = (id) => {

  // You may use either this syntax:
  this.props.history.push({ pathname: '/' + id });
  
  // Or this syntax:
  this.props.history.push('/' + id);
  
}

```

### Guard
If you want to conditionally allow/disallow routing to specific components (for example, when working with authentication and authorization), you have two options:

1- You may add conditions to the rendering of `Route` components:

```javascript
return(
  <div>
    <h1>Some title</h1>
     { this.props.user ? <Route path="/new-post" component={NewPost} /> : null}
     <Route path="/posts" component={Posts} />
  </div>
);
```
This will only allow the route to be considered for rendering if an user is present.

2- You may add conditions to components lifecycle methods:

```javascript
componentDidMount () {
  if (!this.props.user) {
    this.props.history.replace('/posts');
  }
  // Continue intended behavior
}
```

### Handling 404's
To deal with missing pages, add a `Route` that will always match (if using `Switch`, move it to the bottom of the list):

```javascript
const the404 = (
  <div>
      <h1>404</h1>
      <h2>The page you requested does not exist</h2>
  </div>
);

return (
  <Switch>
      <Route path="/posts" component={Posts} />
      <Route render={() => the404} />
  </Switch>
):
```

**Note**: keep in mind that "/" will also always match, so you can't use both these routes at the same `Switch` encapsulation.

### Lazy Loading Routes
In order to spare the user from loading all "pages" from your application, you should definetly consider **lazy loading** (or code-splitting): what that means is that the user will only load the pages it navigates to, lowering the footprint and increasing the access speed.

To do so, create a Higher Order Component `asyncComponent`:

```javascript
import React, { Component } from 'react';

const asyncComponent = (importComponent) => {
    return class extends Component {
        state = {
            component: null
        }

        componentDidMount() {
            importComponent()
                .then(cmp => {
                    this.setState({ component: cmp.default });
                });
        }

        render() {
            const C = this.state.component;
            return C ? <C {...this.props} /> : null;
        }
    }
}

export default asyncComponent;
```

and within the place where you are implementing your `Route`s, instead of directly importing the component like you're used to, do this:

```javascript
// Old way
// import NewPost from './NewPost/NewPost';

// Async loading
import asyncComponent from '../../hoc/asyncComponent';
const AsyncNewPost = asyncComponent(() => {
    return import('./NewPost/NewPost');
});
```

then, you may use within your `Route` definitions:

```javascript
<Route path="/new-post" component={AsyncNewPost} />
<Route path="/posts" component={Posts} />
```



Basic example
------
To get started, add the `BrowserRouter` class to your root component (or the father of the children you want to add routing):

```javascript
import React, { Component } from 'react';
import { BrowserRouter } from 'react-router-dom';

import Blog from './containers/Blog/Blog';

class App extends Component {
  render() {
    return (
      <BrowserRouter>
        <div className="App">
          <Blog />
        </div>
      </BrowserRouter>
    );
  }
}

export default App;
```

Now, within `Blog`, we can add routes to our components/containers:

```javascript
import React, { Component } from 'react';

import { Route, NavLink, Link, Switch } from 'react-router-dom';

import './Blog.css';

import Posts from './Posts/Posts';
import FullPost from './FullPost/FullPost';
import NewPost from './NewPost/NewPost';
import Dashboard from './Dashboard/Dashboard';

class Blog extends Component {

    render () {
        return (
            <div className="Blog">
                <header>
                    <nav>
                        <ul>
                            <li><Link 
                                 to="dashboard"
                                 >Dashboard</Link></li>
                            
                            <li><NavLink 
                                to="/posts" 
                                exact
                                activeClassName="my-active"
                                activeStyle={{
                                    color: '#fa923f',
                                    textDecoration: 'underline'
                                }}>Home</NavLink></li>
                                
                            <li><NavLink to={{
                                pathname: '/new-post',
                                hash: '#submit',
                                search: '?quick-submit=true'
                            }}>New Post</NavLink></li>
                        </ul>
                    </nav>
                </header>
                <Switch>
                  <Route path="/posts" component={Posts} />
                  <Route path="/new-post" component={NewPost} />
                  <Route path="/dashboard" component={Dashboard} />
                  <Route path="/posts/:id" exact component={FullPost} />
                </Switch>
            </div>
        );
    }
}

export default Blog;
```


### Links
* [Official documentation](https://reacttraining.com/react-router/web/guides/philosophy)
* [React Course (Udemy)](https://www.udemy.com/react-the-complete-guide-incl-redux)
