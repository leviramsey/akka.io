---
layout: post
title: Akka Management 1.1.0 Released
author: Arnout Engelen
short: We are happy to announce the 1.1.0 release of Akka Management
category: news
tags: [releases]
---

Dear hakkers,

We're happy to announce the release of Akka Management 1.1.0.

With this minor release, Akka Management now requires Akka 2.6 and Akka HTTP 10.2.

## Changes since 1.0.10

- Fix missing lease log args [#867](https://github.com/akka/akka-management/issues/867) by [@DennisVDB](https://github.com/DennisVDB)
- Add link to SBR lease documentation [#878](https://github.com/akka/akka-management/issues/878) by [@archeuclid](https://github.com/archeuclid)
- Filter pods where the Akka container has 'waiting' status [#891](https://github.com/akka/akka-management/issues/891) by [@raboof](https://github.com/raboof)
- Link to Akka Cloud Platform [#874](https://github.com/akka/akka-management/issues/874) by [@patriknw](https://github.com/patriknw)

### Updates

- Bump Akka HTTP 10.2.0 [#893](https://github.com/akka/akka-management/issues/893) by [@ignasi35](https://github.com/ignasi35)
- Drop scala 2.11 [#856](https://github.com/akka/akka-management/issues/856) by [@ignasi35](https://github.com/ignasi35)
- Bump to Akka 2.6 [#865](https://github.com/akka/akka-management/issues/865) by [@ignasi35](https://github.com/ignasi35)

### Internal changes

- Remove now-unneeded reflection [#894](https://github.com/akka/akka-management/issues/894) by [@raboof](https://github.com/raboof)
- release to sonatype using GH Actions [#889](https://github.com/akka/akka-management/issues/889) by [@octonato](https://github.com/octonato)
- Fix compilation errors on authors.scala [#863](https://github.com/akka/akka-management/issues/863) by [@ignasi35](https://github.com/ignasi35)
- Don't create unnecessary ActorSystems [#864](https://github.com/akka/akka-management/issues/864) by [@ignasi35](https://github.com/ignasi35)
- sbt 1.4.7 (was 1.3.13) [#845](https://github.com/akka/akka-management/issues/845) by [@scala-steward](https://github.com/scala-steward)
- sbt-site 1.4.1 (was 1.4.0) [#804](https://github.com/akka/akka-management/issues/804) by [@scala-steward](https://github.com/scala-steward)
- Use the latest jabba [#892](https://github.com/akka/akka-management/issues/892) by [@raboof](https://github.com/raboof)

## Credits

For this release we had the help of 6 committers – thank you all very much!

```
commits  added  removed
      9    472      233 Ignasi Marimon-Clos
      4    454      162 Arnout Engelen
      1    396      247 Renato Cavalcanti
      1      3        2 archeuclid
      1      4        0 Patrik Nordwall
      1      3        1 Dennis van der Bij
```

The Akka core team is employed by [Lightbend](https://www.lightbend.com/). If you're looking [to take your Akka systems to the next level](https://www.lightbend.com/akka#subscription), let's [set up a time](https://www.lightbend.com/contact) to discuss our enterprise-grade expert support, self-paced education courses, and technology enhancements that help you manage, monitor and secure your Akka systems - from development to production.

Happy hakking!

– The Akka Team
