# 1. MongoDB Getting Started

## 🎯 Objectives

- Understand MongoDB and NoSQL
- Install MongoDB
- Understand basic concepts
- Use MongoDB Compass
- First operations

## 📋 Table of Contents

1. [Introduction to MongoDB](#introduction-to-mongodb)
2. [Installation](#installation)
3. [Basic Concepts](#basic-concepts)
4. [MongoDB Compass](#mongodb-compass)
5. [First Operations](#first-operations)

---

## Introduction to MongoDB

### What is MongoDB?

**MongoDB** = NoSQL document-oriented database

- **NoSQL** : Non-relational
- **Documents** : Storage in JSON format (BSON)
- **Flexible** : Evolving schemas
- **Scalable** : Horizontal scalability
- **Open-source** : Free and open-source

### Why MongoDB for Data Analyst?

- **Unstructured data** : JSON, logs, APIs
- **Flexibility** : Evolving schemas
- **Aggregation** : Powerful pipeline for analysis
- **Integration** : With Python, R, PowerBI
- **Performance** : Fast for complex queries

---

## Installation

### Windows

1. Go to: https://www.mongodb.com/try/download/community
2. Select Windows
3. Download MSI installer
4. Run installer
5. Choose "Complete" installation

### Linux

**Ubuntu/Debian:**
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```

### macOS

**With Homebrew:**
```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

---

## Basic Concepts

### Database

**Database** = Container of collections

- **Auto-creation** : Created on first use
- **Name** : Unique identifier
- **Collections** : Contains collections

### Collection

**Collection** = Group of documents

- **Equivalent** : Table in SQL
- **Flexible** : No imposed schema
- **Documents** : Contains documents

### Document

**Document** = Record in JSON format

- **Format** : BSON (Binary JSON)
- **Flexible** : Variable structure
- **Fields** : Key-value pairs

---

## MongoDB Compass

### What is Compass?

**MongoDB Compass** = Graphical interface

- **Visualization** : View data
- **Queries** : Execute queries
- **Analysis** : Analyze performance
- **Management** : Manage indexes

### Installation

1. Download: https://www.mongodb.com/products/compass
2. Install
3. Launch Compass
4. Connect to `mongodb://localhost:27017`

---

## First Operations

### Connect with mongosh

```bash
# Launch mongosh
mongosh

# See databases
show dbs

# Use a database
use mydb

# See collections
show collections

# Insert a document
db.users.insertOne({name: "John", age: 30, city: "Paris"})

# Find documents
db.users.find()
```

---

## 📊 Key Takeaways

1. **MongoDB** = NoSQL document-oriented database
2. **Documents** = JSON format (BSON)
3. **Collections** = Groups of documents
4. **Databases** = Containers of collections
5. **Compass** = Graphical interface

## 🔗 Next Module

Proceed to module [2. Basic Operations](./02-basic-operations/README.md) to master CRUD.

