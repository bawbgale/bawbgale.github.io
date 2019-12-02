---
id: 104
title: Why PMs should love CI
date: 2017-04-27T14:38:16+00:00
author: ciforpm
layout: revision
guid: http://www.bawbgale.com/41-revision-v1/
permalink: /41-revision-v1/
---
I love my job as a Program Manager at GameHouse. There are a lot of reasons why — good people on my team, working on interesting problems, opportunities to learn. But that&#8217;s been true of most of my past jobs too. When I really dig into why I like my current job so much, one factor stands out: The degree to which we have embraced Continuous Integration, or CI.

Continuous Integration, for the uninitiated, is the software development practice of having every code change trigger an automatic rebuild, retest and &#8212; in some cases &#8212; redeploy of the entire project. It’s an innovation designed to prevent an all-too-common experience in the history of software development: a team of developers diligently works for weeks or months on their individual features, only to find that when they are put together, the result is a train wreck of incompatible parts that destroys the schedule or, worse, doesn’t do what the customer wanted. CI forces individual developers&#8217; changes to be integrated early and often, and automatically tests them to ensure that they do what was intended and don&#8217;t break other features.

So if CI is so great, it should be a no-brainer and everyone should be already doing it, right? Unfortunately no. Achieving continuous integration requires shifting QA resources from manual to automated testing so you can test dozens of times per day without an army of human testers. It requires spending almost as much time writing code to test your product as you do on the product itself, and writing product code in such a way that it can be easily driven by automated tools. It requires setting up infrastructure to quickly build, test and deploy your product dozens of times per day, far more often than is needed to actually ship your finished product. In short, it requires a lot of deliberate investment and disruption, particularly on projects that haven’t operated this way from the start. And the benefit of this investment may be unclear to the unconverted or to stakeholders outside the development team.

So what, you may be asking, does this have to do with Program Management? Automated testing, coding practices and build infrastructure sound like a lot of things that PMs usually aren’t directly involved with. It’s true that most of the details and implementation steps are out of your hands and in the hands of your team’s engineers. And you won’t get very far trying to dictate how they do their jobs or presuming to know what tools they should choose.

But there are very meaningful things that Program Managers and other non-coders can do to make make CI happen, and that’s what I plan to write about in this blog. Much has been written about the programming and DevOps details of implementing CI. My aim is to lay out the benefits of CI in a way that PMs can relate to and utilize to promote it within their organizations. Because it’s just SOOO GOOOD! If you’re gonna blog, it should be on something you are passionate about, and I&#8217;m passionate about this: Why PMs should love CI, and what they can do to make it happen.

<p style="text-align: right;">
  <a title="Why PMs should love CI, part 2: Better software (duh)" href="/why-pms-should-love-ci-part-2-better-software-duh/">CI4PM Part 2: Better software (duh) >></a>
</p>