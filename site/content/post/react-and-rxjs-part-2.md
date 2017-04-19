---
title: "React and RXjs Part 2 - The Power of streams"
date: 2016-04-20T15:04:10.000Z
description: This series is about using react in combination with rxjs, for fun event-based programming. In this second installment we'll engineer something more complex to show off the power of stream-based programming.
---

## Recompose
We'll use [recompose](https://github.com/acdlite/recompose/blob/master/docs/API.md) for this part, because it gives us higher order components that let us:
- use rxjs observables with react.
- automatically bind react views to streams.
- contain state within streams rather than component state (a code smell).
- use react in the recommended functional way (stateless/functional components).

If you're not familiar with higher order components, have a look at [this part of the react docs](https://facebook.github.io/react/docs/higher-order-components.html), this [egghead lesson](https://egghead.io/lessons/react-react-fundamentals-higher-order-components-replaces-mixins), or Andrew Clarks [excellent presentation about hoc's](https://www.youtube.com/watch?v=zD_judE-bXk), check it out!

Let's start with a simple 

