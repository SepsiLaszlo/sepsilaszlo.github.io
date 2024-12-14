---
layout: post
title: "Transaction Isolation in PostgreSQL"
date: 2024-12-13 13:52:00 +0200
categories: database
---

> This post is based on [Chapter 13. Concurrency Control of the PostgreSQL Documentation](https://www.postgresql.org/docs/17/mvcc.html) and [Serializable Snapshot Isolation in PostgreSQL article](https://arxiv.org/pdf/1208.4179).

Transaction isolation plays a key role in data consistency. Let's take a look at how PostgreSQL implements this feature.

## TLDR on Transaction Isolation Levels

Transactions running concurrently can interfere with each other and cause undesired phenomena like dirty reads, nonrepeatable reads, phantom reads, or serialization anomalies. These phenomena cause serious problems when maintaining data consistency with reality. Isolation levels aim to solve these problems by providing a different set of guarantees.

**Read Uncommitted** isolation level lets transactions read uncommitted changes from other concurrent transactions, so it does not prevent any of the above-mentioned phenomena.

**Read Committed** isolation level guarantees that transactions can only see changes from other committed transactions, thus, it prevents dirty reads.

**Repeatable Read** isolation level makes sure that if a transaction reads a row multiple times, it will always get the same result, thus, it prevents nonrepeatable reads. Dirty reads are also prevented.

**Snapshot** isolation level guarantees that all reads in a transaction see a common snapshot of the data. With this, it prevents phantom reads. Dirty reads and unrepeatable reads are also prevented. For a while, the snapshot isolation level was considered equal to the serializable isolation level, but it was discovered that some anomalies are still possible at the snapshot isolation level, such as write skew or the batch processing anomaly.

**Serializable** isolation level prevents all undesirable phenomena related to concurrent access. It guarantees that when the system executes multiple transactions concurrently, the result will be identical to a result that the transactions would have produced if they were run one at a time. This allows developers to write transactions as if they were executed sequentially.

## How PostgreSQL Implements Transaction Isolation

### Read Committed Isolation Level

PostgreSQL does not allow transactions to read uncommitted data, so the weakest isolation level in PostgreSQL is Read Committed.

This is the **default isolation level for PostgreSQL**.

In a transaction, **each statement sees a different snapshot** of the database, reflecting the state of the data right before it is executed. This means that we might get different results if we execute the same query twice in the same transaction. It is also possible that one query returns a set of rows that are referencing another table (like purchases referencing a customer), and referenced rows (customers) in the other table will not be present for another query, which can lead to an inconsistent report. This can happen because committed changes from concurrent transactions become visible at this isolation level.

To account for changes made by other transactions, a transaction using Read Committed isolation level **waits for the preceding concurrent transaction to complete and then re-evaluates its filter condition** to see if the rows are still matching the condition before it applies its changes. If the row is no longer matching the condition, then the changes are not applied. If the preceding transaction deleted the row, then the subsequent transaction will do nothing, since it can't apply its changes on a non-existent row. If the previous transaction rolled back its changes, then there is nothing special to do; the transaction can apply its changes as usual.

This conflict resolution is made possible by the fact that modification operations lock the affected rows implicitly.

Note that even though transactions detect changes from other concurrent transactions via lock, no error is raised in these scenarios. This means that all kinds of undesired concurrency phenomena can happen at this isolation level, except dirty reads.

### Repeatable Read Isolation Level

The Repeatable Read isolation level in PostgreSQL provides more guarantees than the SQL standard requires. Repeatable Read isolation in PostgreSQL essentially functions as Snapshot isolation, thanks to PostgreSQL's implementation of Multi-Version Concurrency Control.

Queries in the Repeatable Read isolation level in PostgreSQL **see a common snapshot of the data** taken before the first non-transaction control statement. This means that committed changes from other concurrent transactions can't affect what data is read by queries in transactions using the Repeatable Read isolation level in PostgreSQL. This eliminates the possibility of unrepeatable reads and phantom records.

If a transaction tries to modify a record using the Repeatable Read isolation level that has pending changes from another concurrent transaction, it will **wait for the preceding concurrent transaction to complete and raises an error and rolls back if the selected row is changed**. The error in these cases will display the following message:

```ruby
could not serialize due to concurrent update
```

This behavior is different from the way transactions using the Read Committed isolation level function. While a transaction using the Read Committed isolation level applies its changes on top of the changes made by a concurrent transaction, transactions using the Repeatable Read isolation level do not. Aborting a transaction in these cases is the safest option, since the transaction does not know about the changes made by a concurrent transaction, so applying both changesets could lead to broken business rules.

Although a transaction using the Repeatable Read isolation level is aborted when it tries to modify a row that was already changed by a concurrent transaction, it is not aborted when it reads a row modified by a concurrent transaction. This can still cause unintended phenomena.

### Serializable Isolation Level

The Serializable isolation level is the strongest isolation level. It guarantees the full isolation of transactions; no concurrency phenomena are possible at this level.

PostgreSQL achieves this by issuing predicate locks on reads and checking for possible anomalies on commit. This check only happens at the time of committing a transaction. Reads from a serializable transaction can't be considered safe until their transaction has committed. The exception is `SERIALIZABLE READ ONLY DEFERRABLE` transactions that wait until a safe snapshot becomes available that can no longer be modified by any concurrent transactions. When a possible anomaly is detected on commit, the transaction gets rolled back, with the following error:

```ruby
could not serialize access due to read / write dependencies
```

This makes application development much easier since engineers don't have to worry about concurrency phenomena or use complicated locking schemes. PostgreSQL's serializable isolation level implementation generally has better performance than locking.

## Why is the Read Committed Isolation Level the Default for PostgreSQL?

At first glance, it might seem unusual that the weakest isolation level is the default. One possible reason could be that this isolation level was implemented first, because it was the simplest. Another reason could be that stronger isolation levels may result in serialization errors, which can cause problems for applications that are not prepared to deal with these kinds of errors.

## Which Transaction Isolation Level Should I Use?

The [13.4. Data Consistency Checks at the Application Level](https://www.postgresql.org/docs/17/applevel-consistency.html) chapter of the PostgreSQL official documentation states that it is very difficult to enforce business rules regarding data integrity using Read Committed transactions. The documentation also states that integrity checks will not work correctly without locking using the Repeatable Read isolation level, due to read/write conflicts. **If we want to guarantee consistency without locking, we must use the Serializable isolation level.** When using Serializable isolation level we must make sure that all transactions are run using this isolation level because even ad-hoc nonserializable transactions can introduce inconsistencies to the system.
