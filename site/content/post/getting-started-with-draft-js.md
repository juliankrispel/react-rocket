---
title: "Getting started with draft.js"
date: 2017-06-16T14:49:00.000Z
description: Draft.js is a powerful framework for creating text based editors. It removes a lot of the complexity of working with contentEditable. This is a basic tutorial on how to get started with draft-js.
---

## Heads up

- I'm writing the code for this tutorial at the same time. If you see a git commit hash somewhere (this would look something like `#f3280j39jd12390jke1`), it's a link to a git commit related to that particular stage of the tutorial, here's [a link to the repository](https://github.com/juliankrispel/basic-draft-js-tutorial).
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
<small>[#e10afa02b8645fe2cb233ade942691168eb9927d](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/e10afa02b8645fe2cb233ade942691168eb9927d)</small>

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
<small>[#f0eb7a975077a607315de9e2fc5ab9bafd337b91](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/f0eb7a975077a607315de9e2fc5ab9bafd337b91)</small>

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

![Hello draft-js editor](/img/blog/hello-world.gif)

## Updating editorState

But if you look at your console, you'll see an error: `Uncaught TypeError: this.props.onChange is not a function`. And if you interact with the editor right now, you'll notice some weird behaviour. The first keystroke will be missed, it won't let you undo anything etc.

That's because we're missing the other required Editor prop - `onChange`. Let's add it and update our state with the onChange method, here's what we've changed:

![onChange prop for draft-js Editor](/img/blog/on-change-draft-js.png)

<small>[#7c7e299952ff6a2c91cd77b3ad3e941a6c1cc72d](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/7c7e299952ff6a2c91cd77b3ad3e941a6c1cc72d)</small>

the method that you mount on the `onChange` prop gets called whenever a change event happens inside the draft-js editor, this gets called with a new editorState instance, that contains the changes made to it, that may include, as we said above: content-changes, updates to the undo stack and other things. This onChange callback provides an opportunity to do stuff with the update, like saving to a db or localStorage.

## Rich text editing with key commands

Right, so now we have the Editor component working, let's implement some basic rich text editing with your standard key combinations, like `option + b` for bold, `option + i` for italic, `option + u` for underlined text. Draft-js comes with a utilities to make this really easy.

![Rich text editing with key commands](/img/blog/draft-js-rich-utils.png)
![Rich text editing with key commands](/img/blog/draft-js-rich-utils.png)

<small>[#16d29d7a83d5373153a99cf89139d3db44d4076a](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/16d29d7a83d5373153a99cf89139d3db44d4076a)</small>

Let's go over what we did here.
1. First we import the RichUtils library from draft-js `import { Editor, EditorState, RichUtils } from 'draft-js';`

2. The handleKeyCommand prop on the editor will let us handle key commands. We pass a method to handle the key command:
```js
  render() {
    return (
      <Editor
        editorState={this.state.editorState}
        handleKeyCommand={this.handleKeyCommand}
        onChange={this.onChange}
      />
    );
  }
```
3. Inside the handleKeyCommand method we now have an opportunity to intercept key commands and change our editorState. The argument that the handleKeyCommand callback receives is a string, like `backspace`, or `bold`. The method `RichUtils.handleKeyCommand` handles a bunch of key commands out of the box. We pass it the editorState and the command and if this returns a new editorState we pass it to our onChange method to update the state.
```js
  handleKeyCommand = (command) => {
    const newState = RichUtils.handleKeyCommand(this.state.editorState, command);

    if (newState) {
      this.onChange(newState);
      return 'handled';
    }

    return 'not-handled';
  }
```

Bear in mind that we need to return `'handled'` or `'not-handled'` in the `handleKeyCommand` method to tell the editor that the key command has been handled. Otherwise it'll fall back to native command handling.

## Rich text editing with buttons
We can initiate commands with buttons too, quite easily with the `RichUtils.toggleInlineStyle` method. Here's all I needed to change:

![Rich text editing with buttons](/img/blog/draft-js-rich-text-buttons.png)

<small>[#49a546be40fb04465ce00c56d5b5eedb275d803c](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/49a546be40fb04465ce00c56d5b5eedb275d803c)</small>

What I did was add a button that has an onClick callback. Which then calls the method `onUnderlineClick`. `RichUtils.toggleInlineStyle` needs the editorState as well as the style you want to apply to your selection, and it will return a new editorState with the changes applied. Simple right? Of course you can do this for all sorts of things. Have a look at the RichUtils docs to see what other methods it offers, quite a few! Let's do one more thing before we close of with this tutorial, let's implement code blocks, since we love code blocks don't we!

## Code blocks with RichUtils

`The RichUtils.toggleCode` method lets us make a block into a code block. We can implement it the same way we did with the Underline button. I just need to change two lines of code.

![Toggling code blocks](/img/blog/draft-js-toggle-code-block.png)

<small>[#534d124b240a58a04bd5b2a6029554bc4080b50a](https://github.com/juliankrispel/basic-draft-js-tutorial/commit/534d124b240a58a04bd5b2a6029554bc4080b50a)</small>

Same concept, we create a method that modifies editorState, this time with the `RichUtils.toggleCode` method, and update the state.

And boom, here's what we have!

![Code blocks with draft-js](/img/blog/code-block.gif)

Pretty neat huh!

I hope this helped getting you started with draft-js. Stay tuned for more and let me know if you think I've missed something!
