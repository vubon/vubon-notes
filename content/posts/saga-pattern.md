---
title: "Saga Pattern: Handling Distributed Transactions in Microservices"
date: 2026-06-03
draft: false
tags:
  - "System Design"
  - "Pattern"
  - "Microservices"
  - "Distributed Systems"
  - "Payments"
categories:
  - "System Design"
description: "A practical introduction to the Saga Pattern, why rollback breaks down across microservices, and how compensation keeps distributed workflows consistent."
aliases:
  - "/saga-pattern/"
---

Whenever microservices enter a system design discussion, one question always comes to my mind:

**What happens when a transaction succeeds in one service but fails in another?**

Let's take a payment system as an example.

Imagine a user initiates a $100 transfer. The workflow looks like this:

- Transfer Service creates the transaction ✅
- Wallet Service deducts $100 from the user's balance ✅
- Bank Service sends the transfer request ❌

Now we have a problem.

The user's balance has already been deducted, but the bank transfer failed.

Since each service owns its own database, we can't simply rollback everything using a single database transaction.

This is where the **Saga Pattern** comes in.

![Saga Pattern diagram](/images/saga-pattern.png)

## Breaking the Workflow Into Local Transactions

Instead of treating the workflow as one large transaction, Saga breaks it into multiple local transactions:

```
Transfer Service → Wallet Service → Bank Service
```

If every step succeeds, the state transitions look like this:

```
CREATED → BALANCE_DEDUCTED → BANK_SUCCESS → COMPLETED
```

But if the Bank Service fails:

```
CREATED → BALANCE_DEDUCTED → BANK_FAILED → COMPENSATION → BALANCE_RELEASED
```

The system doesn't rollback. **The system compensates.**

In this case, the compensation action is releasing the reserved balance back to the user.

## Why This Matters

Traditional database rollback works when everything lives in one database. One transaction, one commit, one rollback.

But in a microservice architecture, each service commits its own local transaction independently. By the time the Bank Service fails, the Wallet Service has already committed its deduction.

There is no global transaction to rollback.

Saga solves this by designing compensation as a first-class part of the workflow. Every forward step has a corresponding compensation step defined upfront.

## When Compensation Itself Can Fail

Of course, compensation can fail too.

What if the Wallet Service is down when the system tries to release the balance? What if the recovery worker crashes mid-process?

That is why mature systems combine Saga with:

- retry mechanisms
- recovery workers
- reconciliation jobs

The goal is not immediate consistency. **The goal is eventual business consistency.**

The system keeps trying until it reaches a known, stable state.

## Last Word

As I always say, choose the pattern that works for your system. There is no good or bad design pattern.

The right choice depends on your business requirements, consistency requirements, operational complexity, and system architecture.

For example, **2PC (Two-Phase Commit)** is another way to solve distributed transaction challenges. Every approach comes with trade-offs.

Choose the one that fits your situation and system design.
