---
title: "Handling Postgres Locks and Connection Starvation in Go (Part 2)"
description: "Learn how PostgreSQL locks can cause connection pool starvation in Go applications, and discover how to safely mitigate these bottlenecks using GORM and pinned connections."
keywords:
  - "Go"
  - "Golang"
  - "GORM"
  - "PostgreSQL"
  - "connection pool starvation"
  - "database locks"
  - "pg_advisory_lock"
  - "pg_advisory_xact_lock"
date: 2026-07-24T10:00:00+07:00
draft: false
categories: ["Go", "PostgreSQL", "Architecture"]
tags: ["golang", "gorm", "connection-pool", "locks", "database", "backend"]
---

In [Part 1 of this series]({{< ref "postgres-connections-and-locks-part-1.md" >}}), we explored how PostgreSQL manages backend processes and OS-level locking. Now, we will look at how those database-level locks interact with an application's connection pool, causing connection starvation, and how to elegantly mitigate these issues in Go using GORM.

<!--more-->

## 1. The Application Layer: Connection Starvation

Modern applications enforce safety guardrails using connection pools. In Go, pools are managed by setting thresholds like `SetMaxOpenConns(5)`.

When all connections in a pool become frozen waiting on database locks, your application suffers **pool starvation**. Let's empirically demonstrate this using a Go app built with GORM.

We set our PostgreSQL `max_connections` limit to `5`, and our Go app's `MaxOpenConns` to `5`. We then launched 5 goroutines, all trying to acquire a session-level lock (`pg_advisory_lock(42)`). 

Here is the terminal output from our application:

```text
==========================================================
 Go Level Experiment: Connection Starvation & PG Exhaustion
==========================================================

Spawning 5 goroutines to acquire pg_advisory_lock(42)...

Go Connection Pool Stats:
InUse: 5, Idle: 0, WaitCount: 0
```

All 5 slots in the Go connection pool were consumed. What happens at the Postgres level?

```text
[Visual Log] Checking Postgres pg_stat_activity from outside the Go app...
psql: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  sorry, too many clients already
```

The database completely rejected us because the connection limit (5) was hit. 

Finally, our Go application attempted to execute a harmless, completely unrelated query (`SELECT 1`):

```text
[App Lockout] Attempting to run a simple 'SELECT 1'...

[App Lockout] Query failed/timed out after 2.001s: context deadline exceeded
[App Lockout] PROOF: The application pool is starved! We cannot even run a SELECT 1.
```

The failure bottleneck shifted into the Go memory stack. The `SELECT 1` query hung indefinitely because GORM couldn't find an available TCP wire to the database.

## 2. Real-World Case Study: golang-migrate

![golang-migrate distributed locks](/images/golang-migrate-locking.jpg)

