---
layout: post
title: "On Runtime Monitoring Tools"
description: "a way to classify runtime monitoring tools"
category: "Security"
tags: ["Security"]
tagline: "there categories of existing dynamic monitoring schemes"
---

    Runtime monitoring tools make use of runtime information to
    detect bugs that are difficult to detect statically. Existing
    dynamic
    monitoring schemes fall into three categories:
    programming-rule based
    (PRB), invariant-rule-based (IRB), and execution-statistics based
    (ESB).

## Programming Rule Based (PRB)

PRB checks for violations of **programming language specifications**,
**programming paradigms**, or **software development specifications**.
For example ``array index cannot exceed the array bounds''
and ``concurrent accesses to a shared variable should be
synchronized''
are the kinds of rules PRB checkers use to detect bugs.

## Invariant Rule Based (IRB)

IRB extracts invariant rules (e.g. PC and value invariants) from
**successful runs** or **multiple periods of a single long-running
execution**, and then uses these rules to check for violations in a
later execution or later periods of an execution in a long-running
job. Valueinvariance maintains, for each variable, a set of values
that the variable hes held. When a value not in the set is found at
runtime, it is recorded as an invariant violation. Program Counter
(PC) invariance maintains for each memory location a set of program
counter values(i.e. program locations) that access the variable. When
a datum is accessed by a program location that has not previously
accessed it, the access is recorded as an anomalous access.

## Execution Statistics Based (ESB)

ESB extracts **statistics that characterize the program's runtime
behavior**, and uses statistical analysis to detect bugs on the fly.

{% include JB/setup %}
