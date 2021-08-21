---
layout: post
title: Observability
date: 2021-08-14 20:00:00
summary: Observability, the three pillars and what you can do with them
categories: ['Technology']
thumbnail:
tags:
 - Observability
 - Logs
 - Distributed tracing
---
 
## Introduction
 
Ensuring that your customers are able to seamlessly use your web application is just as important as building it.
This is usually done by having tests and setting up monitoring. 
### Tests
Tests help you ensure that your code is correct, and any new changes don't unintentionally break existing features. Once your code is deployed however, tests do not help.
In other words, tests help in best effort verification of correctness.
### Monitoring
Monitoring tools help you ensure optimal user experience in the following ways:
 1. _Performance of your application_: 
  You can get high level metrics, ensure your application responds in time, and trigger alerts otherwise.
 2. _Functional correctness_: 
 You can define tests which run regularly to ensure your application is returning correct responses.
 3. _Uptime_: 
 You can ensure your application is available at and trigger alerts if not. 
 In other words, monitoring helps in detection of known failure modes - degraded performance, downtime, functional correctness
 [Check out Postman Monitors](https://www.postman.com/api-monitor/) 
 
While testing and monitoring help you find that something is wrong, they are not sufficient to answer why. 
Software systems are complex by nature, and can fail for any number of reasons. Failure cannot be prevented, therefore in every step of building software, possibility of failure needs to be acknowledged. 
In addition to being maintainable, understandable and evolvable, software needs to be debuggable. This becomes crucial when something eventually breaks. 
This is where observability comes in.
 
Observability helps you identify what is wrong with your system, as quickly as possible. 
 
__In other words, observability helps you identify unknown failure modes.__
 
## The three pillars of observability
 
Observability is about collecting useful data that can help you identify faults. This data can be categorised into three verticals - Logs, metrics and traces (affectionately called the three pillars of observability). Each have their own advantages and disadvantages, and should be used based on requirement. 
 
### Logs: 
Logs contain the most granular level of data that helps in debugging. They can record the occurrence of an event, along with other contextual information such as timestamp, the service name, etc.
Typically, services utilise a logger such as winston, and define a custom format that records all useful metadata along with the log. 
 
Logs are easy to generate and understand, but have performance penalties ("Logging is not free"). 
While generating logs is easy, ingestion and storage is not. This is because the volume of logs generated increases proportionally to traffic. Also, the amount of value offered by logs may not justify the cost of storing them. To mitigate this sampling can be used, but this gives an incomplete picture. 
 
A better approach to logging would be to ensure every log is actionable. 
 
### Metrics: 
Metrics help you understand the health of your system at a high level. 
 
Unlike logs, they are optimised for storage, and don't incur increased costs with increase in traffic. Also, they are suitable for building dashboards, and other number-crunching. Metrics typically don't contain context along with each measurement. 
However, they have low granularity and only offer system level insights. 
 
Metrics are suited for measuring performance, and to trigger alerts. 
 
 
### Traces: 
While logs give you hints, and metrics tell you the theme, traces tell you the whole story. 
Traces track flow of a request through the entire system, and measure the performance of each part of the system involved. They are especially useful in microservices where each request travels through multiple services - where logs and metrics are not helpful. 
 
However, traces require effort to integrate with existing systems, and have a performance overhead. Due to the sheer amount of data collected, sampling might be required. Based on the sampling technique, there may be loss of data, or even additional cost in transferring it over a network. 
 
## Some more useful tools
While logs, metrics and traces give some amount of visibility into the application, they don’t cover an important aspect - the hardware demands of the application. 
It is crucial to understand the resource consumption pattern of your application and understand the cost of each new feature added in terms of these resources. Tools like memwatch, gc-profiler, flame graphs can be used for this on Node JS. 
 
Sometimes, if there is a fatal exception and your application crashes, logs may not be present. Exception tracking tools such as sentry can be used for this purpose. 
 
Events: Some observability platforms like newrelic let you record and query events. They have the advantages of metrics and logs as they can be queried, and contain some amount of contextual information. 
 
## Summary
Software applications need to be understandable, maintainable, evolvable and debuggable. They are complex by nature, and ease of identifying faults when something inevitably goes wrong is crucial. 
 
While tests and monitoring look for known failure states, the three pillars of observability - logs, metrics and traces can assist in identifying unknown failure states. 
 
Having logs, metrics and traces alone does not make a system observable. Often, asking yourself questions like "How do I know if this breaks in production?" during development goes a long way in ensuring it. 
 
### Relevant Links: 
[Distributed Systems Observability](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/)  
[Newrelic](https://newrelic.com/)   
[Apache SkyWalking](https://skywalking.apache.org)  
[Sentry](https://sentry.io)  
[Memwatch](https://www.npmjs.com/package/memwatch)  
[GC profiler](https://www.npmjs.com/package/gc-profiler)  
[Flame graphs](https://github.com/brendangregg/FlameGraph)  