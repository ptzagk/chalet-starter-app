## Chalet Template Kit
Chalet template kit is the reborn [react redux starter kit](https://github.com/davezuko/react-redux-starter-kit) with the advantage of [create-react-app](https://github.com/facebookincubator/create-react-app).

The *react redux starter kit* was mature project and contained much production-based npm package such as [redux](https://github.com/reactjs/redux), [redux-thunk](https://github.com/gaearon/redux-thunk), [react-router](https://github.com/ReactTraining/react-router), and so on. Furthermore, its [Fractal Project Structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure) is a creative concept and make the folder structure clear. It's absolutely one of awesome starter kits. Unfortunately, it was [deprecation](https://github.com/davezuko/react-redux-starter-kit#deprecation-warning).

*Create react app* was a good substitution. It has an [elaborate user guide](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md) and [useful scripts](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#available-scripts) which make developers do not have to configure *webpack*, *babel*, *eslint*, and so on. This saved a lot of effort and developers could concentrate their attention on business logic.

Accordingly, the project not only succeeded to *react redux starter kit* but also applied the advantage of *create react app* to it.


## Quick Overview
Like *create-react-app*, you don't need to use `git clone <the app repository>` every time you start a new project.

Just install **create-starter-app** instead of create-react-app at first.

```sh
npm install -g create-starter-app

create-starter-app my-app
cd my-app/
yarn start
```

Then open [http://localhost:3000/](http://localhost:3000/) to see your app.<br>
When you’re ready to deploy to production, create a minified bundle with `yarn run build`.

<img src='https://camo.githubusercontent.com/506a5a0a33aebed2bf0d24d3999af7f582b31808/687474703a2f2f692e696d6775722e636f6d2f616d794e66434e2e706e67' width='600' alt='npm start'>

## Style React Components
There are [four ways to style react components](https://medium.com/@aghh1504/4-four-ways-to-style-react-components-ac6f323da822). Chalet template kit offers two of them, *CSS Stylesheet* and *CSS Modules*. Just use the former as usual. However, it apply `mcss` file extension to identity the later. They are both scss sytax but a little different in JSX. React css modules component automates loading of CSS Modules using **styleName** property.

```jsx
import 'styleContainsFoo.scss'
import 'styleContainsBar.msss'

<span className='Foo'>Hello</span>   
// Foo is global css stylesheet.

<span styleName='Bar'>World</span>
// Boo is modular and do not pollute any other css stylesheet.
```

## Fractal Project Structure
_Also known as: Self-Contained Apps, Recursive Route Hierarchy, Providers, etc_

Small applications can be built using a flat directory structure, with folders for `components`, `containers`, etc. However, this structure does not scale and can seriously affect development velocity as your project grows. Starting with a fractal structure allows your application to organically drive its own architecture from day one.

This structure provides many benefits that may not be immediately obvious:
* Routes can be bundled into "chunks" using webpack's [code splitting](https://webpack.github.io/docs/code-splitting.html) and merging algorithm. This means that the entire dependency tree for each route can be omitted from the initial bundle and then loaded *on demand*.
* Since logic is self-contained, routes can easily be broken into separate repositories and referenced with webpack's [DLL plugin](https://github.com/webpack/docs/wiki/list-of-plugins#dllplugin) for flexible, high-performance development and scalability.

Large, mature apps tend to naturally organize themselves in this way—analogous to large, mature trees (as in actual trees :evergreen_tree:). The trunk is the router, branches are route bundles, and leaves are views composed of common/shared components/containers. Global application and UI state should be placed on or close to the trunk (or perhaps at the base of a huge branch, eg. `/app` route).

*Note: We recommend keeping your store flat, which is not strictly fractal. However, this structure provides a rock-solid foundation for creating or migrating to truly fractal apps by dropping in frameworks such as [redux-elm](https://github.com/salsita/redux-elm).*

#### Code Splitting Anatomy

We use `react-router` [route definitions](https://github.com/reactjs/react-router/blob/master/docs/API.md#plainroute) (`<route>/index.js`) to define units of logic within our application.

It's important to understand how webpack integrates with `react-router` to implement code splitting, and how everything is tied together with redux. Let's dissect the counter route definition:

```js
/*  1. ReactRouter -  Create PlainRoute definition object  */
export default (store) => ({
  path: 'counter',

  /*  2. ReactRouter -  Invoked when path match (lazy)  */
  getComponent (nextState, cb) {

    /*  3. Webpack (build) -  Create split point
        4. Webpack (runtime) -  Load async chunk with embedded jsonp client  */
    require.ensure([], (require) => {

      /*  5. Webpack (build) -  Require all bundle contents  */
      const Counter = require('./containers/CounterContainer').default
      const reducer = require('./modules/counter').default

      /*  6. Redux -  Use store and helper to add async reducer  */
      injectReducer(store, { key: 'counter', reducer })

      /*  7. ReactRouter -  Return component */
      cb(null, Counter)

    /*  8. Webpack -  Provide a name for bundle  */
    }, 'counter')
  }
})
```

 1. **ReactRouter** - We export a function that accepts the instantiated store for async reducer/middleware/etc injection and returns a `PlainRoute` object *evaluated by react-router during application bootstrap*.
 2. **ReactRouter** - The `getComponent` callback is registered but it is not invoked until the route is matched, so it is the perfect place to encapsulate and load our bundled logic at runtime.
 3. **Webpack (Build)** - Webpack uses the `require.ensure` callback to create a split point and replaces it with a call to it's own internal `jsonp` client with relevant module information.
 4. **Webpack (Runtime)** - Webpack loads your bundle over the network.
 5. **Webpack (Build)** - Webpack walks the required dependency tree and runs a chunking algorithm to merge modules into an async bundle, also known as code-splitting.
 6. **Redux** - Use `injectReducer` helper and instantiated store to inject `counter` reducer on key 'counter'
 7. **ReactRouter** - Pass resolved component back up to `Router` (using CPS callback signature)
 8. **Webpack (Build)** - Create named chunk using `require.ensure` callback


**Notes**

- Your entire route hierarchy **can and should** be loaded during application bootstrap, since code-splitting and bundle loading happens lazily in `getComponents` the route definitions should be registered in advance!
- Additional child routes can be nested in a fractal hierarchy by adding `childRoutes`
- This structure is designed to provide a flexible foundation for module bundling and dynamic loading
- Using a fractal structure is optional, smaller apps might benefit from a flat routes directory

**Usage with JSX**

We recommend using POJO (Plain Old Javascript Object) route definitions, however you can easily integrate them with JSX routes using React Router's [`createRoutes`](https://github.com/ReactTraining/react-router/blob/v3/docs/API.md#createroutesroutes) helper. Example of POJO routes using JSX:

```js
// ...
import SubRoutes from './routes/SubRoutes' // JSX Routes

export default {
  path: '/component',
  component: Component,
  children: createRoutes(SubRoutes)
}

```

- Alternatively, the JSX route definition file can `export default createRoutes(<Route />)`
- JSX can easily use POJO routes by passing them as a prop, ie `<Route children={PlainRoute} />`



#### Recommendations

Above all, you should seek to find the best solution for the problem you are trying to solve. This setup will not fit every use case, but it is extremely flexible. There is no "right" or "wrong" way to set up your project. Here are some general recommendations that we have found useful. If you would like to add something, please submit a PR.

##### Routes
* A route directory...
  - *Should* contain an `index.js` that returns route definition
  - **Optional:** assets, components, containers, redux modules, nested child routes
  - Additional child routes can be nested within `routes` directory in a fractal hierarchy

##### Store
 - Your store **should not reflect the hierarchy of your folder structure**
 - Keep your store as flat and normalized as possible. If you are dealing with deeply nested data structures, we recommend using a tool such as [normalizr](https://github.com/gaearon/normalizr).
 - Note that the `injectReducer` helper can be repurposed to suit your needs.

##### Layouts
* Stateless components that dictate major page structure
* Useful for composing `react-router` [named components](https://github.com/reactjs/react-router/blob/master/docs/API.md#components-1) into views

##### Components
* Prefer [stateless functional components](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions)
  - eg: `const HelloMessage = ({ name }) => <div>Hello {name}</div>`
* Top-level `components` and `containers` directories contain reusable components

##### Containers
* Containers **only** `connect` presentational components to actions/state
  - Rule of thumb: **no JSX in containers**!
* One or many container components can be composed in a stateless functional components
* Tip: props injected by `react-router` can be accessed using `connect`:

  ```js
    // CounterWithMusicContainer.js
    import { connect } from 'react-redux'
    import Counter from 'components/Counter'
    export const mapStateToProps = (state, ownProps) => ({
      counter: state.counter,
      music: ownProps.location.query.music // why not
    })
    export default connect(mapStateToProps)(Counter)

    // Location -> 'localhost:3000/counter?music=reggae'
    // Counter.props = { counter: 0, music: 'reggae' }
  ```

## Authors
- Jay Chung - [twitter](https://twitter.com/wuceh14678)
