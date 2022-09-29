---
layout: post
title: Akka Projections 1.2.0 Released
author: Patrik Nordwall
short: We are happy to announce the 1.2.0 release of Akka Projections
category: news
tags: [releases]
---

Dear hakkers,

We're happy to announce the release of Akka Projections 1.2.0.

## Changes since 1.1.0

- New functionality to pause and resume projections, see [documentation](https://doc.akka.io/docs/akka-projection/current/management.html#pause-and-resume), [#25](https://github.com/akka/akka-projection/issues/25)
- Fix case of table names problem with `OracleDialect` and default settings, [#543](https://github.com/akka/akka-projection/pull/543) by [@ihostage](https://github.com/ihostage)
- Akka 2.6.14 and Jackson 2.11.4, [#541](https://github.com/akka/akka-projection/pull/541)
- Rename `createOffsetTableIfNotExists` and `dropOffsetTableIfExists`, [#546](https://github.com/akka/akka-projection/pull/546)
- Configurable path of dispatcher for JDBC projections [#542](https://github.com/akka/akka-projection/pull/542) by [@ihostage](https://github.com/ihostage)
- Retry management operations if timeout, [#531](https://github.com/akka/akka-projection/issues/531)

This version requires creating the new `projection_management` table, which is used for storing the pause/resume state of the projections. This table is required even if the new pause/resume functionality isn't used. For rolling update you can create the table while the 1.1.0 version is still running. No other database maintenance is required since all existing database elements are compatible.

See schema definitions:

* JDBC: https://doc.akka.io/docs/akka-projection/current/jdbc.html#schema
* Slick: https://doc.akka.io/docs/akka-projection/current/slick.html#schema
* Cassandra: https://doc.akka.io/docs/akka-projection/current/cassandra.html#schema

## Credits

For this release we had the help of 4 committers – thank you all very much!

```
commits  added  removed
      5    486      140 Renato Cavalcanti
      4   3207      137 Patrik Nordwall
      2     39       19 Sergey Morgunov
      1      1        1 Sean Glover
```

The Akka core team is employed by [Lightbend](https://www.lightbend.com/). If you're looking [to take your Akka systems to the next level](https://www.lightbend.com/akka#subscription), let's [set up a time](https://www.lightbend.com/contact) to discuss our enterprise-grade expert support, self-paced education courses, and technology enhancements that help you manage, monitor and secure your Akka systems - from development to production.

Happy hakking!

– The Akka Team
