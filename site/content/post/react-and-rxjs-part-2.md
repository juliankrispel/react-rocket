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

Let's start with a simple example, we'll use the counter example from part 1, which is also shown as an example in the [recompose docs](https://github.com/acdlite/recompose/blob/master/docs/API.md). Let's go through it step by step:

First, create a scaffold with `create-react-app`, then `npm install --save rxjs recompose`.
At the top of your `/src/App.js` you'll need to import some modules that you'll be using and set up recompose to use rx observables (you don't have to use rxjs, you can use bacon, kefir and other reactive libraries with recompose).

```js
// default create-react-app imports for App.js
import React from 'react';
import './App.css';

// We'll need the Observable class as well as some helpers and hocs from recompose
import { Observable } from 'rxjs/Observable';
import componentFromStream from 'recompose/componentFromStream';
import createEventHandler from 'recompose/createEventHandler';

// we use setObservableConfig to tell recompose to use rxjs
import setObservableConfig from 'recompose/setObservableConfig';
import rxjsconfig from 'recompose/rxjsObservableConfig'
setObservableConfig(rxjsconfig)
```

Recompose works nicely with stateless components, this means you don't need to use classes or state, you can just compose functions, very nice (I know - extra hip - but it gives you a bunch of benefits, functions are much easier to test for example)!

We're using the `componentFromStream` higher order component, it takes a function as an argument, which gets a props stream passed to it as an argument, we'll combine this `props$` stream with a a `count$` stream to produce another stream that prouduces a component. Components as streams!!!!

```js
const Counter = componentFromStream(props$ => {
  // createEventHandler gives us a `handler` and a `stream`, the handler lets us push values into the stream.
  const { handler: increment, stream: increment$ } = createEventHandler()
  const { handler: decrement, stream: decrement$ } = createEventHandler()

  // We're mapping the increment stream to produce +1 and the decrement stream to produce -1
  const count$ = Observable.merge(
      increment$.mapTo(1),
      decrement$.mapTo(-1)
    )
    // the new observable will have the starting value 0
    .startWith(0)
    // and with the scan method we can produce the sum of the current value plus the new value that entered the stream (either +1 or -1)
    .scan((count, n) => count + n, 0)

  // now we combine the props$ stream and the count$ stream to produce a component stream.
  return props$.combineLatest(
    count$,
    (props, count) =>
      <div className="App-intro" {...props}>
        <div className="number">Count: {count}</div>
        <button onClick={increment}>+</button>
        <button onClick={decrement}>-</button>
      </div>
  )
});
```

Simple enough, and it works, tada:

![Counter App with rxjs and recompose](/img/blog/counter.gif)

Now let's produce something that shows off the power of rxjs, a TypeAhead, complete with fetching data and throttling.

First lets import all our dependencies again, just like before we're doing this in `/src/App.js`.

```js
// default create-react-app imports for App.js
import React from 'react';
import './App.css';

// We'll need the Observable class as well as some helpers and hocs from recompose
import { Observable } from 'rxjs/Observable';
import componentFromStream from 'recompose/componentFromStream';
import createEventHandler from 'recompose/createEventHandler';

// we use setObservableConfig to tell recompose to use rxjs
import setObservableConfig from 'recompose/setObservableConfig';
import rxjsconfig from 'recompose/rxjsObservableConfig'
setObservableConfig(rxjsconfig)
```

Then we use componentFromStream to create the component and bind it to a stream:

```js
const TypeAhead = componentFromStream(props$ => {
  // createEventHandler gives us a `handler` and a `stream`, the handler lets us push values into the stream.
  const { handler: inputChange, stream: inputChange$ } = createEventHandler();

  // inputValue$ will track the changes of the input element
  const inputValue$ = inputChange$.map((e) => {
    console.log(e);
    return e.currentTarget.value;
  })
  // We're starting with an empty input field,
  // rxjs has a handy method for initiating the
  // first value of a stream - startWith
  .startWith('');

  return props$.combineLatest(
    inputValue$,
    (props, inputValue) =>
      <div className="App-intro" {...props}>
        <input onChange={inputChange} value={inputValue} type="text" />
      </div>
  )
});
```

The above doesn't do much yet, it just streams the input value. Now let's load some matching data when we type something in. I'm going to use [rx-http-request](https://www.npmjs.com/package/rx-http-request) to fetch data from the [restcountries api](https://restcountries.eu).

For that I have to install the npm module - `rx-http-request` - yarn is my preferred package manager: `yarn add rx-http-request`.

First we need to import the module to use it.
```js
import { RxHttpRequest } from 'rx-http-request';
```
