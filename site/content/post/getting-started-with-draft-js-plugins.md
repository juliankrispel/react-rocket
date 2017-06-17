---
title: "Getting started with draft.js plugins"
date: 2017-06-16T14:49:00.000Z
description: Draft.js is a very powerful framework on its own. But if you've ever used the framework you'll know that it's very easy to build up complexity, draft.js plugins help with that.
---

Heads up: To follow along with this tutorial some experience with draft-js would be beneficial. Feel free to stop here and have a look at my other blogpost on [getting started with draft js](http://reactrocket.com/post/getting-started-with-draft-js/).

Draft.js is a powerful framework for creating rich text editors. But like anything, it comes with draw-backs. Draft.js plugins, the brainchild of the amazing human being that is Nik Graf (aka the nicest man on earth) is a plugin system that solves a major issue.

## Draft.js doesn't encourage modularity by default
The way draft.js is designed doesn't encourage splitting your editors functionality into modules. As a result, what I've seen first hand working on production draft-js codebases is that functionality is often tightly coupled, which makes code hard to read, test and share.

Facebook libraries usually (like react) don't come with opinions on how you should structure your application. With draft-js being as young as it is there isn't much out there to guide you on how to structure your application. Draft.js plugins solves this by providing a plugin system. Rather than putting all your functionality into one messy component, plugins encourage you to place your functionality into distinct code buckets that you can just plug into your editor, like this:


```js
<PluginEditor
  editorState={this.state.editorState}
  onChange={this.onChange}
  plugins={[hashtagPlugin, mentionPlugin, linkifyPlugin, dragAndDropPlugin]}
/>
```

As you can see above, we extend the editors functionality just by handing it a bunch of plugins, each one of them we can disable/enable by adding/removing it from the plugins prop. By enabling modularity for draft-js, plugins open up a space for the community to share functionality amongst each other.

This is very important for the community and open source work. Making draft-js functionality modular means we can share our code, and extend our editors by using plugins from other contributors.

## Draft.js doesn't come with a lot of default functionality
This is another problem that plugins solve. Currently, there are 17 (and growing) plugins in the [draft-js-plugins repository](https://github.com/draft-js-plugins/draft-js-plugins). They cover use-cases such as emojis, @mentions, autolinking, embedding video, stickers images and more. There's even a decent [implementation for tables](https://github.com/draft-js-plugins/draft-js-plugins/tree/master/legacy/draft-js-table-plugin), which has been deprecated because it's hard to maintain (draft.js doesn't really cater for nested document structure).

So by default draft-js-plugins come with a ton of functionality!

## Getting started with plugins
So let's get started using some draft-js plugins. It's pretty simple really. I'm writing the code for this tutorial as I'm writing the tutorial so feel [free to follow along here](https://github.com/juliankrispel/getting-started-with-draft-js-plugins). If you see a commit hash somewhere (that looks like this #321df1rf4ej280fhje2j) it's a link to github illustrating that particular point in the tutorial.



