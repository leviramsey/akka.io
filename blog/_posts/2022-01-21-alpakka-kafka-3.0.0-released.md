---
layout: post
title: Alpakka Kafka 3.0.0 Released
author: Enno Runne
short: Alpakka Kafka 3.0.0 has been released!
category: news
tags: [releases]
---

Dear hAkkers,

The Alpakka contributors are happy to announce the release of Alpakka Kafka 3.0.0 final.

Alpakka Kafka 3.0 features the upgrade to the current Kafka client library 3.0.0 which is backward compatible with Kafka brokers of older versions. Alpakka Kafka 3.0 is published for Scala 2.13.

Alpakka Kafka does not change its APIs compared to the 2.1 series, even previously deprecated APIs are kept. The major version upgrade is motivated by the Kafka clients upgrade and some changes to its dependencies.

See the release history on [GitHub releases](https://github.com/akka/alpakka-kafka/releases).

## Notable changes since 2.1.1

* [Kafka client 3.0.0](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients/3.0.0) does not require Jackson anymore
* With being Scala 2.13 only the Scala Collection Compatibility library was removed

## Upgrading to Alpakka Kafka 3.0

We recommend you ensure to be on current versions of the Alpakka Kafka dependencies before upgrading:
* Scala 2.13.8
* Akka 2.6.18

Happy hakking!

– The Akka & Alpakka Team
