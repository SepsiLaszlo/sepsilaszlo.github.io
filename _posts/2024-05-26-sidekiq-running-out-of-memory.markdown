---
layout: post
title: "Sidekiq running out of memory"
date: 2024-05-26 10:04:00 +0200
categories: ruby
---

During the monitoring of a Ruby on Rails project, I noticed that the Sidekiq process was consuming more memory every day. At this rate, our VM would run out of memory in a week! So, I started investigating.

The ever-growing memory usage for a long-running process is a typical symptom of a memory leak. Since Ruby lifts the burden of memory management from developers, I couldn't imagine how this issue could occur. I began researching how Ruby manages memory.

I found a great talk, [Building a Compacting GC for MRI by Aaron Patterson](https://youtu.be/8Q7M513vewk?si=64OHAbsI_rPtlbY2). Aaron explores Ruby's memory management, focusing on Copy on Write, building a compacting GC, and memory inspection tools. After listening to this talk, I knew more about how Ruby uses memory, but it was still unclear what could cause the issue with our Sidekiq process.

Searching for more talks on Ruby memory management, I found [The Hitchhiker's Guide to Ruby GC by Eric Weinstein](https://youtu.be/NnqId_OvUU4?si=ZANFxF-f8d0nK8lC). This talk gave a really good overview and presented GC configuration options that we can tweak to achieve better memory usage. It also described the Mark and Sweep mechanism and how keeping track of generations can help garbage collectors based on the fact that _"most objects die young"_. But the most important insight for me was:

> When there's no more free memory, Ruby marks all active objects, then combines (sweeps) inactive objects.

This means that Ruby won't run the GC procedure until there is any free memory left. The reason is that the GC procedure is really costly, since it stops everything while it runs and can take multiple seconds to complete.

I wanted to double-check this information. AppSignal's blog had a thorough post about [Practical Garbage Collection Tuning in Ruby](https://blog.appsignal.com/2021/11/17/practical-garbage-collection-tuning-in-ruby.html), which confirmed my suspicion.

> Contrary to a common assumption that GC runs happen at fixed intervals, GC runs are triggered when Ruby starts running out of memory space. Minor GC happens when Ruby runs out of free_slots.

This post exactly described how my previous assumption about Ruby's GC was wrong. Our application runs in containers managed by Docker Compose. Docker Compose doesn't limit the memory for the containers by default, so each of them could potentially use up all the free memory on the host machine. This is what happened since Ruby's garbage collector doesn’t run periodically!

The fix was easy. I just set some reasonable memory limits for the service.

```yaml
# docker-compose.yml
version: "3"
services:
  sidekiq:
    build: .
    restart: always
    command: "bundle exec sidekiq"
    deploy:
      resources:
        limits:
          memory: 150M
```
