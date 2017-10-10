---
title: "Upgrade your react.js hoc with renderProps"
date: 2017-10-09T12:45:00.000Z
description: In this blog-post you'll learn how easy it is to turn higher order components into normal components with render props.
---

![HOCs as Components](https://i.imgflip.com/1xcf7x.jpg)

### __tldr;__

In short, this post is about how you can take higher order components such as `connect()` from react-redux, `graphql()` from apollo or any other and use them like normal components anywhere within your render cycle, with the render prop pattern.

__Heads up__: This is not a rant about higher order components vs render props. In fact the two are very easy to combine and in doing so you can overcome most of the drawbacks of hoc's. First of all, let's outline what those drawbacks are:

### The disadvantages of higher order components

Higher order components have become ubiquitous! They've been the go-to abstraction for composition in the react community. However there are some limitations with hoc's, here are three of them:

1. As a user of higher order components it's not explicit what props will be added.
2. Higher order components don't protect you from prop collision.
3. Higher order components restrict us to static composition. You can't just use a hoc inside JSX, you have to do so at the edge of a component. This means that our usage of hoc's often determine our component boundaries and vice versa :(.

Now imagine debugging the following:

```js
compose(
  connect( ... ),
  enhance( ... ),
  withRouter( ... ),
  lifecycle( ... ),
  somethingElse( ... )
)(MyComponent);
```

In my experience from writing apps with the pattern - code that uses lots of functional composition can be very hard to penetrate. Having tried this with a bigger teams, I've often had feedback that working with such a code-base is harder than it should be.

## Render Props to the rescue

You may have heard of the render prop pattern or at least seen it in action. Michael Jackson wrote a [great article](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) on the topic. Even if you haven't heard of it you'll probably have used it with with libraries like [react-router](https://reacttraining.com/react-router/), [react-motion](https://github.com/chenglou/react-motion) or another of the many libraries which use this pattern.

What it boils down to is this: callbacks which are used for rendering, passed down to a component as prop - Hence `renderProps`.

__First, here's an example__ to show you how it looks like, taken from the readme of [react-responsive](https://github.com/contra/react-responsive#rendering-with-a-child-function)

```jsx
<MediaQuery minDeviceWidth={700}>
  {(matches) => {
    if (matches) {
      return <div>Media query matches!</div>;
    } else {
      return <div>Media query does not match!</div>;
    }
  }}
</MediaQuery>
```

Okay so what's happening here. As you can see we have an expression inside our `MediaQuery` component, containing a function. Just by looking at it you can see that this defeats 3 of our problems:


1. The state that the component is responsible for is clearly exposed.
2. Prop collision is suddenly a non-issue, since props are only passed as arguments to our render prop rather than our child components.
3. Place your component anywhere: Although we still have to configure our hoc's statically, composition is not confided to the edge of components anymore. Our hoc's are now completely mobile and we're free to place them anywhere within the render cycle.

## <div class="tac"> :tada: :tada: :tada: </div>

__By the way__ this pattern is actually quite old. Here's an example of [render props](https://gist.github.com/gaearon/b0c060edf413fe23db0a#gistcomment-1410409) from March 2015 by none other then [Dan Abramov](https://twitter.com/dan_abramov), weeks before the first commit of [redux](https://github.com/reactjs/redux).

### Upgrading your HOC into a Component with a render prop with one line of code

But what if you have a code-base full of higher order components, you can't just ditch everything right?

Fortunately, it's pretty easy to turn a higher order component into a component with render props. All we need is a component with a render prop and wrap it with our hoc like so:

```
const RenderComp = ({ children, ...props }) => children(props);
const RenderHoc = myHOC(RenderComp);
```

Please note - We're using the `children` prop here, but really you can use any prop you like.

And now we can use it like so:

```
<RenderHoc>
  {({ firstName, lastName }) => (
    <div>
      <h1>{firstName}</h1>
      <h2>{lastName}</h2>
    </div>
  )}
</RenderHoc>
```

#### That's really all there is to know about this pattern

You now have everything you need to use your higher order components as normal Components, with the freedom to render them dynamically, anywhere in your render cycle.

In case it hasn't sunk in yet, let's have a look at some examples. I'll take some popular higher order components and turn them into normal components with render props:

### react-redux - connect hoc

Probably the most used hoc in our community.

```
const Render = ({ children, ...props}) => children(props);

const mapStateToProps = ({
  currentUser: {
    firstName,
    lastName
  }
}) => ({ firstName, lastName });

const ConnectUser = connect(mapStateToProps)(Render);
```

So we define our renderProp component and our `mapStateToProps` which we need for the `connect` hoc. Then we wrap our `connect` hoc around our `Render` component and voila, we can render it as a normal component:

```
render() {
  return (
    <div>
      <Toolbar />
      <ConnectUser>
        {(currentUser) => <Profile currentUser={currentUser} /> }
      </ConnectUser>
    </div>
  )
}
```

### apollo - [graphql hoc](http://dev.apollodata.com/react/higher-order-components.html#graphql)

Another well used higher order component nowadays would be the apollo graphql hoc. Here's an example of a configured graphql hoc, which we turn into a component with a render prop:

```
const graphqlHoc =  graphql(gql`
  query TodoAppQuery {
    todos {
      id
      text
    }
  }
`);

const Render = ({ children, ...props}) => children(props);
const GraphqlTodos = graphqlHoc(Render);
```

Same exact pattern, wrapping our graphql hoc around our Render component. And now we can render it anywhere:

```
render() {
  return <div>
    <Header />
    <Sidebar />
    <main>
      <GraphqlTodos>
        {({ todos }) => todos.map((todo, index) => <Todo todo={todo} key={index} />)}
      </GraphqlTodos>
    </main>
  </div>
}
```

### [recompose](https://github.com/acdlite/recompose/blob/master/docs/API.md)

Recompose is a hoc tool belt. It's widely used and made by the admirable [Andrew Clark](https://twitter.com/acdlite). Recompose encourages composition with higher order components and as such it can be very powerful. Let's consider we have a code-base that makes heavy use of recompose. Here's an example of a composed higher order component straight [from the recompose docs](https://github.com/acdlite/recompose/blob/master/docs/API.md#withhandlers):

```jsx
const withValue = compose(
  withState('value', 'updateValue', ''),
  withHandlers({
    onChange: props => event => {
      props.updateValue(event.target.value)
    },
    onSubmit: props => event => {
      event.preventDefault()
      submitForm(props.value)
    }
  })
)
```

Now all we do is wrap it around a render prop component as we have done so far:

```
const WithValue = enhance(({ children, ...props }) => children(props));
```

Please note: Instead of having a separate component I'm just creating the component definition ( a plain arrow function ) inline for brevity.

And yet again, we can use the hoc as a normal component, anywhere we please.

```
render() {
  return <div>
    <WithValue>
      {({ value, onChange, onSubmit }) => (...)}
    </WithValue>
  </div>;
}
```

### RxJS and recompose

Here's one final example using RxJS in combination with recompose, taken straight [from the docs](https://github.com/acdlite/recompose/blob/master/docs/API.md#withstatehandlers):

```
const Counter = componentFromStream(props$ => {
  const { handler: increment, stream: increment$ } = createEventHandler()
  const { handler: decrement, stream: decrement$ } = createEventHandler()
  const count$ = Observable.merge(
      increment$.mapTo(1),
      decrement$.mapTo(-1)
    )
    .startWith(0)
    .scan((count, n) => count + n, 0)

  return props$.combineLatest(
    count$,
    (props, count) =>
      <div {...props}>
        Count: {count}
        <button onClick={increment}>+</button>
        <button onClick={decrement}>-</button>
      </div>
  )
})
```

Let's not talk too much about the implementation here, you don't really need to know how RxJS works in order to refactor this component to use a `renderProp`. All one needs to do is change the render method at the lower half of the component definition. Here's how that would look like:

```
const Counter = componentFromStream(props$ => {
  const { handler: increment, stream: increment$ } = createEventHandler()
  const { handler: decrement, stream: decrement$ } = createEventHandler()
  const count$ = Observable.merge(
      increment$.mapTo(1),
      decrement$.mapTo(-1)
    )
    .startWith(0)
    .scan((count, n) => count + n, 0)

  return props$.combineLatest(
    count$,
    (props, count) => props.children({ count, increment, decrement })
  )
})
```

All we did here is remove all jsx from the view and simply call the children prop function with the state we need to expose.

So now what I can do is this:

```
<Counter>
  {({ count, increment, decrement }) => (<div>
    Count: {count}
    <button onClick={increment}>+</button>
    <button onClick={decrement}>-</button>
  </div>)}
</Counter>
```

As you can see, I am now completely in control of the actual presentation. All the counter component does is manage and expose the state I need to interact with the component.

### Conclusion

I hope this post makes it pretty clear how simple it is to turn your hoc components into render prop components and what power it yields. If you're excited about this as much as me please have a look at [Michael Jackson's article](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) as well as his presentation at React Phoenix - [never write another hoc](https://www.youtube.com/watch?v=BcVAq3YFiuc), gotta love that title :D
