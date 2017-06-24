---
title: "Best practices when working with draft js"
date: 2017-06-25T15:04:10.000Z
description: This is a collection of tips and opinions on using draft-js. Given the framework's youth and lack of guidance on the internet, I feel like sharing some of my experience in the hope it'll be useful to others starting out.
---

__Disclaimer:__ It's hard to exclude my personal opinion from this blog post, but I'll try as best as I can. Please give me feedback, whether you think I'm wrong or you agree with me, or if you want to help submitting a pull request containing some known best practices. Also, please join the [draft.js slack](https://draftjs.herokuapp.com/) or the [reactiflux](https://www.reactiflux.com) __draft-js__ channel, there's usually some great, friendly people just like you, who want to figure stuff out and improve things!

Doing work for clients on draft.js apps in production and also talking to people on community chat I've seen myself and other people make the same mistakes. So I figured it's time to start writing down what I have learnt so far, with the hope of helping others.

### 1. React and redux are as slow as their competing frameworks, unless you use it right.
In conversations with community members I've heard reports that redux slows their app down when used in combination with draft.js. This really surprised me because it hadn't happened to me so I dug in. Here's a common scenario:

- There's a react component containing the draft.js Editor, this component is connected with the redux store, which stores the editor's state.
- The state gets updated every time the editor fires an `onChange` event.
- The editor state makes it's round trip, but there appears to be a performance penalty when doing so, there's a considerable delay for every key press.

In every case the problem wasn't related to using draft.js or redux - The most common performance problem that react apps suffer is this: Rendering too much at the same time.

When using react with redux, the `connect` method implements shouldComponentUpdate for you, so generally you don't have to worry about optimizing a components state update, that is as long as you only give it the state it needs. The `connect` higher order component will compare the props object it's given with strict equality. So if only one of the props has changed, the component and all it's children will re-render. If you're subscribing your component to data it doesn't need, it's highly likely that unintentional renders will occur and that's what will affect your apps performance.

Check out this excellent article on [building performant react apps](https://medium.com/dailyjs/react-is-slow-react-is-fast-optimizing-react-apps-in-practice-394176a11fba) by François Zaninotto to learn more about optimizing your app, it's a must read for every react developer!

### 2. Only use convertToRaw and convertFromRaw when absolutely necessary
`convertToRaw` and `convertFromRaw` are used for converting draft.js state into serializable objects and vice versa, making your `contentState` persistable. These are expensive operations and should only be used when absolutely necessary. It's absolutely fine to store your editorState in your redux store, you don't have to serialize it (I'm saying this because I've seen people implement it this way).

Typically, you'd use `convertToRaw` to dispatch your state to the server, and `convertToRaw` once you fetched it.

### 3. Always use the `children` prop type to render content.
For [custom components](https://draftjs.org/docs/advanced-topics-block-components.html) and [decorator components](https://draftjs.org/docs/advanced-topics-decorators.html), you should always use the `children` prop to render the content of an entity or a block, if you don't draft-js will misbehave - editor interaction will break if you don't

### 4. Just like any react component, you can optimize your decorators and custom block components with `shouldComponentUpdate`.
A feature-rich draft.js editor with lots of decorators and custom block components can easily cause performance issues, especially for large documents. Draft.js doesn't do anything special to prevent unnecessary rendering of these components, you can however optimize by using the [`shouldComponentUpdate` method](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate), which puts you in control of when your components re-render.

Here's an example of a `shouldComponentUpdate` implemented for a custom block which is expensive to render

```js
shouldComponentUpdate(nextProps) {
  return this.props.children.props.block.getText() !== nextProps.children.props.block.getText()
}
```

This simply compares the current text content with the last text content. Early optimization however can be a timesink. If your app doesn't have performance problems, it's probably too early to look into optimization!

### 5. There's no good way to implement tables right now.
Draft.js's content model is flat. That's a restriction that comes with it's pros and cons. Nested structures are harder to optimize and reason about, so this makes maintaining draft.js easier. However, this restriction means that implementing a table layout is hard to do without nesting editors. Nested Editors, although doable and probably the best wa to implement table functionality in draft.js roight now, comes with a bunch of extra complexity, try to stay away from this if you can.

There is an ongoing effort to implement a tree [structure in draft-js](https://github.com/facebook/draft-js/issues/143) but we have no idea as to when this feature will actually land.

### 6. Be careful with your architecture
Developing an architecture for a draft.js application is hard. There's not much advice out there and the draft.js website unfortunately lacks documentation about how to best structure your application. With draft.js it's easy to get into a vicious cycle of building complex component state and highly coupled functionality.

This is where [draft-js-plugins](https://www.draft-js-plugins.com/) become very helpful indeed - because you get a plugin architecture with it. It's fairly straight forward to [write your own plugin](https://github.com/draft-js-plugins/draft-js-plugins/blob/master/HOW_TO_CREATE_A_PLUGIN.md) with  `draft-js-plugins` and it gives you the opportunity to separate your draft-js functionality.

Right, that's it. Again, if you 
