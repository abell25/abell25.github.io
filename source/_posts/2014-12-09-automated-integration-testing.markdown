---
layout: post
title: "automated integration testing"
date: 2014-12-09 16:05:39 -0600
comments: true
categories: 
---

I've been thinking lately about how integration testing could be automated at my job.  When I come upon a new service I haven't worked on before I often do not know how it's supposed to work, let alone figure out how to construct clever robust tests to test it.  Once I become fluent in how something works, though, I am able to construct clever integration tests.  I want to be able to write down my clever tests and have those tests be able to run for anybody.

I feel like System administration had a similar issue and they solved it.  The hand-coded configuration files and manually set-up servers are being replaced by DevOps where code as infrastructure both documents the actions that were taken and "embody" all the tricks that seasoned SysAdmins would have commited to these actions.  I am wondering if integration testing could also benefit by a system where we can define integration tests as code and this knowledge will not be lost.

The idea I am starting to develop is that an end-to-end system test is inevitably composed of smaller integration tests which in turn may be decomposed into smaller tests.  Some steps must wait for other steps to complete.  This seems like a natural fit for hierachal task network planning.  A framework could be provided, much like a unit-testing framework, where the developer could specify how a test breaks down into smaller tests, what it's supposed to do and how to tell if it is
successful.  We could have a web interface where you specify what high-level tests should be ran and how often along with hooks into CI tools such as TeamCity and Jenkins.  They would run on a beta environment, essentially simulating actual use-cases.

I feel that major benefits could be derived from these tests.  One of the major ones is that the people who understand a part of the system can design the test for that part.  People who understand the system at a higher level can use this test as a step in their larger test with-out having to know all the details of this particular sub-system.  This could translate to tests written by the people most competent to write them which means more bugs get caught.  Another benefit is that
people who come to a system for the first time could look at the integration tests and get a feel for how things are supposed to work.  And if they break a behaviour due to their ignorance, the test will let them know it!



