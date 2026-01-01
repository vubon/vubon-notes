---
title: 'Fan-Out Fan-In in Golang — The easy way'
date: 2026-01-01
lastmod: 2026-01-01
draft: false
tags:
  - 'Golang'
  - 'Concurrency'
  - 'Go'
categories:
  - 'Golang'
aliases:
  - '/fan-out-fan-in-in-golang/'
---

These days, low latency isn’t “nice to have” — it’s table stakes. So let’s talk about the Fan-Out/Fan-In pattern and why it’s my go-to for faster APIs and background work. After 6+ years building FinTech systems, I’ll share a real example that makes the concept easy to digest and hard to forget. You’ll be able to plug it into your services right away.

But before the demo, let’s hit the basics.

## What’s Fan-Out / Fan-In?

Think about making breakfast: You start the coffee machine ☕ and pop bread into the toaster 🍞 at the same time. Both run independently, and when they’re done, you bring them together for your meal.


That’s basically Fan-Out/Fan-In in Go:

- Fan-out → Start multiple tasks at the same time.
- Fan-in → Wait for all of them to finish, then combine results

Let’s visualize with the diagram below.

```
        Request
           │
           ▼
     ┌───────────┐
     │   Fan-Out │
     └─────┬─────┘
           │
   ┌───────┴────────┐
   │                │
   ▼                ▼
Task A          Task B
   │                │
   └───────┬────────┘
           │
     ┌───────────┐
     │   Fan-In  │
     └─────┬─────┘
           │
           ▼
     Final Result
```

## Visa Transaction Example

Let’s walk through a real example. Assume we need to process a payment with Visa (Card Scheme). But before we can hit Visa’s endpoint, we have to call to a couple of own internal microservices:

- **IIN Service** → to figure out the issuing bank
- **3DS Service** → to check if authentication is required

```go
func ProcessVisaTransaction(cardNumber string) (VisaPayload, error) {
 var wg sync.WaitGroup
 var iinResp IINResponse
 var dsResp ThreeDSResponse
 var iinErr, dsErr error

 wg.Add(2)

 // IIN Service
 go func() {
  defer wg.Done()
  iinResp, iinErr = IINService(cardNumber)
 }()

 // 3DS Check
 go func() {
  defer wg.Done()
  dsResp, dsErr = ThreeDSService(cardNumber)
 }()

 // FAN-IN happens here
 wg.Wait()

 // Do something base on failure logic
 if iinErr != nil {
  return VisaPayload{}, fmt.Errorf("IIN error: %v", iinErr)
 }
 if dsErr != nil {
  return VisaPayload{}, fmt.Errorf("3DS error: %v", dsErr)
 }

 // Combine results and call Visa API
 return VisaThirdPartyAPI(iinResp, dsResp)
}
```

## Where’s the Fan-Out & Fan-In?

The fan-out is when we launch both goroutines:

```go
go func() { ... }()
go func() { ... }()
```

The fan-in is here:
```go
wg.Wait()
```
That’s the moment we stop, wait for both to finish, and then combine the results.

Let’s visualize the whole scenario — this will make it easier to remember the pattern for a long time.

![Fan-Out Fan-In Pattern](/images/fan-out-fan-in.png)

## Why Bother?
Let’s put some numbers on this to see the difference. Suppose each service responds with the following latencies:

- IIN Service takes: 200ms
- 3DS Check takes: 300ms

**Sequential way**: 200ms + 300ms = 500ms total
**Concurrent way**: max(200ms, 300ms) = 300ms total

## When to Use?
You can use this pattern when each task can execute independently. So there’s no barrier to running them in parallel.

## Gotchas


## Gotchas

- Watch for **data races** when writing to shared variables.
- Always **handle errors separately** for each goroutine.
- Concurrency doesn’t always mean faster — if tasks are CPU-heavy instead of IO-heavy, you might just be adding complexity.

## Final Thoughts
Fan-Out/Fan-In is one of those patterns that’s super simple but incredibly useful. Whenever you’ve got independent tasks — whether it’s calling APIs, running background jobs, or anything else you can think of — you can consider using this pattern.

**Less waiting, more doing**

