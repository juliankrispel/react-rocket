---
title: "Good practices when working with draft js"
date: 2017-05-09T15:04:10.000Z
description: This is a collection of tips and opinions on using draft-js. Given the framework's youth and lack of guidance on the internet, I feel like sharing some of my experience in the hope it'll be useful to others starting out.
---

__Disclaimer:__ It's hard to exclude my personal opinnion from this blog post, but I'll try as best as I can. Please call me out if you think I'm wrong or if you agree with me, or if you want to help submitting a pull request containing some known best practices. Also, please join the [draft.js slack](https://draftjs.herokuapp.com/) or the [reactiflux](https://www.reactiflux.com) __draft-js__ channel, there's usually some great, friendly people who just like you, want to figure stuff out!

Doing work for my clients and also talking to the community on the draft-js slack I've received much help and I've also seen a lot of the same questions. So I figured it's time to start writing down what I have learnt so far, with the hope of helping others.

### 1. When using draft.js and redux together, make sure you use redux's optimization features.
In conversations with community members I've heard reports that redux slows their app down when used in combination with draft.js. This really surprised me because it hadn't happened to me so I dug in. Here's a common scenario:

- There's a react component containing the draft.js Editor, this component is connected with the redux store, which stores the editor's state.
- The state gets updated every time the editor fires an `onChange` event.
- The editor state makes it's round trip, but there appears to be a performance penalty when doing so, interacting with the draft.js editor feels sluggish.

In all cases it became apparent that the entire app, or at least a significantly large part of it was rerendering, because the react-redux's `connect` method couldn't use it's 

### 2. Only use convertToRaw and convertFromRaw when absolutely necessary
`convertToRaw` and `convertFromRaw` are used for converting draft.js state into serializable objects and vice versa, making your `contentState` persistable.


### 3. Always use the `children` prop type to render content.
For [custom components](https://draftjs.org/docs/advanced-topics-block-components.html) and [decorator components](https://draftjs.org/docs/advanced-topics-decorators.html), you need to use 

### 4. When necessary, optimize your custom block components with `shouldComponentUpdate`.

### 5. There's no good way to implement tables right now.
Draft.js's content model is flat. That's a restriction that comes with it's pros and cons. Nested structures are harder to optimize and reason about, so this makes maintaining draft.js easier. However, this restriction means that implementing a table layout is hard to do without nesting editors. And nesting editors, although it's doable, come with a bunch of extra complexity, also not optimal.

### 6. Be careful with your architecture
Developing an architecture for a draft.js application is hard. There's not much advice out there and the draft.js website unfortunately lacks documentation about how to best structure your application.

### 7. 

- When you write helper methods to modify editor content, prefer returning ContentState
- 
- Avoid using `forceSelection` - it can sometimes cause unexpected behaviour.
- Avoid adding interactive elements to decorators unless it really makes sense
- Selec

## Write your own helper methods and put them online!

## Notes on design:
- I like that there is not too much opinion in this framework
- I like that it isn't too powerful, I feel like I only get the tools that I really need - kudos to the architects at facebook, I'm having consistently good experiences with facebook libraries!
- People want to have different types of 'atomic' blocks, yet in draft js atomic is the block type, this doesn't make much sense, it'd be better to be able to specify a block as atomic.



## Questions
- What are atomic block types?
