---
title: "Draft-js and Redux - the perfect couple"
date: 2017-06-20T15:32:00.000Z
description: In this tutorial you'll learn how to use Draft.js together with Redux, our favorite state management system for react apps.
series: 'learning-draft-js'
series_weight: 4
---

__Heads up__:

- You can find the [complete project in this github repository](https://github.com/juliankrispel/draft-js-and-redux)
- This is the 4th in a series of blog posts on draft.js. Check out the above blog posts to get a primer on draft-js.

We like draft.js (At least I hope you do since you're here reading this.) and we possibly also like redux, the go to state management system for react. The two of them are actually a really good couple. So let's marry them and use them together for all the things!

### Setup
I created a simple little boilerplate for draft-js projects. All you need to do is clone [this repository](https://github.com/juliankrispel/draft-js-barebones-boilerplate), cd into your project folder and run `yarn` or `npm install` to install the dependencies. It's been created with [create-react-app](https://github.com/facebookincubator/create-react-app) and has just one extra dependency: draft.js.

However, you also want to add redux and react-redux for this tutorial, so go please ahead and do that:

```js
yarn add redux react-redux
```

### Setting up a store
In case you know redux well, you've probably done this before - feel free to skip it. In case you haven't, here's how:

First I'm going to import all the tools I need to use redux in my app:

- The `Provider` component and connect method from [react-redux](https://github.com/reactjs/react-redux).
- The `createStore` method from [redux](http://redux.js.org/).

```js
import { Provider, connect } from 'react-redux';
import { createStore } from 'redux';
```

Then I'll create the default state for my store, this is how it'll look like:

```js
const defaultState = {
  editorState: EditorState.createEmpty(),
};
```

The default state is the first value that my redux reducer will produce and I'll just use it as the default argument in my reducer. A redux reducer is simply a function, this is my reducer:

```js
const reducer = (state = defaultState, { editorState, type }) => {
  if (type === 'UPDATE_EDITOR_STATE') {
    return {
      ...state,
      editorState,
    };
  }
  return state;
};
```

The redux reducer takes two arguments, state and action. So everytime we dispatch an action to the reducer the reducer will be called. I only have one action for my reducer so this will be fairly straight forward, I call the type of my action `'UPDATE_EDITOR_STATE'`. If that action is passed into the reducer I extend the previous state with the editorState that is passed in by my action (Contained in the `payload`).

Now that we have a reducer, let's use that to create a store:

```js
const store = createStore(reducer);
```

And in turn let's use that store and hand it to our `Provider` component, which we wrap our app in:

```js
class App extends Component {
  render() {
    return (
      <Provider store={store}>
        ...
      </Provider>
    );
  }
}
```

Whatever component is mounted inside our Provider component will have access to our store via the connect method. We want to mount our Editor inside this provider, and we want our editor to have access to the editorState and an action to update the editor state. Since we're now delegating our state management to redux, we don't need to handle our state inside a component anymore and so we can just wrap our Editor in a stateless component. This is a best practice, generally you want to avoid state management in component's as much as possible, it's not always possible I'm afraid. So here's our Editor Component:

```js
const AppEditor = ({ editorState, onSaveEditorState }) => (
  <Editor
    editorState={editorState}
    onChange={onSaveEditorState}
  />
);
```

Now we want to use [connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) to connect our AppEditor with the editorState and an action that updates the editorState. We'll pass in the first two arguments that the `connect` method takes:
- `mapStateToProps` - this maps our store state and mounts the results as props on our component.
- `mapDispatchToProps` - this lets us mount actions as props onto our component.

`mapStateToProps` is pretty straight forward, all we want to do is get the `editorState` from our store and give it to our component, so it'll be just that

```js
const mapStateToProps = ({ editorState }) => ({ editorState });
```
<small>Btw, I'm using an es6 feature here called destructuring, if you haven't seen this, [please have a look](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), it saves a bit of typing.</small>

Next, we need to define `mapDispatchToProps`, that's a little bit more code:

```js
const mapDispatchToProps = (dispatch) => ({
  onSaveEditorState: (editorState) => {
    dispatch({
      type: 'UPDATE_EDITOR_STATE',
      payload: editorState,
    })
  }
});
```

`mapDispatchToProps` receives the dispatch method as it's first argument. What we want to do in this method is return any methods that we want to mount as props on our editor. Here we're defining a method called `onSaveEditorState` which we will call whenever the editorState changes. This way we can dispatch an action to our reducer. The argument for the dispatch method is the action and the action is a plain object that consists of a `type` and a `payload`. The payload is the `editorState` because that is the state we want redux to manage for us.

Right, now let's use those two methods to mount our desired props onto our editor. The `connect` method returns a higher order component - It's a component that wraps another component, that way you can compose behaviours. Our new editor we'll call `ConnectedEditor`.
<small>If you want to learn more about higher order components or hoc's [read this](https://facebook.github.io/react/docs/higher-order-components.html) [and this](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e).

```js
const ConnectedAppEditor = connect(
  mapStateToProps,
  mapDispatchToProps,
)(AppEditor);
```

So we use the `connect` method, pass `mapStateToProps` and `mapDispatchToProps` into it, which returns a higher order component. Then we use that to wrap our AppEditor and voila, we have our ConnectedAppEditor. And then we mount that inside our Provider and we're done:

```js
class App extends Component {
  render() {
    return (
      <Provider store={store}>
        <ConnectedEditor/>
      </Provider>
    );
  }
}
```

And here's our result, I'm logging our redux actions inside the reducer, for illustration:

![Draft js with redux](/img/blog/draft-js-with-redux.gif)

Now here's the App.js in it's full glory, complete with reducer, actions and components:

```js
import React, { Component } from 'react';
import { EditorState, Editor } from 'draft-js';
import { Provider, connect } from 'react-redux';
import { createStore } from 'redux';

const defaultState = {
  editorState: EditorState.createEmpty(),
};

const reducer = (state = defaultState, { payload, type }) => {
  if (type === 'UPDATE_EDITOR_STATE') {
    console.log('redux action: ', type, payload.getCurrentContent().getPlainText());
    return {
      ...state,
      editorState: payload,
    };
  }
  return state;
};

const store = createStore(reducer);

const AppEditor = ({ editorState, onSaveEditorState }) => (
  <Editor
    editorState={editorState}
    onChange={onSaveEditorState}
  />
);

const mapStateToProps = ({ editorState }) => ({ editorState });

const mapDispatchToProps = (dispatch) => ({
  onSaveEditorState: (editorState) => {
    dispatch({
      type: 'UPDATE_EDITOR_STATE',
      payload: editorState,
    })
  }
});

const ConnectedEditor = connect(
  mapStateToProps,
  mapDispatchToProps,
)(AppEditor);

class App extends Component {
  render() {
    return (
      <Provider store={store}>
        <ConnectedEditor/>
      </Provider>
    );
  }
}

export default App;
```

You can find the [complete project in this github repository](https://github.com/juliankrispel/draft-js-and-redux)

### A note on best practices
To keep things simple, I didn't completely adhere to best practices in this tutorial. Ideally you should:
1. Have separate files for components.
2. Separate your actions and reducers from your components.
3. Have a fixed action shape instead of defining it in your action creator.

To learn more about redux and best practices, I strongly recommend watching [Dan Abramaov's free egghead course on redux](https://egghead.io/courses/getting-started-with-redux), he's the creator of redux and an awesome dude throughout!
