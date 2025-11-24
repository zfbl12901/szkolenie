# 4. MongoDB Indexes and Performance

## 🎯 Objectives

- Understand indexes
- Create different index types
- Analyze performance
- Optimize queries
- Use explain()

## 📋 Table of Contents

1. [Introduction to Indexes](#introduction-to-indexes)
2. [Index Types](#index-types)
3. [Create Indexes](#create-indexes)
4. [Analyze Performance](#analyze-performance)
5. [Optimization](#optimization)

---

## Introduction to Indexes

### What is an Index?

**Index** = Data structure to speed up queries

- **Performance** : Faster search
- **Cost** : Additional disk space
- **Maintenance** : Automatic updates

---

## Index Types

### Simple Index

```javascript
// Index on one field
db.users.createIndex({email: 1})
```

### Compound Index

```javascript
// Index on multiple fields
db.users.createIndex({name: 1, age: -1})
```

### Unique Index

```javascript
// Ensure uniqueness
db.users.createIndex({email: 1}, {unique: true})
```

---

## Create Indexes

### Creation Methods

```javascript
// Create an index
db.collection.createIndex({field: 1})

// See indexes
db.collection.getIndexes()

// Drop an index
db.collection.dropIndex({field: 1})
```

---

## Analyze Performance

### explain()

```javascript
// Execution plan
db.users.find({email: "john@example.com"}).explain()

// Detailed statistics
db.users.find({email: "john@example.com"}).explain("executionStats")
```

---

## Optimization

### Best Practices

**1. Index frequently searched fields:**

```javascript
db.users.createIndex({email: 1})
```

**2. Compound index for multiple queries:**

```javascript
db.users.createIndex({name: 1, age: 1})
```

**3. Avoid too many indexes:**

- Each index slows writes
- Use only necessary indexes

---

## 📊 Key Takeaways

1. **Indexes** : Speed up searches
2. **Types** : Simple, compound, unique
3. **explain()** : Analyze performance
4. **Optimization** : Index frequent fields
5. **Balance** : Not too many indexes

## 🔗 Next Module

Proceed to module [5. Data Modeling](./05-data-modeling/README.md) to learn modeling.

