# 5. MongoDB Data Modeling

## 🎯 Objectives

- Understand data models
- Choose between Embedded and References
- Design flexible schemas
- Apply best practices
- Optimize structure

## 📋 Table of Contents

1. [Data Models](#data-models)
2. [Embedded vs References](#embedded-vs-references)
3. [Flexible Schemas](#flexible-schemas)
4. [Best Practices](#best-practices)
5. [Practical Examples](#practical-examples)

---

## Data Models

### Embedded Model

**Everything in one document:**

```javascript
{
  _id: ObjectId("..."),
  name: "John",
  address: {
    street: "123 Main St",
    city: "Paris"
  }
}
```

**Advantages:**
- Fast access (single document)
- No joins
- Consistent data

### References Model

**Separate documents with references:**

```javascript
// Collection users
{_id: ObjectId("..."), name: "John"}

// Collection addresses
{_id: ObjectId("..."), user_id: ObjectId("..."), street: "123 Main St"}
```

**Advantages:**
- No size limit
- No duplication
- Flexibility

---

## Embedded vs References

### When to Use Embedded?

**Use cases:**
- Data often accessed together
- Small amounts of data
- 1:1 or 1:few relationship
- Data that rarely changes

### When to Use References?

**Use cases:**
- Large amounts of data
- 1:many or many:many relationship
- Shared data
- Data that changes often

---

## Flexible Schemas

### Schema Evolution

```javascript
// Initial document
{name: "John", age: 30}

// Add field later
{name: "John", age: 30, email: "john@example.com"}
```

---

## Best Practices

### Modeling Patterns

**One-to-Few:**
```javascript
// Embedded
{name: "John", addresses: [{street: "123 Main St"}]}
```

**One-to-Many:**
```javascript
// References
// Collection users
{_id: ObjectId("..."), name: "John"}

// Collection orders
{user_id: ObjectId("..."), items: [...]}
```

---

## 📊 Key Takeaways

1. **Embedded** : For data often accessed together
2. **References** : For large amounts or complex relations
3. **Flexibility** : Evolving schemas
4. **Patterns** : One-to-Few, One-to-Many, Many-to-Many
5. **Performance** : Balance access and consistency

## 🔗 Next Module

Proceed to module [6. Advanced Features](./06-advanced/README.md) to deepen.

