# MongoDB Training for Data Analyst

## 📚 Overview

This training guides you through learning **MongoDB** as a Data Analyst. MongoDB is a NoSQL document-oriented database, ideal for managing unstructured and semi-structured data.

## 🎯 Learning Objectives

- Understand MongoDB and NoSQL
- Install MongoDB
- Master CRUD operations
- Use queries and aggregations
- Optimize with indexes
- Model data
- Integrate MongoDB into your workflows
- Create practical projects for your portfolio

## 💰 Everything is Free!

This training uses only:
- ✅ **MongoDB Community Server** : Free and open-source
- ✅ **MongoDB Compass** : Free graphical interface
- ✅ **MongoDB Atlas** : Free cluster (512 MB)
- ✅ **Official Documentation** : Complete free guides
- ✅ **Online Tutorials** : Free resources

**Total Budget: $0**

## 📖 Training Structure

### 1. [MongoDB Getting Started](./01-getting-started/README.md)
   - Install MongoDB
   - Basic concepts
   - First operations
   - MongoDB Compass interface

### 2. [Basic Operations](./02-basic-operations/README.md)
   - CRUD (Create, Read, Update, Delete)
   - Collections and Documents
   - Data types
   - Query operators

### 3. [Queries and Aggregation](./03-queries-aggregation/README.md)
   - Advanced queries
   - Aggregation pipeline
   - Aggregation operators
   - Grouping and calculations

### 4. [Indexes and Performance](./04-indexes-performance/README.md)
   - Create indexes
   - Index types
   - Performance analysis
   - Query optimization

### 5. [Data Modeling](./05-data-modeling/README.md)
   - Data models
   - Relations (Embedded vs References)
   - Flexible schemas
   - Best practices

### 6. [Advanced Features](./06-advanced/README.md)
   - Transactions
   - Replication
   - Sharding
   - Text Search

### 7. [Best Practices](./07-best-practices/README.md)
   - Security
   - Performance
   - Maintenance
   - Backup and Restore

### 8. [Practical Projects](./08-projets/README.md)
   - Python application with MongoDB
   - Data pipeline
   - Data analysis
   - Portfolio projects

## 🚀 Quick Start

### Prerequisites

- **Operating System** : Windows, Linux, or macOS
- **4 GB RAM** : Minimum recommended
- **Disk Space** : 5 GB free

### Quick Installation

**Windows:**
1. Download MongoDB: https://www.mongodb.com/try/download/community
2. Install with default options
3. Verify: `mongod --version`

**Linux:**
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```

**macOS:**
```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

### First Test

```bash
mongod
mongosh
use test
db.collection.insertOne({name: "test"})
db.collection.find()
```

## 📊 Use Cases for Data Analyst

- **Unstructured data** : JSON, logs, APIs
- **Flexibility** : Evolving schemas
- **Aggregation** : Powerful pipeline for analysis
- **Integration** : With Python, R, PowerBI
- **Big Data** : Horizontal scalability

## 📚 Free Resources

### Official Documentation

- **MongoDB Documentation** : https://docs.mongodb.com/
- **MongoDB University** : https://university.mongodb.com/
- **MongoDB Compass** : https://www.mongodb.com/products/compass

