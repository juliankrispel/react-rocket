---
title: "Notes on working with draft js"
date: 2017-05-09T15:04:10.000Z
description: Here's a list of learnings from using draft js while working as contractor for Tettra
---

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
