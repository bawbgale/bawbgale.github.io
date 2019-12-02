---
id: 64
title: 'Why PMs should love CI, part 3: Prioritizing the code that&#8217;s needed, not the code they know'
date: 2017-01-01T18:20:05+00:00
author: Bob Gale
layout: revision
guid: http://www.bawbgale.com/58-autosave-v1/
permalink: /58-autosave-v1/
---
_Continuous Integration is often presented as a “dev team” thing — not something us PMs should worry our pretty little heads about. But CI is good for PMs too, and is more successful when PMs buy in. This is a series about why PMs should care about CI and what they can do to help foster it in their teams._

Have you ever faced this situation? You’ve painstakingly prioritized the backlog with your stakeholders. Maybe you’ve even had to balance the needs of several stakeholders, horse-trading to come up with a priority order that not only reflects business priorities but keeps the peace among competing interests.

But when you bring the backlog to your development team, you find that most of the features at the top of the list can only be done by the same few overloaded developers. Maybe it’s not even that the features can’t be done by others, it’s just that those overworked few wrote it from scratch, know it inside and out, and an air of nervous apprehension descends on the team at the suggestion of someone else taking it on. So the other developers who have available cycles skip down the list and pull up lower priority features while the more important ones wait for some rock star attention.

Having a robust Continuous Integration process can help avoid this situation. The foundation of such a process is a thorough set of automated tests that run every time code changes are committed. These tests provide a roadmap for how each part of the product is expected to behave — given these inputs, when that feature is triggered, then this is the correct result. Studying these tests is a great way for a developer to learn how a feature works now, how it should respond in different situations, and how different parts interact. It also makes it safer for the developer to dive in and start changing things because automated tests provide feedback about the results (possibly unexpected ones) and catch any inadvertent breakage. &#8220;Oh I see, when I change this here, that changes over there.&#8221; This can go a long way toward mitigating a developers&#8217; reluctance to take on changes in code they are less familiar with.

Of course, developers’ task choices are not always just a matter of familiarity with the code. Sometimes the feature is truly outside their capabilities. It may be mostly user-interface work, and the developer with time available is primarily a database expert. Specialization is real and often a good thing, and occasional mismatches between priorities and resources are inevitable. But if it happens too often, it may be a sign that you need to acquire more resources, either temporarily through contractors or borrowing from other teams, or permanently through hiring. But here too, a good automated test suite can be invaluable in helping new developers get up to speed faster.

So if your team is considering implementing Continuous Integration, keep these benefits in mind. Building a good automated test suite takes time and attention, and the team will need your buy in to prioritize it. But it will pay off for you in a more flexible, adaptable team who can prioritize the code that&#8217;s needed, not just the code that they know.

[<< CI4PM Part 2: Better software (duh)](http://www.bawbgale.com/why-pms-should-love-ci-part-2-better-software-duh/ "Why PMs should love CI, part 2: Better software (duh)") | [CI4PM Part 4: Coding your way to clearer requirements >>](http://www.bawbgale.com/why-pms-should-love-ci-part-4-coding-your-way-to-clearer-requirements/ "Why PMs should love CI, part 4: Coding your way to clearer requirements")