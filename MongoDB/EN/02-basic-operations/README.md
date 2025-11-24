# 2. MongoDB Basic Operations

## 🎯 Objectives

- Master CRUD operations
- Understand collections and documents
- Use data types
- Apply query operators
- Manage updates

## 📋 Table of Contents

1. [CRUD Operations](#crud-operations)
2. [Collections and Documents](#collections-and-documents)
3. [Data Types](#data-types)
4. [Query Operators](#query-operators)
5. [Updates](#updates)

---

## CRUD Operations

### Create

```javascript
// Insert one document
db.users.insertOne({
  name: "John",
  age: 30,
  email: "john@example.com"
})

// Insert many documents
db.users.insertMany([
  {name: "Alice", age: 28},
  {name: "Bob", age: 32}
])
```

### Read

```javascript
// Find all documents
db.users.find()

// Find with filter
db.users.find({age: 30})

// Find one document
db.users.findOne({name: "John"})

// Limit results
db.users.find().limit(5)

// Sort
db.users.find().sort({age: 1})
```

### Update

```javascript
// Update one document
db.users.updateOne(
  {name: "John"},
  {$set: {age: 31}}
)

// Update many documents
db.users.updateMany(
  {age: {$lt: 30}},
  {$set: {status: "young"}}
)
```

### Delete

```javascript
// Delete one document
db.users.deleteOne({name: "John"})

// Delete many documents
db.users.deleteMany({age: {$lt: 18}})
```

---

## Data Types

### Basic Types

```javascript
// String
{name: "John"}

// Number
{age: 30}
{price: 99.99}

// Boolean
{active: true}

// Date
{created_at: new Date()}

// Array
{hobbies: ["reading", "coding"]}

// Object (Embedded)
{address: {street: "123 Main St", city: "Paris"}}
```

---

## Query Operators

### Comparison Operators

```javascript
// Equal
db.users.find({age: 30})

// Greater than
db.users.find({age: {$gt: 30}})

// Less than
db.users.find({age: {$lt: 30}})

// In a list
db.users.find({age: {$in: [25, 30, 35]}})
```

### Logical Operators

```javascript
// AND
db.users.find({
  $and: [
    {age: {$gt: 25}},
    {age: {$lt: 35}}
  ]
})

// OR
db.users.find({
  $or: [
    {age: {$lt: 25}},
    {age: {$gt: 35}}
  ]
})
```

---

## Updates

### Update Operators

```javascript
// $set: Set a field
db.users.updateOne(
  {name: "John"},
  {$set: {age: 31}}
)

// $inc: Increment
db.users.updateOne(
  {name: "John"},
  {$inc: {age: 1}}
)

// $push: Add to array
db.users.updateOne(
  {name: "John"},
  {$push: {hobbies: "swimming"}}
)
```

---

## 📊 Key Takeaways

1. **CRUD** : Create, Read, Update, Delete
2. **Documents** : Flexible JSON format
3. **Collections** : Groups of documents
4. **Operators** : For filtering and updating
5. **Types** : Various data types supported

## 🔗 Next Module

Proceed to module [3. Queries and Aggregation](./03-queries-aggregation/README.md) for advanced queries.

