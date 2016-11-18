# Reintroduction to Components

## Overview
Let's set the stage for a second: you're on Netflix and would like to re-watch your favorite episode of The Office. You click 'Search' and begin to type in "the o". As you type each character Netflix filters down the shows until all you see is Michael Scott looking pensive. (insert picture). We know how to accomplish this same feat. with vanilla React - we would set the state of our parent component, add a function that checked for user input, then pass down the callback function as props into a child component. In redux our interaction with state inside components is a bit different...

## Objectives
1. Explain how components subscribe to store
2. Describe how a user's interactions with a component trigger updates to data
3. Describe how data gets back to components

### State flow

Now that we know about where state is being held in our Redux application; how will our components be able to access and manipulate state? Our components cannot know about the store unless they are explicitly told about it. In React a component could only know about data from a parent component if it was passed in as a prop. Let's try doing exactly that: We'll build out a band tracking App. We'll first instantiate our store in `index.js` then pass it down to the first rendered component `App.js`:

*index.js*

```js
  import React from 'react';
  import ReactDOM from 'react-dom';

  import App from './components/app';

  ReactDOM.render(<App />, document.getElementById('container'))
```

By now the above code should look pretty familiar. This is how we start the majority of React apps. We are importing the react and react-dom libraries as well as a top level component. Last, we tell our react app which component should hold the rest of our app and where in our `index.html` to render it to. Now, let's bring in our store:

```js
  import React from 'react';
  import ReactDOM from 'react-dom';

  import { createStore } from './store';

  import App from './components/app';

  const bandReducer = (state=[], action) => {
      return state;
  }

  const store = createStore(bandReducer);

  ReactDOM.render(<App store={store}/>, document.getElementById('container'))
```

Why do we want to send down our `state` to components instead of having our components maintain their own `state`? By keeping the list of bands in one place, we can be sure that it always stays in sync accross many different components. This way we won't have to ever worry about grabbing the right data for the App - it will all be stored in one central location.

To understand what our `createStore` function is doing let's take a quick look at our store:

*store.js*

```js
  export const createStore = (reducer) => {
    let state;
    let listeners = [];
    const getState = () => state;

    const dispatch = (action) => {
      state = reducer(state, action);
      listeners.forEach(listener => listener())
    };

    const subscribe = (listener) => {
      listeners.push(listener);
    };

    dispatch({});
    return {
      getState: getState,
      dispatch: dispatch,
      subscribe: subscribe
    };
  }
```

Let's review what `getState`, `dispatch`, and `subscribe` should do:

`getState` is a function that, when called, returns the state object of our application.

`dispatch` is a function that only get's called when an action occurs. It resets our application's `state` object then iterates through our listener array calling each function inside.

`subscribe` is a function that is called when we want to add a new function to our listener array.

Why do we want to add a function to our `listener`'s array? We typically add these functions so that when `dispatch` is called and it updates our state we can re-render our components. As an example let's refer back to our Netflix scenario. For this example Netflix has only two components: 1. SearchBar - which has an input field and updates `store.state` everytime a letter is entered to the input. 2. VideoSearchList - which can access the user input entered by calling on `store.getState()` and filters through the Apps database of shows based on the user input to eventually display. Each letter that is typed into the SearchBar's input field fires an action that calls `store.dispatch(action)` to update the `state` object of the Netflix application. If the VideoSearchList component (where the video thumbnails are displayed) could not re-render we would never see an updated list of Netflix shows. VideoSearchList would only be able to show the initial list of videos stored in `state` when the App first loads. This is why Netflix needs their child component of searchbar to re-render every time `state` is changed.

Ok, so now we know why our components should re-render when there's a `state` change, but how do we tell our components to do so? This is where our `listeners` come into play. Note: This is for demonstration purposes only. We, normally, would not add our own `listeners`. This array of functions should hold the functions responsible for re-rendering our components. Knowing this let's think about how our components are rendered in the first place - with the `render()` method!

Let's take a look at how a stock class based component uses the `render()` method:

```js
import React from 'react';

export default class App extends React.Component {
	render(){
		return(
			<h1> I MIGHT LOVE OFFICE APPLIANCES! </h1>
		)
	}
}

```

Now, our oddly enthusiastic `h1` tag will display whenever the `render()` method is called. In React our component's `render()` method will be called anytime our component is mounted (called). That's the nature of the `React.Component` library. The React library looks for a `render()` method in our component and runs it  when the component mounts. As a reminder for how a component would mount / be called let's take a look at our top level `index.js` file again:

```js
  import React from 'react';
  import ReactDOM from 'react-dom';

  import todosReducer from './reducers/todos_reducer';
  import { createStore } from './store';

  import App from './components/app';

  const store = createStore(todosReducer);

  ReactDOM.render(<App store={store}/>, document.getElementById('container'))
```

We can see that our component `App`, which is imported from the components folder, is being mounted / called in our `ReactDOM.render(<App store={store}/>, document.getElementById('container'))` as `<App />`. We now know that when our `App` component is being mounted it will call on that components' `render()` function - ultimately displaying our love for office appliances to the page. But, we have one problem... the mounting of our `App` component will only happen once (when the App first loads). What if we wanted this component to re-mount and re-render every time the state is changed? We know that our state changes anytime `store.dispatch(action)` is called. We, also, know that our `dispatch` action will iterate through and call on each function in listeners array, which are each meant to re-render a component. To get our `App` component to re-mount / re-render we know that we'll have to create a function for our listeners array that calls `ReactDOM.render(<App store={store}/>, document.getElementById('container'))`. If we remember our `subscribe` method's sole purpose is to push new functions into the `listeners` array. Let's try wrapping our mounting / calling of `App` in a method and pass that method into our listeners array through `subscribe`:

```js
  import React from 'react';
  import ReactDOM from 'react-dom';

  import todosReducer from './reducers/todos_reducer';
  import { createStore } from './store';

  import App from './components/app';

  const store = createStore(todosReducer);

  const renderApp = () => {
    ReactDOM.render(<App store={store}/>, document.getElementById('container'))
  }
  
  store.subscribe(render);
```

Brilliant! Now, every time `store.dispatch(action)` is called our `renderApp()` method will trigger, which will re-mount / re-render our `App` component. There is just one small issue with the above code. Though we're able to save our `renderApp()` function to the `listeners` array we are not yet calling on `renderApp()`. We will have to add one more line of code to accomplish this.

```js
  ...
  const renderApp = () => {
    ReactDOM.render(<App store={store}/>, document.getElementById('container'))
  }

  store.subscribe(render);
  renderApp();
```

Now, all we have to do is call on `renderApp` when the application is loaded, which will call and mount our `App` component!

Hooray! We now know that our components will re-render whenever `store.dispatch(action)` is called. In the next section we will take a look at how user intereaction will need to call `store.dispatch(action)` to update the `state`.
