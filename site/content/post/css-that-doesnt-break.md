---
title: "CSS is broken."
date: 2017-09-13:15:00.000Z
description: As anyone knows, CSS kinda sucks. While contracting for various startups, I still find that most teams get CSS wrong.
---

CSS sucks. There you go, I said it. I'm not the first one though, surely you've heard that one before. There's a reason there are so many projects out there right now trying to change the way we write CSS. There are hundreds of CSS-in-JS libraries for one purpose - To give CSS some of the characteristics that programming languages have, empowering us to write reusable code.

Reusable CSS however, is a pipedream for most developers.

## Here's what a typical workflow of a frontend developer looks like

Writing a frontend component for an app or a website or whatever, we'll typically write a view first. This view consists of two things, the markup (html) and the styling (css). Sometimes we use a CSS framework, sometimes we have our own custom ones, whatever the case, what usually happens is that we have to write CSS for this new view. Let's say we're implementing a profile view, so we'll start a new css (or scss, or less) file called profile.css.

### Scenario A - Custom CSS

We have a custom CSS codebase, no CSS framework is used. We write the html and assign classes to each element

## Reusability


## Anti Patterns

### 1. Element rules

To make HTML more accessible it should be as minimal as possible right?
TODO: Read about semantic web.
TODO: Investigate whether lots of class names actually make a website less accessible.

This is an anti pattern that I see all the time. It's endorsed by people who believe that html should be as minimal and therefore as `accessible` as possible. 

### 2. Nesting rules

Nesting rules make it close to impossible to have mobile code. You'll be battling specificity.


### 3. Writing custom css for different views

This is the opposite of DRY and it happens all the times. Typically we write custom css for every view. This is often combined with nesting and it encourages us to think about css in a very coupled way. We're basically writing the same boilerplate code every time. Also, it's the opposite of scalable, since the CSS for our app/website grows with every new view we implement.


## The solution: composable CSS

The idea of reusable CSS is nothing new. The first well defined outline for reusable CSS was in the form of Nicole Sullivans [OOCSS](https://github.com/stubbornella/oocss). I also.

Right now I'm using tachyons.

The idea is, write your CSS as composable CSS.
