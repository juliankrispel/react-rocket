---
title: "Notes on working with draft js"
date: 2017-05-09T15:04:10.000Z
description: Here's some notes/opinions on using draft js.
---

Doing work for my clients and also talking to the community on the draft-js slack I've received much help and I've also seen a lot of the same questions.

## When using draft.js and redux together, 



## Only use convertToRaw and convertFromRaw when absolutely necessary
`convertToRaw` and `convertFromRaw` are used for converting draft.js state into serializable objects and vice versa, making your `contentState` persistable.




- Write your own helper methods and put them online!
- When you write helper methods to modify editor content, prefer returning ContentState
- Always use the children prop type to render entity content
- Avoid using `forceSelection` - it can sometimes cause unexpected behaviour.
- Avoid adding interactive elements to decorators unless it really makes sense
- Selec

## Notes on design:
- I like that there is not too much opinion in this framework
- I like that it isn't too powerful, I feel like I only get the tools that I really need - kudos to the architects at facebook, I'm having consistently good experiences with facebook libraries!
- People want to have different types of 'atomic' blocks, yet in draft js atomic is the block type, this doesn't make much sense, it'd be better to be able to specify a block as atomic.

## Questions
- What are atomic block types?
