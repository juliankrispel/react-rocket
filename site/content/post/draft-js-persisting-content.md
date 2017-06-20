---
title: "Draft-js - Saving data to the server"
date: 2017-06-20T12:15:00.000Z
description: Persisting data with draft js is not very obvious but actually fairly straightforward. In this tutorial you will learn how you would typically interact with a server.
---

__Heads up:__ There are two previous posts in this series. If you're fairly new to [draft.js](https://draftjs.org/) you should check out at least the 1st one, it describes the basics of using and developing with draft.js.

1. One on [Getting started with draft.js](/post/getting-started-with-draft-js/)
2. Another on [Getting started with draft-js-plugins](/post/getting-started-with-draft-js-plugins/)

Also, all this code is [in a github repository](https://github.com/juliankrispel/draft-js-persisting-data), if you see a commit hash somewhere (like `#d31jf249i321j8...`), it's a link to a commit at that particular point in the tutorial.

## Draft.js is nice, but how on earth do I save data to the server?
Unfortunately there doesn't seem to be that much documentation out there, nevermind tutorials. So when [Nik suggested on twitter](https://twitter.com/nikgraf/status/876449396401025024) that I write on how to save draft-js data to the server it seemed like an obvious choice - It's easy enough for a short tutorial and super useful: Unless you're just playing around with draft-js you will need to save to the server.

### Setup
To make starting a little quicker I created a little boilerplate. All you need to do is clone [this repository](https://github.com/juliankrispel/draft-js-barebones-boilerplate), cd into your project folder and run `yarn` or `npm install` to install the dependencies. It's been created with [create-react-app](https://github.com/facebookincubator/create-react-app) and has just one extra dependency: draft.js.

Once that's done you can start the app by running `yarn start` or `npm start` and you should see the draft.js editor working

![basic draft.js editor](/img/blog/draft-js-basic-editor.gif)

The component that contains your editor should now look like this:

```js
import React, { Component } from 'react';
import { EditorState, Editor } from 'draft-js';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      editorState: EditorState.createEmpty(),
    }
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
      </div>
    );
  }
}

export default App;
```

### Making draft.js data persistable
To be able to save your draft.js content to the server you'll need to first produce a data structure that is persistable and which you can send across a transfer protocol like HTTP. Although the draft.js content model is an [immutable](https://facebook.github.io/immutable-js/) data structure, you can convert it to a plain JavaScript object, convertable to `JSON`, which we all know and love.

The draft.js library comes with handy utility functions to serialize and unserialize it's immutable data structure to a plain JS object and vice versa, there are two methods for this:

- `convertFromRaw` converts raw JS object to `ContentState`.
- `convertToRaw` converts `ContentState` to raw JS Object.

So let's say we're writing a document editor and whenever change our content, we want to persist our content. Conveniently, we can just use our onChange handler for this. As we established above we need to first make our data persistable, so we need to use the `convertToRaw` utility that draft.js provides. As a first step let's just use that and log the output of `convertToRaw(contentState)`.

So here's the current state of my `App.js`.

```js
import React, { Component } from 'react';
import { EditorState, Editor, convertToRaw } from 'draft-js';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      editorState: EditorState.createEmpty(),
    }
  }

  onChange = (editorState) => {
    const contentState = editorState.getCurrentContent();
    console.log('content state', convertToRaw(contentState));
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
      </div>
    );
  }
}

export default App;
```
<small>[#effa9685bdd1f5725b4e7ba83227708e759539e5](https://github.com/juliankrispel/draft-js-persisting-data/commit/effa9685bdd1f5725b4e7ba83227708e759539e5)</small>

To use `convertToRaw` we need to import it first, so alongside Editor and EditorState we'll import `convertToRaw`
```js
import { EditorState, Editor, convertToRaw } from 'draft-js';
```

Then in our onChange handler we log the return value of `convertToRaw`. So now if you start your app you should see that your raw data is logged into the console:

![Logging raw content state](/img/blog/logging-raw-content.gif)

In our next step we should persist our data. For brevity's sake, we'll use localStorage for now. I'll create a method called `saveContent` which will save the content to localStorage. localStorage only accepts strings as items, so in order to persist the data we need to use `JSON.stringify` to convert it into a JSON string.

```js
saveContent = (content) => {
  window.localStorage.setItem('content', JSON.stringify(convertToRaw(content)));
}
```

And call it from within our `onChange` handler:

```js
onChange = (editorState) => {
  const contentState = editorState.getCurrentContent();
  this.saveContent(contentState);
  this.setState({
    editorState,
  });
}
```

Now that we're persisting our content in localStorage, let's recover it from localStorage so we can reload our page. We should do that in a callback which happens only once so an appropriate place would be the constructor. Since we save the content state as JSON, we'll need to convert it back. For that we'll need to use `JSON.parse` in combination with `convertFromRaw` which converts the plain JavaScript object to our immutable `ContentState` record.

Here's how my constructor now looks like:
```js
constructor(props) {
  super(props);
  this.state = { };

  const content = window.localStorage.getItem('content');

  if (content) {
    this.state.editorState = EditorState.createWithContent(convertFromRaw(JSON.parse(content)));
  } else {
    this.state.editorState = EditorState.createEmpty();
  }
}
```

What I'm doing here is I get the content item from local storage. If it doesn't exist I create an empty EditorState with `EditorState.createEmpty` as before, if it does exist I first use `JSON.parse` to transform the `JSON` string into a JavaScript object, then I use `convertFromRaw` to convert the JavaScript object into an immutable ContentState object and then I use `EditorState.createWithContent` with this `ContentState` object. Here's the whole flow of persisting our content state, as a recap:

```
when content changes:
  editorState -> editorState.getCurrentContent -> convertToRaw -> JSON.stringify -> localStorage.setItem

when component initializes:
  localStorage.getItem -> JSON.parse -> convertFromRaw -> EditorState.createWithContent -> editorState
```

And here's an example of the editor persisting it's content:

![Draft js editor with localStorage](draft-js-editor-with-localstorage.gif)

## LocalStorage is all good, but how do I hook this up with an actual server?
It's the same principal, but instead of using the localStorage API we use http to exchange our `JSON` data with the server. So let's do just that.

I wrote up a tiny json api with express, the code for [the server is here](https://github.com/juliankrispel/draft-js-persisting-data/blob/master/server.js) in case you're interested, but really writing a node.js server isn't what this tutorial is about. All you need to know is this, the server has one endpoint -  `/content`. It accepts `GET` and `POST` requests. The `GET` endpoint returns the data you saved (if you have saved any), the `POST` endpoint let's you save your content. The server has a couple of dependencies so it'd be best if you just [download it from here](https://github.com/juliankrispel/draft-js-persisting-data/releases/tag/with-working-server) and install the depenencies by running `yarn` or `npm install`

I've modified the `start` script so that it doesn't only start the dev server for our draft-js app, but it also starts the node server.

Now let's add some server interaction:

I'm going to use the [fetch api](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) as my ajax interface, might as well. Fetch is a fairly new standard, so if you use it in production you want to back it up with a stable polyfill, like [this one from github](https://github.com/github/fetch), it covers most use-cases that ajax libraries cater for, we used it at [rainforest actually](https://rainforestqa.com).

Alright, let's do this. First we're going to change how we fetch our data. We don't want to use localStorage anymore, since we use our server for persistance now. So I'm getting rid of my localStorage code in constructor, my data fetching will now live in [componentDidMount](https://facebook.github.io/react/docs/react-component.html#componentdidmount), the callback that fires when a react component mounts, for asynchronous server interaction, componentDidMount is a fairly conventional place.

This is now my constructor, pretty bare:

```js
constructor(props) {
  super(props);
  this.state = { };
}
```

And this is now my `componentDidMount` callback
```js
componentDidMount() {
  fetch('/content').then(val => val.json())
  .then(rawContent => {
    if (rawContent) {
      this.setState({ editorState: EditorState.createWithContent(convertFromRaw(rawContent)) })
    } else {
      this.setState({ editorState: EditorState.createEmpty() });
    }
  });
}
```

Right let's go through the above step by step:

1. We fetch from `/content`.
2. We use the `.json()` method to return a promise that produces json - the json method will automatically parse our response, so we can skip our `JSON.parse` step.
3. Then, as before with our localStorage implementation, we check if the content exists. If it does we create a ContentState with content, otherwise we create an empty state.

In case you didn't notice, `this.state.editorState` will be undefined until we successfully performed our api request. So I'm also implementing a loading message until the content is loaded. Rendering empty content state at this point would be bad UX because the content would be reset once it is loaded. So here's how my render method now looks:

```js
render() {
  if (!this.state.editorState) {
    return (
      <h3 className="loading">Loading...</h3>
    );
  }
  return (
    <div>
      <Editor
        editorState={this.state.editorState}
        onChange={this.onChange}
      />
    </div>
  );
}
```

As you can see, it renders a loading element if `this.state.editorState` is empty, otherwise it renders the Editor.

And that's basically it, if you make those changes and run `yarn start` you'll see it working:

![Draft js with server](/img/blog/draft-js-with-server.gif)

YIPPIE!

One more thing. What's really bugging me right now is that every time there's a keystroke a http request will be fired, that's bad practice, it's wasteful, ideally we want our autosaving to occur once every second or so. We can debounce our persistance method with a [handy lodash utility](https://lodash.com/docs/4.17.4#debounce). If you haven't used lodash yet, give it a try it's full of handy utilities and highly optimized. I use it in almost every single project, because there's usually a usecase. Right, so I install lodash and import it.

So I import my debounce method 

```js
import debounce from 'lodash/debounce';
```

And simply wrap my saveContent method with it:

```js
saveContent = debounce((content) => {
  ...
}, 1000);
```

That's it, now instead of saveContent firing all the time, it waits for one second of inactivity before saving, much better!

Right, our server interaction is wrapped up, this is how my App.js looks now:

```js
import React, { Component } from 'react';
import { EditorState, Editor, convertToRaw, convertFromRaw } from 'draft-js';
import debounce from 'lodash/debounce';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = { };
  }

  saveContent = debounce((content) => {
    fetch('/content', {
      method: 'POST',
      body: JSON.stringify({
        content: convertToRaw(content),
      }),
      headers: new Headers({
        'Content-Type': 'application/json'
      })
    })
  }, 1000);

  onChange = (editorState) => {
    const contentState = editorState.getCurrentContent();
    this.saveContent(contentState);
    this.setState({
      editorState,
    });
  }

  componentDidMount() {
    fetch('/content').then(val => val.json())
    .then(rawContent => {
      if (rawContent) {
        this.setState({ editorState: EditorState.createWithContent(convertFromRaw(rawContent)) })
      } else {
        this.setState({ editorState: EditorState.createEmpty() });
      }
    });
  }

  render() {
    if (!this.state.editorState) {
      return (
        <h3 className="loading">Loading...</h3>
      );
    }
    return (
      <div>
        <Editor
          editorState={this.state.editorState}
          onChange={this.onChange}
        />
      </div>
    );
  }
}

export default App;
```

Thanks for reading! I hope this helped your understanding of draft.js. I'm going to write more on draft.js so if you have an idea for a particular blog post please [reach out on twitter](https://twitter.com/juliandoesstuff). Also, join the [reactiflux](https://www.reactiflux.com) and [draft-js](https://draftjs.herokuapp.com/) communitys on slack and discord, they are full of friendly awesome helpful people!
