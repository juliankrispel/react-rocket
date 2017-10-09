---
title: "React: Turn your hoc into a render prop component with one line of code"
date: 2017-10-09T15:32:00.000Z
description: In this blog-post you'll learn how easy it is to turn higher order components like connect from react-redux, graphql from apollo, or any other higher order component into a normal component with a render prop. Which in turn will enable you to use those components dynamically, inside your render methods.
---

Disclaimer: This is not a rant about how nasty higher order components are and how much better render props are. Instead what I'm trying to show you is how we can combine them and use the advantages of both!

Higher order components are useful! For a while now they've been the go-to abstraction to compose functionality for the react community. However there are some limitations with `hoc`s, here are three of them:

1. As a user it's not explicit what props a higher order component will add.
2. Higher order components don't protect you from prop collision.
3. Higher order components restrict us to static composition. You can't just use a hoc inside JSX, you have to do so at the edge of a component. This means that our usage of hoc's often determine our component boundaries :(.

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

If you've been a user of recompose like myself and some of the teams I've been working with you won't have to imagine it ;)

## Render Props to the rescue

You've probably heard of the render prop pattern, either from [Michael Jackson's great article](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) or from using [react-router](https://reacttraining.com/react-router/), [react-motion](https://github.com/chenglou/react-motion) or another of the many libraries which uses the render prop pattern, or from way back when, when react-redux still had a `Connector` component :D.

If you haven't, let's have a quick look at an example, here's one from the readme of [react-responsive](https://github.com/contra/react-responsive#rendering-with-a-child-function)

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

Ok so what's happening here. As you can see we have a react expression inside our MediaQuery component, this contains a function, which gets passed an boolean argument. This is great stuff because both of our 2 problems are defeated:

1. Since the state you're using from your higher order components is clearly exposed, you don't have search for this information anymore.
2. There's a lower likelihood of prop collision (except between the edges of the hoc's themselves).
3. Although we still have to configure our hoc's statically, we're not confided to the edge of components anymore. We can use them anywhere within our render methods.

### Turning your HOC into a Component with a render prop in one line

But what if you have a code-base full of higher order components, you can't just ditch everything right?

Fortunately, it's pretty easy to turn a higher order component into a component with render props. All we need is a component with a render prop and wrap it with our hoc like so:

```
const RenderComp = ({ children, ...props }) => children(props);
const RenderHoc = myHOC(RenderComp);
```

Please note - We're using the `children` prop here, but really you can use any prop name you like.

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

Let's dive straight in with a bunch of real world examples that you can start using right now!!!

### react-redux - connect hoc

Probably the most used hoc in our community.

```
const RenderComp = ({ children, ...props}) => children(props);

const mapStateToProps = ({ currentUser: { firstName, lastName } }) => ({ firstName, lastName });
const ConnectUser = connect(mapStateToProps)(RenderComp);
```

So instead of wrapping your main component you just wrap it in your render prop component and now you can use it anywhere in your render methods:

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

Another well used higher order component nowadays would be the apollo graphql hoc. Here's an example of a configured graphql hoc, we'll do the same thing again:

```
const graphqlHoc =  graphql(gql`
  query TodoAppQuery {
    todos {
      id
      text
    }
  }
`);

const RenderComp = ({ children, ...props}) => children(props);
const GraphqlTodos = graphqlHoc(RenderComp);
```

We used the same simple pattern again, now we can use it as a component in our render method:

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

Recompose is a hoc tool belt. It's widely used and made by the amazing [Andrew Clark](https://twitter.com/acdlite). Recompose encourages composition with higher order components and as such it can be very powerful. Let's consider we have a code-base that makes heavy use of recompose. Here's an example of a composed higher order component straight [from the recompose docs](https://github.com/acdlite/recompose/blob/master/docs/API.md#withhandlers):

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
Please note: Instead of having a separate component I'm just creating the component definition ( a plain arrow function ) inline, to show you a shorter alternative.

And yet again, we can use it as a component, anywhere we please, and the props that the higher order component produces won't clash with other props:

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

Here's one final example using RxJS in combination with recompose. Here's the example I took straight from the recompose docs:

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

Let's not talk about the implementation here, you don't really need to know how RxJS works in order to refactor this component to use a `renderProp`. All one needs to do is change the render method at the lower half of the component definition. Here's how that would look like:

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

All I did here is remove all jsx and simply call the children function with the state we need.

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

I hope this post makes it pretty clear how simple it is to turn your hoc components into render prop components and what power it yields. If you're excited about this please have a look at [Michael Jackson's article](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) as well as his presentation at React Phoenix - [never write another hoc](https://www.youtube.com/watch?v=BcVAq3YFiuc), gotta love that title :D
