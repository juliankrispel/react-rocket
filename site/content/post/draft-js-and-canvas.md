---
title: "Draft js interactive content"
date: 2019-04-05T16:32:00.000Z
description: Let's experiment with canvas and draft js and make a note-taking/drawing app
series: 'learning-draft-js'
series_weight: 7
---

Say we want to add drawings into our rich text document, right there and then, with our mouse. Yep, that's totally possible, and here's how:

Our canvas will be a [custom block](https://draftjs.org/docs/advanced-topics-block-components). It needs to be of type `'atomic'` since it isn't text-based.

First we need a button or something that triggers the insertion of the canvas.
When clicking said button we trigger a callback that inserts a custom block into our editor state object, let's call this method `insertCanvas`. Since we'll be inserting a block that isn't text, we'll need the [`insertAtomicBlock` method.](https://draftjs.org/docs/api-reference-atomic-block-utils#insertatomicblock). Don't worry, this'll all make sense soon (hopefully)

I'm assuming here that we're in a component where `editorState` is bound to this.state, and `insertCanvas` is a class method.

```
insertCanvas = () => {
  const { editorState } = this.state;
  let content = editorState.getCurrentContent();

  // Create IMMUTABLE entity of type CANVAS, content is a string that contains the data for our drawing, for starters this is empty
  content = content.createEntity(
    'CANVAS',
    'IMMUTABLE',
    { content: '' }
  )

  const entityKey = content.getLastCreatedEntityKey();

  this.setState({
    editorState: AtomicBlockUtils.insertAtomicBlock(
      editorState,
      entityKey,
      ' ',
    ),
  });
}
```

So in this method we need to add an atomic block, with an entity that describes our custom block (I'm calling the entity type `'CANVAS` but you can call it anything).

In our component we'll have a button that calls `insertCanvas` when clicked like so `<button onClick={this.insertCanvas} />`, the only thing that's left to do now is to render the thing, we do that by using the `blockRendererFn` prop like this:

```
<Editor
...
blockRendererFn={(block) => {
    const { editorState } = this.state;
    const content = editorState.getCurrentContent();

    if (block.getType() === 'atomic') {
      const entityKey = block.getEntityAt(0);
      const entity = content.getEntity(entityKey);

      if (entity != null && entity.getType() === 'CANVAS') {
        return {
          component: () => <Canvas />,
        }
      }
    }
  }
}
/>
```

We first check if the block type is atomic and then if the entity type is `'CANVAS'`. and then we render the canvas component, easy peasy, you could use a component [like this one](https://www.npmjs.com/package/react-canvas-draw) or write your own.

Finally, you might want to also save the drawings you've made, I've added that extra little bit of code and [made it available here](https://github.com/juliankrispel/draft-js-with-canvas-block), you can also try this in [a sandbox here](https://codesandbox.io/s/github/juliankrispel/draft-js-with-canvas-block)
