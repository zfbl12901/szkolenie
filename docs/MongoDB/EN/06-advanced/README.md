# 6. MongoDB Advanced Features

## 🎯 Objectives

- Use transactions
- Understand replication
- Master sharding
- Use text search
- Advanced features

## 📋 Table of Contents

1. [Transactions](#transactions)
2. [Replication](#replication)
3. [Sharding](#sharding)
4. [Text Search](#text-search)
5. [Other Features](#other-features)

---

## Transactions

### What is a Transaction?

**Transaction** = Group of atomic operations

- **Atomic** : All or nothing
- **Consistency** : Consistent data
- **Isolation** : Isolated operations
- **Durability** : Persistent changes

### Use Transactions

```javascript
const session = db.getMongo().startSession()
session.startTransaction()

try {
  db.users.insertOne({name: "John"}, {session})
  db.orders.insertOne({user_id: "...", items: [...]}, {session})
  session.commitTransaction()
} catch (error) {
  session.abortTransaction()
} finally {
  session.endSession()
}
```

---

## Replication

### What is Replication?

**Replication** = Multiple copies of data

- **High availability** : No single point of failure
- **Redundancy** : Automatic backup
- **Performance** : Read from multiple servers

---

## Sharding

### What is Sharding?

**Sharding** = Horizontal partitioning

- **Scalability** : Distribute data
- **Performance** : Process in parallel
- **Storage** : More capacity

---

## Text Search

### Text Index

```javascript
// Create text index
db.articles.createIndex({
  title: "text",
  content: "text"
})

// Search
db.articles.find({
  $text: {$search: "mongodb tutorial"}
})
```

---

## 📊 Key Takeaways

1. **Transactions** : Atomic operations
2. **Replication** : High availability
3. **Sharding** : Horizontal scalability
4. **Text Search** : Text search
5. **Validation** : Optional schemas

## 🔗 Next Module

Proceed to module [7. Best Practices](./07-best-practices/README.md) for best practices.

