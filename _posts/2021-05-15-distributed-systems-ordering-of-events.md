---
title: Distributed Systems Notes - Ordering of Events
date: 2021-05-30 15:00:00 +0530
categories: [Distributed Systems, Failure & Time]
tags: [distributed-systems, basics, notes, ordering-events]
math: true
---

Hello! 

In this blog post, we will be discussing the various techniques of ordering events in distributed systems. We will briefly discuss *Lamport Clocks* and it's generalized form, the *Vector clocks*. This post is my personal notes on this topic as collected from various resources. Please check the references for further readings.

## Introduction
It's important to order events in a distributed system in order to reason about it and detect bugs. If an event *A* is ordered earlier than *B* according to some clock (we'll discuss which ones), we say that *A happens-before B*. Since it's impossible sometimes to say that one event occurred first, the *happens-before* relation is a [partial order](https://en.wikipedia.org/wiki/Partially_ordered_set#Formal_definition)[2].

Assume *A happens-before B* which is denoted by $$ A \to B $$, we can conclude two things[1]:
1. *A* could have caused *B*
2. *B* could not have caused *A*

The above two points could be valuable in designing and debugging systems.

## Defining our System
The system is a collection of processes each consisting of a sequence of events. Some of these events are internal to the process while others are message send and receive events[1]. We use *Lamport diagrams* to denote these events. It is a space-time diagram where the horizontal direction represents space and the vertical direction represents time [2]. Later times are indicated lower in the diagram.

![Lamports Diagram](/assets/img/distributed-systems-notes/lamports-diagram.png)

The above diagram shows a simple Lamports diagram of a single process with events _X_, _Y_ and _Z_.

### Happens-Before
Formally, we define that *A happens-before B* i.e. $$ A \to B $$ if any one of the following is true[2]:

1. A & B occur on the same process line with B after A
2. A is a message send event and B is the corresponding message receive event
3. If $$ A \to C $$ and $$ C \to B $$ (transivity)

As noted earlier, *happens-before* is a partial order. If two distinct events _A_ and _B_ are such that $$ A \not\to B $$ and $$ B \not\to A $$, then they are said to be _concurrent_ [2]. We denote this relation by $$ A \parallel B $$.

![Happens Before example](/assets/img/distributed-systems-notes/happens-before-example-lamport-diagram.png)

For example, in the above diagram we can conclude the following:
1. $$ C \to E $$ (rule 1)
2. $$ B \to C $$ (rule 2)
3. $$ B \to G $$, since $$ B \to C $$, $$ C \to E $$ and $$ E \to G $$ (rule 3) 
4. $$ G \parallel I $$, since $$ G \not\to I $$ and $$ I \not\to G $$ 

## Lamport Clocks (LC)
Lamport clocks can be used to order events in a distributed system by assigning an integer counter *LC(e)* to every event *e*. In this case, an event could be a message send/receive or internal to a single process.

The rules for assigning the counter are as follows[2]:

1. Each process has a counter initialized to 0
2. Each process increments it's local counter after each event.
3. When sending a message, a process includes its current counter along with the message.
4. When receiving a message, the receiving process sets its counter to $$ max(local counter, message counter) + 1 $$

![assigning lamport clocks](/assets/img/distributed-systems-notes/assigning-lamport-clocks.png)

### Causality using LC
Consider two events A and B, with their Lamport clocks values *LC(A)* & *LC(B)* respectively. We can conclude the following results:

1. If $$ A \to B $$, then $$ LC(A) < LC(B) $$
2. If $$ LC(A) < LC(B) $$, then $$ A \to B $$ or $$ A \parallel B $$

As we can see, $$ LC(A) < LC(B) $$ does not gurantee $$ A \to B $$. It only guarantees that $$ B \not\to A $$ i.e. *B* could not have caused *A*. It can be considered as a limitation of Lamport clocks.

## Vector Clocks (VC)
Vector clocks are a generalized form of the Lamport clocks where every event *e* is assigned a *n* dimensional vector (hence called vector clocks) instead of a single integer counter. Here *n* is the number of processes in the system.

The rules for assigning the counter are as follows[1]:
1. Each process maintains a vector of integers initialized to all zeroes, one for each process.
2. Each process increments its own position in the vector clock after each event.
3. When sending a message, each process includes the current clock vector with it after incrementing its own position (since send is an event as well)
4. When receiving a message, the receiving process updates its vector clock to the maximum of the receiving clock and its own. It then increments its own position in the clock. Here maximum of the two vector clock is taken position wise.

![vector clock diagram](/assets/img/distributed-systems-notes/vector-clock-diagram.png)

In the above diagram, we have three processes $$ P_1, P_2 $$ and $$ P_3 $$ having internal events and sending/receiving messages amongst each other. The diagram shows the vector clock timestamp of each event according to the discussed rules.

### Causality using VC
Consider two events A and B, with their Vector clock values *VC(A)* and *VC(B)* respectively. We can conclude the following results:

1. If $$ A \to B $$, then $$ VC(A) < VC(B) $$
2. If $$ VC(A) < VC(B) $$, then $$ A \to B $$

Note that in the case of a n-dimensional vector, the $$ < $$ operator can be defined as follows:

For two clocks $$ VC(A) $$ and $$ VC(B) $$, we say that $$ VC(A) < VC(B) $$ when $$ \forall i \in [0..n), VC(A)_i <= VC(B)_i $$ and $$ VC(A) \ne VC(B) $$. Here, $$ VC(A)_i $$ denotes the i'th position in the vector. 

## Conclusion
In this post, we discussed the various techniques of ordering events in distributed systems.

Thanks for reading!

## References

[1] [UC Santa Cruz lectures - Lindsey Kuper](https://www.youtube.com/playlist?list=PLNPUF5QyWU8O0Wd8QDh9KaM1ggsxspJ31) - Lectures 3, 4 and 5 are relevant to this post 

[2] [Lamport Clocks paper](https://amturing.acm.org/p558-lamport.pdf)

[3] [Vector Clocks - Wikipedia](https://en.wikipedia.org/wiki/Vector_clock)

