---
title: Distributed Systems Notes - Failure Modes
date: 2021-05-08 15:00:00 +0530
categories: [Distributed Systems, Failure & Time]
tags: [distributed-systems, basics, notes, failure-modes]
---

Hello! 

The participants of a distributed system can fail in various ways. In order to build a fault tolerant distributed system, the designers first need to understand the various ways in which their system could fail. **A fault model specifies the kinds of failures that a system may exhibit which can then be used to inject fault tolerance into the system**. In this brief blog post, we will be discussing the various failure modes(fault models) in distributed systems. 

## Definitions
At a high level, there are four types of failure modes in distributed systems:

1. Crash faults
2. Omission faults
3. Performance(Timing) faults
4. Byzantine faults

### Crash Faults
In a crash fault, the participant crashes and stops responding to messages. Usually there is no way for a healthy participant to tell if another participant has crashed. The fault model in which it's possible for the healthy participant to detect the failure of another participant is called *Fail-stop faults*. Hence, Fail-stop faults are a subset of the crash faults.

### Omission Faults
In the omission faults model, a message is lost (omission of message). A faulty participant could fail to send or receive a message.

### Performance/Timing Faults
In this fault model, a faulty participant could respond slowly i.e. even though it's responding with the correct message, it's takes a lot of time.

### Byzantine Faults
In the Byzantine fault model, a participant could behave in an arbitrary manner which may be malicious as well. The faulty participant could also forge messages from other servers i.e. pretend to be another participant in the system. A subclass of Byzantine faults is *Authentication Detectable Byzantine faults* (Referred as AD Byzantine in later sections) in which the forging of messages is not possible.

## Hierarchy
The discussed fault models form a hierarchy as depicted in the below figure.

![Hierarchy of Failure Modes](/assets/img/distributed-systems-notes/distributed-systems-failure-modes.png)

At the center of the ring is the *Fail-Stop* failure mode superceded by the *Crash* failure mode. Crash faults can be understood as a special case of omission fault where every message is lost to & from the unhealthy participant.

Omission fault is a special case of a Performance fault where the unhealthy participant responds infinitely slowly to a request.

Similarly, Performance fault could be considered as a special case of Byzantine fault where an unhealthy (or malicious) participant could purposefully delay the response of a request.

We can deduce that if our system is tolerant towards a failure mode, it is also tolerant towards the modes within it's ring. Eg. If a system can tolerate Performance faults, it can also tolerate Omission and Crash faults.

Moreover, the complexity of tolerating a failure mode increases as we go farther from the center. For eg. building systems that tolerate Byzantine failures has higher complexity (and usually lower performance) than a system that only handles Crash faults.

## Conclusion
In this post, we discussed the various fault models in distributed systems and their relationship with each other.

Thanks for reading!
