---
title: "Draft-js building search and replace functionality"
date: 2017-06-21T12:32:00.000Z
description: In this tutorial you'll learn how to implement a search and replace functionality. You'll use Modifiers to update state and decorators to highlight text.
series: 'learning-draft-js'
series_weight: 5
---

In this episode of learning-draft-js we'll build something usefu - The functionality to search and replace strings. The working code for this tutorial is contained [within this repo](https://github.com/juliankrispel/draft-js-building-search-and-replace).

Let's get right into it.

## Setup
To start we'll clone the [draft js boilerplate repository](https://github.com/juliankrispel/draft-js-barebones-boilerplate), let's cd into our app folder, run `yarn` to install all dependencies and get going!


### Our brief - A way to search for text and replace it.
We're building a basic IDE. The first task we're supposed to tackle is to build an interface for searching an replacing text.

Before we deal with the draft.js implementation, we'll simply create a search and replace interface. I'm just defining two input fields, both update the state so we can share the search text and the replace text. Here's how my `App.js` now looks like now:

```js
import React, { Component } from 'react';
import { EditorState, Editor } from 'draft-js';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      search: '',
      replace: '',
      editorState: EditorState.createEmpty(),
    }
  }

  onChangeSearch = (e) => {
    this.setState({
      search: e.target.value,
    });
  }

  onChangeReplace = (e) => {
    this.setState({
      replace: e.target.value,
    });
  }

  onReplace = () => {
    console.log(`replacing "${this.state.search}" with "${this.state.replace}"`);
  }

  onChange = (editorState) => {
    this.setState({
      editorState,
    });
  }

  render() {
    return (
      <div>
        <Editor
          editorState={this.state.editorState}
          onChange={this.onChange}
        />
        <div className="search-and-replace">
          <input
            value={this.state.search}
            onChange={this.onChangeSearch}
            placeholder="Search..."
          />
          <input
            value={this.state.replace}
            onChange={this.onChangeReplace}
            placeholder="Replace..."
          />
          <button onClick={this.onReplace}>
            Replace
          </button>
        </div>
      </div>
    );
  }
}

export default App;
```

All we've done is added two new fields to our state shape.

- `onChangeSearch` will update `this.state.search` value
- `onChangeReplace` will update our `this.state.replace` value
- and `onReplace` will perform our search and replace action.

Also, I added some CSS to make it look like something:

![Draft js editor with search and replace interface](img/blog/draft-js-search-and-replace-interface.png)

Since we're not doing anything with draft.js yet, this is only half implemented, so let's get that going:

### A little bit about ContentBlocks & Decorators
To use decorators it's helpful to understand what a `ContentBlock` is. It's basically the equivalent of a paragraph. When the draft js editor renders, it loops over all of the documents `ContentBlock`'s and applies it's decorators.

A decorator is a plain object that has two keys, a __strategy__ and a __component__:

#### Decorator Strategy
The strategy is a function that receives two arguments, the current `contentBlock` and a `callback`. The callback is what tells the decorator which parts of the text to decorate, you need to pass it a start and an end position. For example, let's say we have a block with the content `'Hello World'` and we decide we want to decorate the word `'World'`, the start position would be `6` and the end position would be `10`, we pass those indexes as parameters to our callback - `callback(6, 10)`. Of course we wouldn't hard-code that, so typically we'd use a regex to find start and end position of a string.

#### Decorator Component
When the decorator strategy invokes it's callback, it tells the editor to wrap the text inside the decorator component. The decorator component is a normal react component, and the contents for a decorator component are passed in through the `children` prop.

There's one caveat to our solution - if our editorState doesn't change, the editor won't rerender, so what we'll do in our case is remount the decorator everytime we change the search text, that way the editor will be forced to rerender, decorating our searched text.

Now let's implement it, First we import `CompositeDecorator`.

```js
import { EditorState, Editor, CompositeDecorator } from 'draft-js';
```

Then we define a method to generate our decorator:

```js
const generateDecorator = (highlightTerm) => {
  const regex = new RegExp(highlightTerm, 'g');
  return new CompositeDecorator([{
    strategy: (contentBlock, callback) => {
      if (highlightTerm !== '') {
        findWithRegex(regex, contentBlock, callback);
      }
    },
    component: SearchHighlight,
  }])
};
```

