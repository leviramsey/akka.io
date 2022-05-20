---
layout: post
title: Akka gRPC 2.1 performance improvement
author: Johannes Rudolph
short: How Akka gRPC went from nearly last to #1 in performance 
category: news
tags: [announcement,grpc]
canonical_url: https://www.lightbend.com/blog/akka-grpc-update-delivers-1200-percent-performance-improvement
---

## How Akka gRPC went from nearly last to #1 in performance

![social image](https://downloads.lightbend.com/website/blog-images/akka_grpc_social_v3.png)

In our recent announcement of [version 2.1.0 of Akka gRPC](https://discuss.lightbend.com/t/akka-grpc-2-1-0-released/8702#highlights-1), the Akka team was also pleased to see very positive results from a community benchmark that measures performance and latency of over 30 gRPC libraries. Akka gRPC (named “Scala_Akka” in the benchmark) went from 29th place to 1st in just three months, delivering an impressive 1231% increase in processing throughput (req/s) and a 93% reduction in average latency per request.

So what is Akka gRPC, and how did we manage to make it the fastest solution tested? We sat down with Johannes Rudolph ([@virtualvoid](https://twitter.com/virtualvoid)), Senior Engineer on the Akka team at Lightbend, to learn more about this story...

[SEE THE BENCHMARKS ON GITHUB](https://github.com/LesnyRumcajs/grpc_bench/wiki/2021-08-30-bench-results)

## Q: What is Akka gRPC and where does it show up in Lightbend products?
Akka gRPC is our OSS gRPC library that builds on top of Akka and Akka HTTP.  A few years ago we started looking into gRPC over other protocols for microservice communication because of its well-defined interface and solid tooling for different programming languages–i.e. not just JVM languages.

[Kalix](https://kalix.io) (Called Akka Serverless at the time this article was written) utilizes gRPC throughout the stack – [user services and entities](https://docs.kalix.io/services/programming-model.html) are declared in a protobuf definition and can be accessed via gRPC. Likewise, the internal communication of the user code with the infrastructure is handled via gRPC.

Streaming is a first-class concept for gRPC service endpoints which fits our stack well–check out this [documentation](https://doc.akka.io/docs/akka-grpc/current/whygrpc.html) to learn more about when gRPC is a good alternative to a traditional HTTP API for communication between microservices.

## Q: Three months ago, Akka gRPC was ranked in the bottom 10% for performance–what happened since then?
In May 2021, we were preparing the Akka Serverless Beta launch and worked on optimizations in different parts of the system. When testing basic RPC calls to establish a baseline, one thing that turned up in profiling was gRPC communication in our infrastructure. At about the same time, the previous run of [grpc_bench](https://github.com/LesnyRumcajs/grpc_bench/wiki/2021-08-30-bench-results) was published and found our attention after some [public commentary](https://twitter.com/alexelcu/status/1390986472866648066?s=20) about the relatively poor performance of Scala_Akka.

### May 2021
Akka gRPC benchmark (Scala_Akka) for May 2021

![May results](https://downloads.typesafe.com./website/blog-images/2021-05-20-bench-results.png)

Source: https://github.com/LesnyRumcajs/grpc_bench/wiki/2021-05-20-bench-results

### August 2021
Akka gRPC benchmark (Scala_Akka) for August 2021

![August results](https://downloads.typesafe.com/website/blog-images/2021-08-30-bench-results.png)

Source: https://github.com/LesnyRumcajs/grpc_bench/wiki/2021-08-30-bench-results

The results from [grpc_bench](https://github.com/LesnyRumcajs/grpc_bench/wiki/2021-08-30-bench-results) are generated using [ghz](https://github.com/bojand/ghz), a Go-based gRPC benchmarking tool, as the underlying client. It runs a simple RPC scenario without streaming–1000 requests concurrently, spread over 50 persistent connections. It measures the throughput and latency under various conditions, including performance over 1-3 CPUs.

We had not optimized Akka gRPC in particular before–however, we decided to spend several weeks going through our stack to find and eliminate bottlenecks. We made optimizations in Akka itself, Akka HTTP, and Akka gRPC to realize all the potential gains. The full set of improvements is available starting with Akka [2.6.16](https://akka.io/blog/news/2021/08/19/akka-2.6.16-released), Akka HTTP [10.2.5](https://akka.io/blog/news/2021/07/27/akka-http-10.2.5-released), and Akka gRPC [2.1.0](https://akka.io/blog/news/2021/08/31/akka-grpc-2.1.0-released).


| May 2021 - 2 CPU server |	August 2021 - 2 CPU server |
|-------------------------|----------------------------|
| Ranking: 29 out of 33	| Ranking: 1 out of 33 |
| Requests per second: 6117	| Requests per second: 81435 |
| Average latency: 163 milliseconds&nbsp;| Average latency: 12 milliseconds |

These metrics result in the following:

* 1231% increase in processing speed
* 93% reduction in average latency
* 20% higher throughput than second ranked

## Q: What should folks know about benchmarks and how they work?
In our opinion, this test measures the baseline performance in the simplest case of gRPC usage well. In a modern microservice architecture, and in many cases serverless architectures as well, most requests are small and do not require streaming. Also, the set of machines talking to each other are relatively stable, so that persistent connections are common.

One potential issue is that in our tests the benchmarking tool, ghz, can saturate the testing machine with its own resource usage. We found that under top performance (around 60-80k RPS) the benchmarking tool itself started to become the bottleneck. As always, take note of the order of magnitude but don’t interpret too much into the concrete numbers of the benchmark result.

If you want to test this on your own machine, grpc_bench uses a Docker-based workflow that is orchestrated using a few scripts. It is quite simple to build and execute the benchmarks you care about on your own – see its [README](https://github.com/LesnyRumcajs/grpc_bench/blob/master/README.md) for more information.

## Q: What specifically did the Akka team do to enable this massive increase in performance, and how was it driven by Akka Serverless?
We noticed that our [gRPC usage in Akka Serverless](https://docs.kalix.io/java/writing-grpc-descriptors-protobuf.html) (and also in general) are more often of the simpler request/response kind than actually using the streaming capabilities of gRPC. However, the HTTP/2 implementation of Akka HTTP was built around the streaming capabilities. A stream has a significant cost when it is initially set up which limits the achievable throughput for one-off requests.

So we build a fast path for simple requests which required changes throughout the stack:

Akka gRPC needs to process strict requests (non-streaming) eagerly and generate strict responses when possible.
Akka HTTP needs to process those strict responses and also tries to aggregate requests (before falling back to streaming mode).
Akka Stream / IO needs to aggregate outgoing HTTP/2 frames before sending them out to avoid system calls.
Another significant improvement was found by avoiding parsing of HTTP headers when they have been seen before on the same connection. For a list of these and all the other, smaller improvements, you can see [the release notes](https://discuss.lightbend.com/t/akka-grpc-2-1-0-released/8702#highlights-1).

## Q: What's a good example/code experience for utilizing the new features and performance improvements of Akka gRPC?
All users of gRPC (and Akka Serverless) will benefit from the optimizations but simple request/response calls especially will now have a much smaller footprint and leave more processing power to the application. This is important in particular for applications that require processing times shorter than 1ms or need to scale to throughputs of more than thousands of requests per second per core.

Improvements made in Akka HTTP and Akka itself, are not specific to gRPC, so other uses of HTTP/2 or the TCP streams in Akka Streams will also benefit from the improvements. We look forward to seeing how this works out for our users, so please join the discussion!