> In Part 1, we warned that session-level locks (`pg_advisory_lock`) are dangerous for API requests, but perfect for singleton background tasks like schema migrations. The incredibly popular [golang-migrate/migrate](https://github.com/golang-migrate/migrate) library is a perfect real-world example of this.

### How it works in a Distributed System

Imagine you deploy a new version of your Go application to Kubernetes, spinning up 5 replicas simultaneously. All 5 pods boot up and attempt to run your database migrations on startup. If they all tried to execute `CREATE TABLE` at the exact same time, you would get race conditions, deadlocks, and corrupted schemas.

To prevent this, `golang-migrate` implements a **Distributed Mutex Algorithm** using Postgres session locks:

1. **Hash Generation:** To ensure the lock is unique and doesn't accidentally block migrations running in different schemas within the same database, the library [generates a deterministic integer lock ID](https://github.com/golang-migrate/migrate/blob/18966c755f83de2b250dd8e766bad8b769cd0e25/database/postgres/postgres.go#L233) by joining the **Database Name**, **Schema Name**, and **Migrations Table Name**, calculating their `CRC32` checksum, and multiplying it by a hardcoded salt (`1486364155`).
2. **Lock Acquisition:** All 5 pods execute `SELECT pg_advisory_lock(lock_id)`. 
3. **The Race:** Postgres guarantees atomicity. Only Pod A gets the lock. Pods B, C, D, and E are forcefully put to sleep (Session Wait).

   If we query the `pg_stat_activity` table while the pods are racing, we can visually see the losing pods patiently waiting in line without burning CPU:
   ```text
    pid | state  | wait_event_type | wait_event |            query            
   -----+--------+-----------------+------------+-----------------------------
     40 | active | Lock            | advisory   | SELECT pg_advisory_lock($1)
     43 | active | Lock            | advisory   | SELECT pg_advisory_lock($1)
     42 | active | Lock            | advisory   | SELECT pg_advisory_lock($1)
   (3 rows)
   ```
4. **Execution:** Pod A checks the `schema_migrations` table, realizes the database is outdated, and applies the new SQL scripts. Because migrations often include non-transactional commands (like `CREATE INDEX CONCURRENTLY`) and span multiple files, this *must* be a session lock rather than a transaction lock.
5. **Release:** Once finished, Pod A updates the `schema_migrations` table and explicitly calls `pg_advisory_unlock(lock_id)`.
6. **Graceful Wakeup:** Pod B wakes up, acquires the lock, and checks the `schema_migrations` table. Seeing the database is already up to date, it instantly unlocks and continues booting.

This is exactly why `pg_advisory_lock` exists. It leverages the database's shared memory to orchestrate safety across distributed infrastructure!

## 3. Architectural Mitigations

How do we safely execute concurrency locks within GORM without risking pool starvation?

### A. Non-Blocking Queries with Pinned Connections

To prevent connection starvation, you must swap blocking calls for non-blocking queries. Here is how they compare:

| Function | Behavior if Lock is Taken | Risk | Best Use Case |
|---|---|---|---|
| `pg_advisory_lock` | **Blocks** the thread until the lock is released. | Connection pool starvation. | Background workers, schema migrations (`golang-migrate`). |
| `pg_try_advisory_lock` | **Returns immediately** with a `false` boolean. | None. Fail-fast allows immediate release of the connection. | API endpoints, web servers, high concurrency. |

By failing fast using `pg_try_advisory_lock`, we can instantly return an HTTP `409 Conflict` to the user and return the connection to the pool, guaranteeing we never starve the database. 

More importantly, you **must manually pin a connection** before using session-level locks. Why? As warned by the [official Go `database/sql` documentation](https://go.dev/doc/database/execute-transactions), the connection pool executes independent queries across random physical TCP connections. 

If you run `db.Exec("SELECT pg_advisory_lock(42)")` on line 1, and `db.Exec("SELECT pg_advisory_unlock(42)")` on line 2, Go might execute them on two entirely different Postgres PIDs! The unlock command will silently fail, and the original PID will hold the lock forever (or at least until the application crashes and the physical TCP socket is finally severed).

To empirically prove this, we can fetch the `pg_backend_pid()` alongside our lock commands to see exactly which physical connections Go assigns:

```go
var lockPID, unlockPID int
var unlocked bool

// 1. Acquire lock on a random pool connection
db.Raw("SELECT pg_backend_pid(), pg_advisory_lock(42)").Row().Scan(&lockPID, nil)

// 2. Attempt to unlock on the NEXT random pool connection
db.Raw("SELECT pg_backend_pid(), pg_advisory_unlock(42)").Row().Scan(&unlockPID, &unlocked)

fmt.Printf("Lock acquired on PID: %d\n", lockPID)
fmt.Printf("Unlock attempted on PID: %d\n", unlockPID)
fmt.Printf("Unlock Successful? %v\n", unlocked)
```

Running this script exposes the terrifying reality of unpinned connections:

```text
Lock acquired on PID: 74
Unlock attempted on PID: 81
Unlock Successful? false
```

Because the pool grabbed PID `81` for the second query, the unlock failed. If we query the database directly by joining `pg_locks` and `pg_stat_activity`, we can see the damning evidence. PID `74` is completely `idle`, yet it still permanently holds the lock!

```sql
SELECT l.pid, a.state, l.locktype, l.objid
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.locktype = 'advisory' AND l.objid = 42;
```

```text
 pid | state | locktype | objid 
-----+-------+----------+-------
  74 | idle  | advisory |    42
(1 row)
```

You must check out a single, dedicated connection (`sqlDB.Conn`) to guarantee the lock and unlock happen on the exact same Postgres session:

```go
// Pin a single physical connection from the pool
pinnedConn, _ := sqlDB.Conn(context.Background())
defer pinnedConn.Close() 

var lockAcquired bool
// pg_try_advisory_lock returns immediately with a boolean (true/false)
pinnedConn.QueryRowContext(context.Background(), "SELECT pg_try_advisory_lock($1)", 42).Scan(&lockAcquired)

if !lockAcquired {
    return errors.New("resource locked, try again later")
}
defer pinnedConn.ExecContext(context.Background(), "SELECT pg_advisory_unlock($1)", 42)
```

### B. Shift Scopes to Transaction-Level Locks

![transaction lock flow](/images/xact-lock-flow.png)

Session locks are dangerous if connections are reused incorrectly. `pg_advisory_xact_lock` bounds the lock strictly to the lifecycle of the active database transaction. 

But why use an advisory lock if you're already inside a transaction? 
Standard SQL transactions (`BEGIN`/`COMMIT`) ensure database writes succeed or fail together, but they do not serialize concurrent application logic unless you lock specific rows (e.g., `SELECT FOR UPDATE`). If you need to serialize a complex process that doesn't map cleanly to a single database row (e.g., preventing a user from triggering two external Stripe payouts simultaneously), you need an application-level mutex. 

`pg_advisory_xact_lock` gives you that global mutex, but elegantly leverages GORM's built-in transaction methods to guarantee it is automatically cleaned up, eliminating the risk of leaks:

To see why this is so powerful, let's look at two real-world examples: one where standard row locks are enough, and one where advisory locks are mandatory.

#### Scenario 1: Internal Wallet Transfer (Row Lock is Enough)
If you are strictly modifying database state (e.g., deducting a balance), you do not need an advisory lock. A standard GORM transaction with `SELECT ... FOR UPDATE` (a Row Lock) is perfect. The database serializes the writes automatically:

```go
db.Transaction(func(tx *gorm.DB) error {
    var wallet Wallet
    // Row Lock: FOR UPDATE prevents concurrent modifications to this specific row
    if err := tx.Clauses(clause.Locking{Strength: "UPDATE"}).Where("user_id = ?", userID).First(&wallet).Error; err != nil {
        return err
    }
    
    // Update balance
    return tx.Model(&wallet).Update("balance", wallet.Balance - 100).Error
})
```

#### Scenario 2: Wallet-to-Bank Payout (Advisory Lock is Required)
Imagine a merchant withdrawing money from their in-app Wallet to their actual Bank Account. If you need to execute external application logic (like calling a Bank API) *before* updating the database, a row lock is useless. If a user clicks "Withdraw" twice rapidly, both Go routines will query the wallet balance, both will call the Bank API, and you will double-pay the user before the database has a chance to lock the row!

By using `pg_advisory_xact_lock` (hashing the `userID` into an integer lock ID), you serialize the entire Go function block, while still leveraging the transaction to automatically clean up the lock:

```go
db.Transaction(func(tx *gorm.DB) error { 
	// 1. Global Mutex: Prevent any other API requests for this specific user.
    // This lock is automatically deleted from shared memory the microsecond  
	// the transaction hits COMMIT or ROLLBACK.
	tx.Exec("SELECT pg_advisory_xact_lock(?)", hash(userID))
    
    var wallet Wallet
    tx.Where("user_id = ?", userID).First(&wallet)
    if wallet.Balance < withdrawalAmount {
        return errors.New("insufficient balance") // Rolls back, releases lock
    }

    // 2. External Bank API Call (SAFE from concurrent race conditions)
    err := bank.Transfer(wallet.BankAccountID, withdrawalAmount)
    
    if err != nil {
        // Bank rejected it! We insert a FAILED audit log and return nil to COMMIT.
        // By committing, we save the history without deducting the balance.
        // The advisory lock is perfectly safe—it is automatically released upon COMMIT!
        tx.Create(&TransferHistory{UserID: userID, Status: "FAILED", Reason: err.Error()})
        return nil 
    }

    // 3. Success! Update Balance and save audit log
    tx.Model(&wallet).Update("balance", wallet.Balance - withdrawalAmount)
    tx.Create(&TransferHistory{UserID: userID, Status: "SUCCESS"})
    
    return nil // Commits the transaction, which automatically releases the lock
}) 
```

To prove the safety of this method, we ran `pg_advisory_xact_lock` inside a transaction and monitored the `pg_stat_activity` table:

```text
Transaction Lock (pg_advisory_xact_lock)
 pid | wait_event_type |                                query                                 
-----+-----------------+----------------------------------------------------------------------
  74 | Timeout         | BEGIN; SELECT pg_advisory_xact_lock(42); SELECT pg_sleep(2); COMMIT;
```

Once the transaction committed 2 seconds later, Postgres automatically wiped the lock from shared memory. Zero connection leaks, zero deadlocks, and zero double-payouts.

### A Note on Alternative Architectures
While Postgres advisory locks are incredibly powerful, they aren't the *only* way to solve external API race conditions. In highly distributed microservice architectures, you might opt for:
1. **Distributed Caching (Redis Mutex):** Using `SETNX` in Redis to lock the user ID across multiple distributed databases.
2. **Idempotency Keys (Two-Phase Commit):** Inserting a `PENDING` transfer record with a unique constraint before calling the API, relying on database unique key violations to prevent race conditions.
3. **Asynchronous Workers:** Deducting the wallet balance to an "escrow" state instantly using a Row Lock, and letting a background worker queue (like Temporal or Asynq) handle the actual external Bank API call safely.

However, for monolithic Go applications or medium-sized services, `pg_advisory_xact_lock` hits the perfect "sweet spot." It gives you the safety of a global mutex without needing to deploy Redis or build a complex asynchronous worker architecture!

---

By understanding how connection pools interact with database backend processes, you can architect robust backend services in Go that elegantly handle contention without complete application lockouts.