Here we create a new regex containing our highlightTerm, this regex we'll use to find the text ranges that we want to decorate. The strategy will only execute if our highlightTerm isn't empty, seems sensible! Within our strategy function we use `findWithRegex`. `findWithRegex` is a function that we implement as well, it takes a regex, a the contentBlock and our callback and will invoke the callback with the ranges for all the matches found, here's the implementation for `findWithRegex`:

```js
const findWithRegex = (regex, contentBlock, callback) => {
  const text = contentBlock.getText();
  let matchArr, start, end;
  while ((matchArr = regex.exec(text)) !== null) {
    start = matchArr.index;
    end = start + matchArr[0].length;
    callback(start, end);
  }
};
```

What we do inside `findWithRegex` is we first get the text from the block via `contentBlock.getText()`, then we use `regex.exec` to loop over all matches and invoke the callback with the start and end value for every match.

Our decorator component is the other missing piece, it's a simple react component wrapping our content inside a span with a class that is styled to highlight the contents.

```js
const SearchHighlight = (props) => (
  <span className="search-and-replace-highlight">{props.children}</span>
);
```

Finally we need to hand our decorator to our editorState, and we do so whenever we update our search term, inside `onChangeSearch`:

```js
onChangeSearch = (e) => {
  const search = e.target.value;
  this.setState({
    search,
    editorState: EditorState.set(this.state.editorState, { decorator: generateDecorator(search) }),
  });
}
```

Now alongside `this.state.search` we also set `editorState`. Since `EditorState` is an immutable record we use the `EditorState.set` method to return a new state that includes our decorator. This is where we use `generateDecorator` to create a new decorator with the updated search term.

Now the decorator part of this exercise is complete and it'll highlight the search terms.

![Draft js editor with search term decorator](/img/blog/draft-js-search-decorators.gif)

### Modifiers and SelectionState and ContentState
Before we head into implementing our replacement feature, let's dive into some of the draft.js architecture. It's a bit much, but will hopefully make more sense once you implement it and play around with draft.js more.

