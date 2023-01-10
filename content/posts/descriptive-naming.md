---
title: "Give Components Descriptive Names"
date: 2023-01-10T18:26:00+00:00
summary: Some thoughts on descriptive vs fun naming of system components.
draft: false
---

I've seen a few blogs and articles recently about eschewing descriptive names for cute or fun ones.

Funny, I distinctly remember the era of cute names for servers and fun/sci-fi names for internal systems. It was terrible, nobody knew what anything did, but everything old is new again I guess...

Sure, it is a reasonable argument that as time goes on, as we develop and iterate on our system designs and components, their names will become less meaningful - and renaming them later near impossible.

But that's hardly an argument to throw out descriptive naming altogether!

It's always good to remember who the target audience is when we build something: the people who come after us (including ourselves, when our mental contexts have switched elsewhere), not the computer. The computer doesn't give two fucks what you call your fancy new microservice, it will run just the same.

Give components a descriptive name, it's function might change, but it will be easier to figure out the `notifications` service sends emails than if `galactus` does.
