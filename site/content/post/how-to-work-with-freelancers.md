---
title: "Focus is all - How to successfully work with freelance developers"
date: 2017-06-28T09:15:00.000Z
description: "A list of things about what I learned can enable a successful contracting arrangement between a software engineer and the organization that's hiring him."
---

### First, a little background
![Tettra Logo](/img/blog/tettra.png)

[Tettra](https://tettra.co) searched for a draft.js specialist and got in touch with [G2i](http://g2i.co) run by the amazing Greenberg brothers [Gabe](https://twitter.com/gabe_g2i) and Julian. I had joined [G2i](http://g2i.co), a talent platform specifically for React & React Native developers and they matched me with [Tettra](https://tettra.co). The contract was an exemplatory success (for both sides I believe/hope ;), here's what went right and what you can do to make your engineering contracts more successful, from both points of view:

### Avoid distractions at all cost
Focus for engineering work is a requirement for efficiency. If the engineer is allowed to put all efforts into their work without having to think of anything outside their primary responsibilities, that's what enables establishing focus. All below points are things that helped achieve focus in the working arrangement I had with tettra:

### When you know what you want to do, get a specialist (or be one)
It may be that a developer is good in many things, maybe even design and project management. But switching between different tasks can cost both sides in terms of efficiency.

- As the client when you know what you want, it enables you to hire a specialist. It's generally more efficient to hire someone who can provide quality results in a timely manner.
- As a developer/contractor you'll be able to provide more value, let alone for the fact that you can achieve more focus by avoiding the cost of switching between different areas of expertise. More value means happier clients, more business, higher rates and consequently a happier self.

### Onboarding needs to be smooth
Make sure that your contractor has all information and access to services that he needs to work and communicate with you, those are things like:

- Access to __version control__ like github, gitlab or whatever it is you're using.
- Access to the company __group chat__ (if you haven't got one you should, you can use [slack](https://slack.com/) or [discord](https://discordapp.com/)) they are both great products.
- Access to your CI (like [travis](https://travis-ci.com), [circle](https://circleci.com/) or your jenkins instance).
- Access to any guidelines the developer should related to the development process.
- All the info that the developer needs to get their dev setup sorted (if you use a VM there may be a more complicated setup procedure). With tettra this was exceptionally easy, I just ran `yarn` and `yarn dev` and could start work :). 10 min start time.
- Make sure there is someone on call to respond to the engineer when they start work, in case there are any issues.
- If the contractor is standing in for someone who's on holiday, make sure there is an overlap, so any incomplete work can be handed over to the newly onboarded engineer.

### Good communication

- __Be available via group chat__. It's not enough to have a slack channel. All involved should be online the majority of their work day so they can chat or pair. If your company has an office and they only occasionally contract with remote developers, communication like slack or skype becomes increasingly important.
- __Don't have too few, or too many meetings__. Meetings can take up time so there's a balance to be had. But also make sure you do meet and at least spend a little time to get to know each other.
- __Establish a positive tone__. Personally or professionally, it's best to start off on a positive note, maybe with some authentic small talk (Yes, there is such a thing :P). Introduce yourselves, laugh a little! Making friends is what networking is all about after all since when you work together you want to enjoy each other's company.
- __Pair sometime__. Two eyes are better than one when it comes to finding issues. If the company's engineers and the contractor pair, they can learn from each other, debug problems faster and subsequently create more value.

I want to commend [Shauni](https://twitter.com/rogueraspberry) - Tettra's incredibly supportive, __well organized Head of Engineering__ who repeatedly reached out to make sure communication is working well and if the team could do anything to accomodate me better (That alone is unheard of <3).

![Shauni on slack](/img/blog/shauni-on-slack.png)

### Thorough planning is key
Having the work a contractor is tasked with documented the beginning of the contract is incredibly helpful. Bear in mind that this could also be part of the contractors job, but that needs to be explicit (this is a very common occurence btw), otherwise time will be lost. This is amplified when the contractor is remote and/or works part-time (which was my case).

- Make sure that all work the engineer is tasked with is accounted for and accessible. The contractor needs to know what they are working on. If there are holes in your stories or specifications, both sides will have to spend more time on communication than necessary.
- If there is time pressure, communicate your expectations with the developer. As long as there is good documentation and it is the developer's area of expertise, they should be able to provide a more or less accurate estimate.

### Quality Assurance
In a contract arrangement like this, quality must be a major concern. Both as the organization and the developer you want to make sure that nothing will break when your work is deployed to production and that your engineering guidelines are adheered to.

- __Have a testing plan__. I'm not saying you have to have unit tests for everything. It really depends on the type of product you're building, sometimes integration testing can be a better solution. However, you'll need some way to assure that you won't break anything in production and that gets especially more important when a team grows, whether you're adding a permanent or temporary engineer.
- Review each others code. There's now one more member on the team. By reviewing each others code we can learn from each other meanwhile ensuring that we're not breaking any conventions or are making architectural mistakes. Again, a second pair of eyes goes a long way. According to Kevin Burke, [code reviews](https://kev.inburke.com/kevin/the-best-ways-to-find-bugs-in-your-code/) are more efficient when it comes to catching bugs.

----

Well that's pretty much it, last but not least - I want to thank these people for making this contract the pleasant learning experience it was!

- Shoutout to [Shauni](https://twitter.com/rogueraspberry) for being an excellent Head of Engineering - well organized and supportive and accomodating. Also shoutout to Shauni and [Oscar](https://twitter.com/oscargemorrison) for being incredibly supportive, constantly reviewing my prs and helping me to move forward.
- Shoutout to Gabe and Julian Greenberg from [G2i](http://g2i.co), a React & React Native developer platform, who connected me with [Tettra](https://tettra.co).