`ContentState` and `EditorState` are both immutable objects so in theory we could use the immutable api to update state, but that's not recommended - in fact you should avoid doing so at all cost since manually managing the editor's state is complex and you're highly likely to run into issues. Instead, you should use a `Modifier` whenever you want to update the content of the editor. There's a list of operations outlined in the [docs on Modifiers](https://draftjs.org/docs/api-reference-modifier.html#content), they should be all you need to update the editors content state, whether it's replacing text, or applying entities, moving text or whatever.

Modifiers always need a `SelectionState` object to perform updates on content state. When I learned to use draft.js that wasn't very intuitive, because the name `SelectionState` implies that something is actively selected in the editor and that's only partly true. It is of course also used for the editor's current selection, but moreover you can think of the `SelectionState` as a way to define a text range. It contains the anchor and the focus position of a range (anchor and focus are equivalent to start and end). Both anchor and focus positions contain a block key and an offset, the block key is a way to identify the block, the offset marks the text position inside said block.

So let's say we have a block with key `'123'` and the content `'Goodbye'`, if we wanted to create a selection state that selects the text fragment `'bye'`, our selection state would have to have contain the following info:
```js
{
  anchorKey: '123',
  anchorOffset: 4,
  focusOffset: '123',
  focusKey: 7,
}
```

In our case the anchorKey and focusKey are the same because we'll operate on individual blocks, but that's not required, you can operate on text ranges spanning multiple blocks.

### Implementing replace
Unfortantely, draft-js doesn't give us a straightforward way to identify decorated ranges, so we'll need to reimplement some of that logic that decorators implement, there are other solutions, but this is by far the simplest:

In technical terms, these are the steps we need to follow to search text fragments and replace them with new text:

1. Retrieve our current `ContentState` object from `EditorState` object.
2. Retrieve and loop over our content blocks (just like our decorator).
3. Find all occurences of our search term inside those blocks (we can reuse our findWithRegex function for this)
4. Create a `SelectionState` for every match we find inside our blocks - remember we need SelectionState to use our `replaceText` modifier.
5. Loop over our selected ranges and modify our `ContentState` with those ranges.
6. Update our `EditorState` with our modified `ContentState`.

For this we first import the parts we'll need from the draft js lib, namely `SelectionState` and `Modifier`.

```js
import { EditorState, Editor, CompositeDecorator, SelectionState, Modifier } from 'draft-js';
```

For the replacement functionality we only need to touch one component method: `onReplace`, which gets called when we press the __replace__ button. Here it is, we'll go through it step by step:

```js
onReplace = () => {
  const regex = new RegExp(this.state.search, 'g');
  const { editorState } = this.state;
  const selectionsToReplace = [];
  const blockMap = editorState.getCurrentContent().getBlockMap();

  blockMap.forEach((contentBlock) => (
    findWithRegex(regex, contentBlock, (start, end) => {
      const blockKey = contentBlock.getKey();
      const blockSelection = SelectionState
        .createEmpty(blockKey)
        .merge({
          anchorOffset: start,
          focusOffset: end,
        });

      selectionsToReplace.push(blockSelection)
    })
  ));
  
  let contentState = editorState.getCurrentContent();

  selectionsToReplace.forEach(selectionState => {
    contentState = Modifier.replaceText(
      contentState,
      selectionState,
      this.state.replace,
    )
  });

  this.setState({
    editorState: EditorState.push(
      editorState,
      contentState,
    )
  })
}
```

First, we grab everything we need for our transformation:

```js
const { editorState, search, replace } = this.state;
const regex = new RegExp(search, 'g');
const selectionsToReplace = [];
let contentState = editorState.getCurrentContent();
const blockMap = contentState.getBlockMap();
```

1. I'm destructuring `this.state` for convenience
2. Then I create a new regular expression with our search term.
3. I define a variable called `selectionsToReplace`, this will contain all the text ranges that we want to replace.
4. Then I get the current contentState from our `editorState` object, and the list of blocks that we want to loop over to find our text ranges. I use a `let` declaration for our contentState because I'm going to modify contentState for every text replacement, for that I'll just reassign the variable.

Now let's loop over our blocks:

```js
blockMap.forEach((contentBlock) => (
  findWithRegex(regex, contentBlock, (start, end) => {
    const blockKey = contentBlock.getKey();
    const blockSelection = SelectionState
      .createEmpty(blockKey)
      .merge({
        anchorOffset: start,
        focusOffset: end,
      });

    selectionsToReplace.push(blockSelection)
  })
));
```

As you can see I'm re-using our `findWithRegex` function, which takes the regex we just created, and the contentBlocks that we're looping over. Since our callback gets called with the `start` and `end` position for our matches we can use that callback to create text ranges and add them to the `selectionsToReplace` array. As you can see I'm using `SelectionState.createEmpty` to create an empty selection contained inside the block with `blockKey`. `SelectionState` is also an immutable object so we can use `.merge` to modify the selection and pass in our start and end positions. Last but not least we push the created selection onto our array.

Now we'll use modifiers to perform the actual content update:

```js
selectionsToReplace.forEach(selectionState => {
  contentState = Modifier.replaceText(
    contentState,
    selectionState,
    replace,
  )
});
```

Since a modifier can only use one selection state object at a time, we're looping over our selection states and use them one by one. As you can see we're re-using the `contentState` variable for this. The first two arguments for any modifier are always `contentState` and `selectionState` and __in all cases Modifiers return a new `ContentState`__ object. The third argument in this case is the text we want to replace our text range with.

Last but not least we'll use `EditorState.push` to update our editorstate with our new content state:

```js
this.setState({
  editorState: EditorState.push(
    editorState,
    contentState,
  )
})
```

`EditorState.push` updates the editorState and does a couple of other things under the hood, most significantly it adds an item to the undo stack. If you press `command + z` you'll see that your replacement will be undone, pretty neat!

And there you go, now you have some lovely search and replace functionality:

![Draft js editor with search and replace functionality](/img/blog/draft-js-search-and-replace-functionality.gif)

This topic has been suggested by Jonathan Prada on the [draft js slack](https://draftjs.herokuapp.com/), thanks dude! If you have a suggestion for a topic I should cover, please hit me up on [twitter](https://twitter.com/juliandoesstuff)

