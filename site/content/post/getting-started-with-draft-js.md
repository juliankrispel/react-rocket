---
title: "Getting started with draft.js"
date: 2017-05-12T10:45:00.000Z
description: Getting started with draft.js
---

## Heads up

- I'm writing the code for this tutorial at the same time. If you see a git commit hash somewhere, it's a link to a git commit related to that particular stage of the tutorial.
- draft.js depends on two technologies: [react.js](http://reactjs.com/) and [immutable.js](https://facebook.github.io/immutable-js/), basic knowledge of both those libraries is recommended to follow along with this tutorial.

Right, let's begin

## Setup

To start of, let's use create-react-app to bootstrap a little app

Install create-react-app - `npm install -g create-react-app`.

And bootstrap an app - `create-react-app  my-draftjs-project`.
Add draft-js as a dependency - `yarn add draft-js`.

## The Editor Component

The center piece for draft js is the [Editor Component](https://draftjs.org/docs/api-reference-editor.html).

It's a react component and as such, state management is essentially up to you.

Let's put the editor component into our react app. I'm placing it in `src/App.js`.

Without reading any documentation, let's just render the Editor compomnent, this will throw an error which will tell us how to properly use the Editor component. Here's my `App.js`:

```js
import React, { Component } from 'react';
import { Editor } from 'draft-js';

class App extends Component {
  render() {
    return (
      <Editor />
    );
  }
}

export default App;
```
[#e10afa02b8645fe2cb233ade942691168eb9927d](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/e10afa02b8645fe2cb233ade942691168eb9927d)

The result is this: `TypeError: Cannot read property 'getCurrentContent' of undefined`.

Right. So this is draft-js telling us that it needs some inital content. If you look at the [docs for the Editor component](https://draftjs.org/docs/api-reference-editor.html#props) you can see that there are two required props - `editorState` and `onChange`. The former is basically the object that draft-js uses for state management, and onChange gives us a callback to update said state. Let's have a look at editorState.

## EditorState

To quote from the draft-js docs, EditorState is an immutable record that represents the entire state of the Draft editor, including:

- The current text content state
- The current selection state
- The fully decorated representation of the contents
- Undo/redo stacks
- The most recent type of change made to the contents

To get started, let's create an EditorState with empty content and give it to our Editor component, here's my updated App.js

```js
import React, { Component } from 'react';
import { Editor, EditorState } from 'draft-js';

class App extends Component {
  constructor() {
    super();
    this.state = {
      editorState: EditorState.createEmpty(),
    };
  }

  render() {
    return (
      <Editor
        editorState={this.state.editorState}
      />
    );
  }
}

export default App;
```
[#f0eb7a975077a607315de9e2fc5ab9bafd337b91](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/f0eb7a975077a607315de9e2fc5ab9bafd337b91)

Let's go through this.

1. First I import the EditorState class from draft-js
`import { Editor, EditorState } from 'draft-js';`

2. Then I contain the editorState within my container component, I do this within the constructor
```js
  constructor() {
    super();
    this.state = {
      editorState: EditorState.createEmpty(),
    };
  }
```
Since draft-js doesn't do state updates automatically we need to manage updates for it. This is good, we want to be in control of state updates. I mount this in the component state so that I can later update the state with the editors onChange method.
3. Then I pass the editorState to the Editor component in our App components render method:
```js
  render() {
    return (
      <Editor
        editorState={this.state.editorState}
      />
    );
  }
```

Now the Editor component will actually render on the page:

