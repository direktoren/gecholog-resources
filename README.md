# gecholog-resources

## Purpose

This repository provides examples of processors that modify or augment the payload or response for LLM API calls made via the [Gecholog](https://github.com/direktoren/gecholog) LLM Gateway.

## Processors

 - `broker` is a load balancer and failover processor. [Try it...](processors/broker/)
 - `charactercount` is a simple template processor. [Try it...](processors/charactercount/)
 - `mock` is used to mock LLM API by recording a real request. [Try it...](processors/mock/)
 - `regex` is a regex extraction processor (for JSON and Markdown). [Try it...](processors/regex/)