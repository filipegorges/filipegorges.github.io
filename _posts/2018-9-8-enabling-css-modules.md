On this post, we'll examine how to enable the CSS Modules feature on a `create-react-app` generated React application.

## CSS Modules

This feature allows the usage of regular CSS files into our components, importing styles as JavaScript objects, 
allowing the flexibility of CSS without polluting the global space.

### How to enable

**0.** Make sure you commit any changes before doing this, as you won't be able to rollback afterwards!

**1.** Eject the projects dependencies

```bash
# on your project's root folder:
npm run eject
```

Confirm when asked.

**2.** Within `/config/webpack.config.dev.js`, scroll down to `module`, and find the `test: /\.css$/,` line, there,

change from:
```javascript
use: [
  require.resolve('style-loader'),
  {
    loader: require.resolve('css-loader'),
    options: {
      importLoaders: 1,
    },
  },
```

to:
```javascript
use: [
  require.resolve('style-loader'),
  {
    loader: require.resolve('css-loader'),
    options: {
      importLoaders: 1,
      modules: true,
      localIdentName: '[name]__[local]__[hash:base64:5]'
    },
  },
```

**3.** Next, within `/config/webpack.config.prod.js`, again scroll down to `module`, and find the `test: /\.css$/` line again,

change from:
```javascript
use: [
  {
    loader: require.resolve('css-loader'),
    options: {
      importLoaders: 1,
      minimize: true,
      sourceMap: shouldUseSourceMap,
    },
  },
```

to:
```javascript
use: [
  {
    loader: require.resolve('css-loader'),
    options: {
      importLoaders: 1,
      modules: true,
      localIdentName: '[name]__[local]__[hash:base64:5]',
      minimize: true,
      sourceMap: shouldUseSourceMap,
    },
  },
```

If `eslint` errors afterwards, check 
[eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb) 
and reapply the settings stated in the README file

In order to use within the project, create your CSS files 
(keep in mind that these classes will need to be imported in JavaScript, so follow camelcase naming convention), for example:

```css
/* App.css */
.App {
  text-align: center;
}
.AppTitle {
  font-size: 1.5em;
}
```

Then, within your `App.js`, import it like so:
```javascript
import React from 'react';
import classes from './App.css';

const App = () => (
  <div className={classes.App}>
    <h1 className={classes.AppTitle}>Welcome to React</h1>
  </div>
);

export default App;
```
