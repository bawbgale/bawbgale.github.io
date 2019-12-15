---
id: 55
title: 'Why PMs should love CI, part 2: Better software (duh)'
date: 2015-02-07T21:59:52-08:00
author: Bob Gale
layout: post
guid: http://www.bawbgale.com/?p=55
permalink: /why-pms-should-love-ci-part-2-better-software-duh/
categories:
  - Uncategorized
tags:
  - agile development
  - CI for PM
  - continuous integration
  - program management
---
_Continuous Integration is often presented as a DevOps concern. This is a series about the benefits for PMs and what they can do to help foster the process in their teams. The first benefit I’ll cover is the same one enjoyed by all stakeholders: better software._

We don’t have a bug tracking system.

This fact dawned on me recently as we were bringing a new developer up to speed on our systems and processes. “What do you use for bug tracking?” he asked. The question caught me off guard. After a few seconds, I realized the answer: “nothing.”

I’m not saying we never have bugs (we do) or that we don’t have a task tracking system (we use [Trello](http://trello.com)). But we find that bugs occur rarely enough that when they do happen, we can deal with them quickly. We post it in [FlowDock](http://flowdock.com) (our team chat app) and someone jumps on it. We never build up a backlog of defects that need to be tracked and prioritized. This is not a small project either. We have thousands of lines of code spanning more than a dozen repos maintained by 10 developers.

How is this possible? Certainly some credit goes to good developers following good coding practices. But a big reason is Continuous Integration, for several reasons.

For starters, building our development environment around CI means that we don’t have to remember to run tests (or assign QA resources to do it). It happens automatically every time we commit new code to our repos. This gives us a frequent check that new features we are developing aren’t inadvertently breaking something else. It frees the developer to focus on a particular part of the code, safe in the knowledge that the test suite is tirelessly checking all the other parts for unintended side effects.

CI also speeds the process of fixing bugs when they do occur by making us more confident that our fix doesn’t break something else. There’s nothing more heartbreaking than when a diligent rush to fix something creates other bugs. Automation is particularly important in such times because a production bug shines an uncomfortable spotlight on the development team, precisely when it’s most important that they keep their cool and work the problem. Once again, the automated test suite has our backs. It never gets nervous, impatient or distracted. It lets us zero in on the primary problem while it dutifully checks for collateral damage.

CI also makes the deployment process safer and faster by encouraging automation of all steps. In a fully built-out CI environment, the path to production always goes through the CI system. We don’t push code to the production boxes; we push it into a special branch of the repo, which is watched by the CI system. This triggers CI to run the code through all of our tests and, if it passes, run a set of scripts that pushes the code to production. These scripts contain all the steps necessary to configure servers and deploy code, so we avoid bugs caused by mistakes in manual configuration and deployment steps. It also means we don’t need to wake up IT people if we need to push an urgent bug fix in the middle of the night.

Naturally, this approach only works if we are diligent about writing tests and deployment scripts. It requires commitment up front and steady maintenance. We help that along by incorporating test writing into the bug fixing process.

The first step in any bug triage process is to identify how to reproduce the bug. But instead of just describing the steps, we write a new automated test that triggers the steps and checks for the expected outcome. This allows us to quickly and repeatedly trigger the repro steps while we are fixing the bug. We know we’re done when this test and all other existing tests pass. The payoff is not only a fixed bug but an addition to our test suite that prevents it from recurring.

Obviously, better software and fewer bugs benefits everyone. But PMs will particularly enjoy being able to spend more time on moving the product forward and less time managing a backlog of defects, or agonizing over how much to prioritize bug fixes next sprint vs. developing new features. It’s just one way that good CI makes happier PMs.

[<< CI4PM Part 1: Why PMs should love CI](/why-pms-should-love-ci/ "Why PMs should love CI") | [CI4PM Part 3: Prioritizing the code that’s needed, not the code they know >>](/why-pms-should-love-ci-part-3-prioritizing-the-code-thats-needed-not-the-code-they-know/ "Why PMs should love CI, part 3: Prioritizing the code that’s needed, not the code they know")