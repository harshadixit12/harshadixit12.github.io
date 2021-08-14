---
layout: post
title: Observability
date: 2021-08-14 20:00:00
summary: Observability, the three pillars and what you can do with them
categories:
thumbnail:
tags:
  - Observability
  - Logs
  - Distributed tracing
---

**Table of Contents**

- TOC
  {:toc}

// Outline

WHat is observability, why it is important, and the venn diagram

Pillars, advantages, disadvantages of each
Logs: levels, operation ID, metadata, instance ID, timestamp
Metrics:
Events: QL, dashboards
Tracing

Newrelic

# Introduction

How do you make sure your service is operating correctly?
First step would be to have tests. But what happens after yout service is deployed to servers? This is the next step - using a monitoring solution helps - to test for functional correctness, uptime and performance regularly to ensure customers are happy. But neither of these help you find the root cause of the issue if your service fails.

Software systems are complex by nature, and can fail for any number of reasons. Failure cannot be prevented, therefore in every step of building software, possibility of failure needs to be aknowledged.
When something does eventually fail, ease of debuggig is crucial, otherwise the system will be hard to maintain.
This is where observability comes in.

Monitoring - use cases (should provide high level metrics, impact of failure, effect of fix deployed), blackbox vs whitebox

Systems need to be maintainable, understandable, debuggable, and evolvable.

Tests - best effort verification of correctness.
Monitoring - Detection of known failure modes - degraded performance, downtime, functional correctness
Observability - Helps you identify unknown failure modes.

# The three pillars of observability

Logs, metrics and traces together make up the three pillars of observability. Note that just having them does not make a system observable.

Logs: record the key events that happened highest granularity - payload + timestamp. Easy to generate, have context. Have performance penalty. Ingestion and storage needs. Increase proportionally to traffic. Elasticsearch is difficult to maintain. Sampling - needs additional compute

Metrics: Optimised for storage, doesnt increase costs with increase in traffic, low granularity - system level. Suited to trigger alerts, easier to query and crunch.

Traces:
Traces tell you the story - the flow of a request through the entire system. Takes effort to use in existing systems and carries overhead. Also needs sampling.

You also need to understand the hardware demands of the application - whether it is consuming too much CPU / memory etc, and identify problems before they reach production.
Use tools like memwatch, gc-profiler, flame graphs.
Along with these, there are more tools which help - exception tracking (sentry).
