---
title: Development Rules and Guidelines For Building Software
date: 2024-01-11T11:24:34.635Z
description: Some rules and guidelines I've discovered from my time developing
  software with teams.
draft: false
showHero: true
showComments: true
thumbnail: /img/referree.jpeg
tags:
  - rules
---
This post started with me trying to write down some guidance for my current team, at some point I someone on my team suggested I should make it more broadly available. While this advice is from my person experience developing on team and managing development teams, it's not all original insights. I have been very influenced by [Martin Fowler's book "Refactoring".](https://martinfowler.com/books/refactoring.html) That book is a great read for anyone who wants to get a practical, straightforward introduction to the craft of writing sustainable code. Also, I enjoy the perspective of [the Grug Brained Developer](https://grugbrain.dev/) and that has likely influenced my approach. I've also benefitted greatly from the insights of [Brian Levy of BridgePort Digital](https://www.bridgeportdigital.com/meet-the-team). When in doubt, assume most of the best insights come from him 😉.\
\
I may make some follow up posts explaining each of, or particular guidelines. Let me know in the comments if that is something you'd like to read. Feel free to put out where you disagree or where it's not clear what I meant.

- - -

There is a difference between a **rule** and a **guideline**. A *rule* is an edict which is imposed via authority or convention - and they are to be followed blindly in order to keep the order, avoid unnecessary conflicts, and keep thing running smoothing and efficiently. A *guideline* is a suggestion which should be taken seriously, considered, referenced, but not necessary followed blindly. It is important to know the difference. There are many guidelines, but only 4 rules.

### Rules

1. **RULE #1**: **D.F.B** (*Don't Fuck (with the) Build*)!!! Don't make a PR where you've added a dependency that prevents or *even complicates* a program from running seamlessly via its entry point. And if for some reason you MUST - you need to **overly explicit** about this in PRs - and make certain anyone who may be viewing the PR is 100% clear about the dire consequences of approving said PR. Any changes in the process of building the software should be the product of a discussion with the team and relevant stakeholders - not a result of your careless 🐂 💩 pull request. This rule is *by far* the most important.\
   **NOTE: The local development build IS INCLUDED in this rule.**
2. **RULE #2**: Use all agreed upon formatters (black, go-fmt, prettier, etc) with agreed upon configurations. No one wants to review a PR and need to separate the meaningful changes from your idiosyncratic formatting preferences.
3. **RULE #3**: Use all agreed upon linters. Linters are great! They help you avoid a whole set of bugs. And they often can be configured to reflect agreed upon conventions. No one should have to make a comment in a PR that a linter would've solved!
4. **RULE #4**: Nothing is released or merged into master without an actual PR and at least one new test per new functionality.

These are all the rules.

The rest here are guidelines.

- - -

### Guidelines

I will first list the guidelines and then provide explanations for reference as needed.

1. **D.M.T: Don't Make Me Think!**

   1. Don't do clever 💩! (I don't want to have to call you during your vacation to figure it out.)
   2. Functions or methods should maintain a singular level of abstraction.

      * This allows for a better understanding of what is relevant to a given function and what the developer needs to be concerned with.
      * There are times when you should ignore this guideline - particularly when the boundaries of an abstraction are not understood well enough to be reliable. (EXAMPLE: when you're prototyping a system you're not familiar with.) In such cases, it is often better to leave the levels together and refactor later, once the boundaries are better understood. What this means is you're essentially forcing the next developer editing the code to hold all the complexity in her head at once, rather than abstracting away something that is not well understood / implemented and could be a dark breeding ground for bugs and exceptions.
   3. Make reliable and understandable data types for your application.
   4. Entities (classes, modules) should have a single, easily understandable, purpose.
   5. Name functions for what they do (start with an "action" verb).
   6. Write features with testability in mind, and you will have a more easily understandable. Doing this makes your app better **even without the tests!**

      * Have well defined inputs and outputs.
      * Don't require a function to be overly context specific.
2. **Code Smells - Pass the Sniff Tests**

   1. When writing code, you should have some reasonable, albeit arbitrary, points where you must ask yourself certain questions. For example, let's say that you've found yourself writing an elixir module with over 500 lines of code, perhaps at this point you should ask yourself something like: "Does this module do to much?" Perhaps you'll tell me that 500 lines of code is reasonable. I do not wish to argue with you about how many lines of code are too many. I do not wish to argue with you about whether it is lines of code you should be counting, or something else. All I want to know is that you have some reasonable threshold of when you ask yourself "does this *thing* do too much?"
   2. There is a useful list of code smells from Martin Fowler in his book Refactoring. There's also [a paper about code smells](https://github.com/Luzkan/smells/blob/main/docs/paper.pdf) written by Marcel Jerzyk and Lech Madeyski that does a deep dive into code smells in general and a little [web app](https://luzkan.github.io/smells/) Marcel Jerzyk made about it. **None of this should be taken as authoritative** but you and your team should consult with some of them to determine which one resonate.
3. **Manage Dependency Hell**

   1. Don't add a new 3rd party dependency for any problem that can be adequately solved in 20 lines of code or less.
   2. When adding a new 3rd party dependency, consult with whatever person / team manages that software first.
   3. When appropriate, separate the specific implementations from what you want your application to do.

      * The simplest way I know of to do this is via the use of the strategy pattern and dependency injection.
   4. If your product does not **currently** yet need to scale like Netflix, Google, etc - and you can't explain to me, **without using any of the following words**, or close synonyms, why something should be a micro-service, it probably should first be a monolith. The words are: 

      1. scaling 
      2. modern
      3. containers
      4. kubernetes
      5. cloud
      6. aws
      7. horizontal
      8. infrastructure
   5. There is an exception to previous statement. This is when you've ignored all the other guidelines to the point where you system is *already totally ummanagable!* At that point, it may be easier to make a little external micro-service.\
      **Explanation:** The reason for this insistence on the monolith is not that I am luddite, who is against the whole containers / docker / k8s innovations that we've invented for scaling systems. It's because I've found that it is always easier to break up a monolith into micro-services, once you've written the  code and understand the problem space. While I think this is true it is not always a compelling enough reason to start messing around with a legacy code base that is not well maintained and you don't know what else you'll break. You may find that you're spending too much time addressing other issues with the code base rather than working on the stuff you're supposed to accomplish.