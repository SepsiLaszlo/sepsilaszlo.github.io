---
layout: post
title: "Reading Recommendations"
date: 2024-05-26 10:04:00 +0200
categories: reading
---

Lately, I have been reading a lot about various topics related to web application development. I thought it would be nice to collect these books, posts, and articles for future reference. Here they are!

[99 Bottles of OOP](https://sandimetz.com/99bottles)

My favorite book on writing Ruby code that is a joy to read. The author describes a test-driven approach to unearth concepts in the domain and create the right abstractions to encapsulate them.

[High Performance PostgreSQL for Rails](https://pragprog.com/titles/aapsql/high-performance-postgresql-for-rails)

This is a great starting point for Rails developers who want to know more about PostgreSQL. It provides practical advice on performance testing, monitoring, and much more.

[Why Uber Engineering Switched from Postgres to MySQL](https://www.uber.com/blog/postgres-to-mysql-migration)

This article compares fundamental architectural differences between PostgreSQL and MySQL. It highlights that PostgreSQL uses an immutable approach to MVCC while MySQL uses a mutable approach. The article also describes the difference in their index implementation. PostgreSQL indexes point directly to the tuples on disk, while MySQL secondary indexes point to the primary indexes, which point to the rows on disk.

[Feral Concurrency Control: An Empirical Investigation of Modern Application Integrity](http://www.bailis.org/papers/feral-sigmod2015.pdf)

A great scientific article on how web applications handle concurrency in large open-source projects. It presents measurements on how application concurrency control fails to guarantee uniqueness on insertion and referential integrity on deletion.

[PostgreSQL Docs - Concurrency Control](https://www.postgresql.org/docs/16/mvcc.html)

PostgreSQL's documentation has amazed me many times with how well written it is. This chapter provides both a conceptual and practical overview of the two most important aspects of concurrency control: isolation and locks.

[Choose a single layer of cleverness](https://dhh.dk/arc/2005_09.html)

DHH's ageless wisdom on keeping the business logic in the object-oriented application layer instead of moving it into the procedural database layer.

[GitLab - Contribute to development](https://docs.gitlab.com/ee/development)

GitLab's handbook is a great resource for building large-scale Rails applications.

[Infinum - Rails Handbook](https://infinum.com/handbook/rails)

Infinum's handbook gives practical tips for Rails developers.

---