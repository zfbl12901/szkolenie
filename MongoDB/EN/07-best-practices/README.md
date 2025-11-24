# 7. MongoDB Best Practices

## 🎯 Objectives

- Security
- Performance
- Maintenance
- Backup and Restore
- Monitoring

## 📋 Table of Contents

1. [Security](#security)
2. [Performance](#performance)
3. [Maintenance](#maintenance)
4. [Backup and Restore](#backup-and-restore)
5. [Monitoring](#monitoring)

---

## Security

### Authentication

```javascript
// Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: ["root"]
})
```

### Secure Connection

```bash
mongosh -u admin -p secure_password --authenticationDatabase admin
```

---

## Performance

### Indexes

```javascript
// Index frequently searched fields
db.users.createIndex({email: 1})

// Compound index for multiple queries
db.orders.createIndex({customer: 1, date: -1})
```

### Queries

```javascript
// Use projection to limit data
db.users.find({}, {name: 1, email: 1})

// Limit results
db.users.find().limit(100)
```

---

## Maintenance

### Cleanup

```javascript
// Delete obsolete documents
db.logs.deleteMany({
  created_at: {$lt: new Date("2024-01-01")}
})
```

---

## Backup and Restore

### Backup (mongodump)

```bash
# Backup a database
mongodump --db mydb --out /backup/
```

### Restore (mongorestore)

```bash
# Restore a database
mongorestore --db mydb /backup/mydb/
```

---

## Monitoring

### Server Status

```javascript
// Server status
db.serverStatus()

// Current operations
db.currentOp()
```

---

## 📊 Key Takeaways

1. **Security** : Authentication and authorization
2. **Performance** : Indexes and optimized queries
3. **Maintenance** : Regular cleanup
4. **Backup** : Regular backups
5. **Monitoring** : Monitor performance

## 🔗 Next Module

Proceed to module [8. Practical Projects](./08-projets/README.md) to create complete projects.

