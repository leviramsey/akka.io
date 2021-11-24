---
layout: post
title: Alpakka towards 1.0
author: Enno Runne
short: Making Reactive Integrations ready for a long life 
category: news
tags: [alpakka,streams]
---

Dear hakkers,

Akka's satellite project to collect Reactive Enterprise Integrations in a library - [Alpakka](https://doc.akka.io/docs/alpakka/current/) - just turned [2 years](https://akka.io/blog/article/2016/08/23/intro-alpakka.html) the other day. 

In these 2 years Alpakka has grown from a handful of included technologies to a large set of Reactive Integrations built on Akka Streams, delivered in more than 30 modules.

These modules are developed and have been extended by Akka Streams users, either coding to solve a company's integration needs, or out of interest and the fun of building interesting software.

As the community's efforts show the interest in creating an ecosystem for Reactive Integrations, Lightbend earlier this year decided to [ramp up Alpakka](https://akka.io/blog/news/2018/05/02/alpakka-team) with a dedicated team of paid developers to support the growing Alpakka community.

## Module structure

During these two years, the Alpakka modules have tried out different approaches and code layouts. A common API structure has evolved that fits most modules we've seen to date. 

The [Alpakka team](https://akka.io/team/) at [Lightbend](https://www.lightbend.com/) has put together some [contributor advice](https://github.com/akka/alpakka/blob/master/contributor-advice.md) and recently implemented a [reference connector implementation](https://doc.akka.io/docs/alpakka/current/reference.html) to document and illustrate the recommended pattern for all Alpakka contributors.

We've now started to change existing modules into this recommended structure, as we believe it will make it easier for all contributors to find their way through more than one module. 

## Binary compatibility

Akka and its ecosystem has a strong track record on keeping [backwards binary compatiblity](https://doc.akka.io/docs/akka/current/common/binary-compatibility-rules.html) across versions of all modules. When binary compatibility is provided, the users are able to upgrade to the latest minor version, without even recompiling the code.

As Alpakka has been evolving quite rapidly, there have not been any guarantees to keep the APIs stable - even though most contributions tried to keep it so. Promising compatibility would have hindered many great additions.

But it is time to change that! When considering the new recommended structure, we've taken care to make sure Alpakka's APIs can evolve more easily in a binary compatible way. In most cases this will mean a module's API will change, when it is adapted to the recommended structure. After that, breaking changes will be kept for major releases, and minor releases should be a reliable replacement.

## The Alpakka is climbing up the road to Alpakka 1.0

When all modules adhere to the recommended structure, and public APIs are considered possible to evolve without breaking existing code, the Alpakka project will release a version 1.0!

For keeping track of everything that takes us up that hill, we've collected [Alpakka 1.0 issues at Github](https://github.com/akka/alpakka/projects/1).

How can **you** help? The best thing you can do is to use Alpakka's Reactive Integrations in anger, and report when something is surprising or inconsistent. And, yes, the Alpakka needs many shepherds, so join the community and help us up that hill to Alpakka 1.0 by working on the listed issues.

Happy hakking!

– The Alpakka Team

