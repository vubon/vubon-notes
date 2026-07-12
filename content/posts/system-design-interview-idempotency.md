---
title: "System Design Interview: Idempotency"
date: 2026-05-04T11:00:00Z
draft: false
tags:
  - "System Design"
  - "Interview"
  - "Idempotency"
  - "Distributed Systems"
  - "API Design"
series:
  - "System Design Interview Prep"
categories:
  - "System Design"
  - "Interview"
description: "A deep dive into idempotency — what it means, why it matters in distributed systems, and how to answer it confidently in a system design interview."
keywords:
  - "system design"
  - "interview prep"
  - "idempotency"
  - "distributed systems"
---

Almost every candidate faces this question during a system design interview:

**How do you handle duplicate transactions?**

Most people answer: *Idempotency key + Cache.*

To be honest — that's a good start.

But the real lesson comes when you build an actual payment system.

---

## Duplicates are normal behavior

In distributed systems, duplicates are not edge cases. They are normal behavior.

Duplicates can originate from anywhere:

- client double-clicks
- mobile retry on timeout
- API gateway retries
- reverse proxies
- silent network failures
- third-party timeouts

So the problem is not *if* duplicates happen.

The real requirement becomes:

> **Retries must be safe without creating duplicate transactions. It shouldn't matter where the retry happens.**

---

## What is idempotency?

An operation is **idempotent** if performing it multiple times produces the same result as performing it once. Charge a customer once or ten times — idempotency means the outcome is always the same: one charge.

![Retries must always be safe — idempotency flow diagram](/images/idempotency-retries-safe.png)

---

## The approach that works

Here's what I've seen work well in production payment systems.

### 1. Redis as a performance optimization

Redis is **not** the guarantee layer.

It's only a fast lookup layer with temporary data. Think of it as a short-circuit — if the key is hot in cache, you skip the DB round-trip. But Redis alone is not enough.

### 2. Database = source of truth

If the same request arrives again:
- don't create a new transaction
- return the existing result

The database is the final source of truth. To handle concurrent retries safely, the insert must be atomic — if two identical requests arrive at the same time, only one should process and the other should return the stored result.

### 3. Persist before external calls

Before calling the bank or any third-party service:
- persist the transfer record
- persist the ledger intent/state

Why? Because external systems are unpredictable. When failures happen, you still have a consistent state and can safely retry or recover.

If the bank call fails or times out, you have the record. You can retry safely — the idempotency key prevents a double charge on the next attempt.

Those three layers together — Redis, database, and persist-before-call — are what make a payment system actually safe to retry.

---

## What a strong interview answer looks like

When an interviewer asks this question, they want to hear three things:

1. **You understand that retries are inevitable** — not edge cases. Networks fail. Clients timeout. Gateways retry.

2. **You know the layered approach** — Redis for speed, database for durability. Not one or the other.

3. **You persist state before calling external systems** — so failures leave you in a recoverable, consistent state.

A complete answer might sound like:

> "I'd have clients generate a UUID before making the request and send it as an `Idempotency-Key` header. On the server, I check Redis first for a fast response. If it's a miss, I check the database. If neither has it, I persist the intent to the DB before calling the external payment provider — so even if that call times out, I have a record and can recover. Once I get a result, I store it in both the DB and Redis. Any retry returns the same stored response."

That answer covers the real problem (retries everywhere), the layered solution, and the persistence-before-external-call pattern — which is what separates a good answer from a great one.

---

## Key takeaways

- Duplicates in distributed systems are **normal**, not edge cases
- Redis is a **performance layer**, not a guarantee
- The **database is the source of truth** — always
- **Persist intent before external calls** — this is the part most people miss
- Idempotency isn't a cache trick. It's a system design mindset

> In payments, correctness matters more than speed. Build systems that are safe to retry and many distributed problems suddenly become simpler.

---

➡️ Next: [Part 2: Partial Failure & Durability](/posts/system-design-interview-partial-failure/)

