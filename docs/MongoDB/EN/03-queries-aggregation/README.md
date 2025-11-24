# 3. MongoDB Queries and Aggregation

## 🎯 Objectives

- Master advanced queries
- Use aggregation pipeline
- Apply aggregation operators
- Perform grouping and calculations
- Analyze complex data

## 📋 Table of Contents

1. [Advanced Queries](#advanced-queries)
2. [Aggregation Pipeline](#aggregation-pipeline)
3. [Aggregation Operators](#aggregation-operators)
4. [Grouping and Calculations](#grouping-and-calculations)
5. [Practical Examples](#practical-examples)

---

## Advanced Queries

### Projection

```javascript
// Select specific fields
db.users.find({}, {name: 1, email: 1, _id: 0})
```

### Sort and Limit

```javascript
// Sort by age (ascending)
db.users.find().sort({age: 1})

// Limit results
db.users.find().limit(10)
```

---

## Aggregation Pipeline

### What is a Pipeline?

**Pipeline** = Series of transformation steps

- **Steps** : Each step transforms data
- **Sequential** : Result of one step = input of next
- **Powerful** : For complex analysis

### Basic Structure

```javascript
db.collection.aggregate([
  { $match: { ... } },      // Filter
  { $group: { ... } },       // Group
  { $sort: { ... } },        // Sort
  { $project: { ... } }      // Select
])
```

---

## Aggregation Operators

### $match

```javascript
db.sales.aggregate([
  {$match: {amount: {$gt: 500}}}
])
```

### $group

```javascript
db.products.aggregate([
  {
    $group: {
      _id: "$category",
      total: {$sum: "$price"},
      count: {$sum: 1},
      average: {$avg: "$price"}
    }
  }
])
```

### $project

```javascript
db.users.aggregate([
  {
    $project: {
      name: 1,
      age: 1,
      isAdult: {$gte: ["$age", 18]}
    }
  }
])
```

---

## Grouping and Calculations

### Accumulation Operators

```javascript
// Sum
{$sum: "$amount"}

// Average
{$avg: "$price"}

// Minimum
{$min: "$price"}

// Maximum
{$max: "$price"}

// Count
{$sum: 1}
```

---

## 📊 Key Takeaways

1. **Pipeline** : Series of transformation steps
2. **$match** : Filter documents
3. **$group** : Group and calculate
4. **$project** : Select and transform
5. **Aggregation** : Powerful for analysis

## 🔗 Next Module

Proceed to module [4. Indexes and Performance](./04-indexes-performance/README.md) to optimize performance.

