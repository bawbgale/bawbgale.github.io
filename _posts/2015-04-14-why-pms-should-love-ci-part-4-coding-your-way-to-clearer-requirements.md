---
id: 74
title: 'Why PMs should love CI, part 4: Coding your way to clearer requirements'
date: 2015-04-14T05:04:08-08:00
author: Bob Gale
layout: post
guid: http://www.bawbgale.com/?p=74
permalink: /why-pms-should-love-ci-part-4-coding-your-way-to-clearer-requirements/
categories:
  - Uncategorized
tags:
  - agile development
  - bdd
  - behavior-driven development
  - CI for PM
  - continuous integration
  - program management
---
_Continuous Integration is not just for DevOps people. This is a series about why PMs should care about CI too and what they can do to help foster it in their teams._ 

One of the most important concerns of software development program managers is requirements definition. Whether you’re writing the requirements yourself or corralling others to do it, you know that everything flows smoother if everyone is on the same page about what the customer wants, how the team will split up the work, and how you’ll know when it’s done. Clear requirements help development teams stay focused, productive and motivated, and can avoid costly rework if the software doesn’t turn out as expected.

But that’s usually easier said than done. Pinning down the customer to a specific set of requirements is one challenge. Getting the development team to pay attention to those requirements is another. Software developers are not known for their love of reading requirements. They would rather be coding! (But wait, you say, in agile methodologies, stories are not supposed to be detailed specification documents. They are placeholders for conversations. Well, many software developers are not known for their love of conversations either. They would rather be coding!) So how can Continuous Integration help with this? By making requirements definition a coding exercise.

One of the core practices of CI is creating and maintaining a suite of automated tests that verify the correct functioning of the system as new features are developed. And one of the most effective ways of building such a suite is by adopting a test-first approach — i.e. starting work on a feature by writing tests that describe its expected behavior, watching the tests all fail when first run, then writing implementation code until all the tests pass.

A test-first approach lets developers start coding right off the bat, but encourages them to think about the expected behaviors of the system rather than the internal implementation of those behaviors. Rather than writing code for algorithms and data models, they’ll start with code that sets up input data, triggers actions, and checks for expected results. This exploits their tendency to jump right into coding, but channels it into establishing a better understanding the desired behaviors first.

As the developers start to express the external behaviors of the system in code, this will inevitably trigger questions, expose vague requirements and point to missing scenarios. Customers will need to clarify those requirements and make concrete choices. Misunderstandings that might have gone undiscovered until later will be resolved sooner. Ambiguity will wither in the face of cold, hard code!

This customer-involved test-first approach is known as Behavior-Driven Development (BDD), a close cousin of Test-Driven Development (TDD). BDD tests are typically written in Cucumber, a business-user-friendly framework where scenarios are expressed as a series of “Given, When, Then” statements. But the language used is less important than the collaboration between developers and non-developers. Whatever gets them talking and produces tests that can be run by the CI process will work. Turning requirements development into a coding exercise gets developers engaged sooner, tightens the feedback loop and ultimately produces clearer requirements and better definitions of done. And what PM doesn’t love that?

[<< CI4PM Part 3: Prioritizing the code that’s needed, not the code they know](/why-pms-should-love-ci-part-3-prioritizing-the-code-thats-needed-not-the-code-they-know/ "Why PMs should love CI, part 3: Prioritizing the code that’s needed, not the code they know")