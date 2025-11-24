# Apache Airflow Training for Data Analyst

## 📚 Overview

This training guides you through learning **Apache Airflow** as a Data Analyst. Airflow is an open-source platform for orchestrating and automating complex data workflows.

## 🎯 Learning Objectives

- Understand Apache Airflow and its role in ETL orchestration
- Install and configure Airflow
- Create DAGs (Directed Acyclic Graphs)
- Use operators, sensors, and hooks
- Orchestrate complex data pipelines
- Integrate with databases and cloud services
- Create practical projects for your portfolio

## 💰 Everything is Free!

This training uses only:
- ✅ **Apache Airflow** : Open-source and free
- ✅ **Python** : Free programming language
- ✅ **PostgreSQL/SQLite** : Free databases
- ✅ **Official Documentation** : Complete free guides

**Total Budget: $0**

## 📖 Training Structure

### 1. [Airflow Getting Started](./01-getting-started/README.md)
   - Install Airflow
   - Basic configuration
   - Airflow web interface
   - First DAGs

### 2. [Fundamental Concepts](./02-concepts/README.md)
   - DAGs (Directed Acyclic Graphs)
   - Tasks and dependencies
   - Scheduling and triggers
   - Variables and connections

### 3. [Operators](./03-operators/README.md)
   - Python operators
   - SQL operators
   - Bash operators
   - Custom operators

### 4. [Sensors](./04-sensors/README.md)
   - FileSensor
   - SqlSensor
   - HttpSensor
   - Custom sensors

### 5. [Hooks](./05-hooks/README.md)
   - Database hooks
   - Cloud hooks (AWS, Azure)
   - HTTP hooks
   - Create custom hooks

### 6. [Variables and Connections](./06-variables-connections/README.md)
   - Manage variables
   - Configure connections
   - Security and best practices
   - Dynamic variables

### 7. [Best Practices](./07-best-practices/README.md)
   - DAG structure
   - Error handling
   - Performance and optimization
   - Testing and debugging

### 8. [Practical Projects](./08-projets/README.md)
   - Complete ETL pipeline
   - Workflow orchestration
   - Database integration
   - Portfolio projects

## 🚀 Quick Start

### Prerequisites

- **Python 3.8+** : Installed on your system
- **pip** : Python package manager
- **PostgreSQL** (optional) : For metadata database

### Quick Installation

```bash
# Create a virtual environment
python -m venv airflow-env

# Activate the environment
# Windows
airflow-env\Scripts\activate
# Linux/Mac
source airflow-env/bin/activate

# Install Airflow
pip install apache-airflow

# Initialize the database
airflow db init

# Create an admin user
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com

# Start the web server
airflow webserver --port 8080

# In another terminal, start the scheduler
airflow scheduler
```

### Access the Web Interface

1. Open a browser
2. Go to: `http://localhost:8080`
3. Log in with the created credentials

## 📊 Use Cases for Data Analyst

- **ETL Orchestration** : Coordinate data pipelines
- **Scheduling** : Schedule recurring tasks
- **Monitoring** : Monitor workflow execution
- **Error Handling** : Automatic retry and alerts
- **Integration** : Connect multiple tools and services

## ⚠️ Remote Installation

If you install Airflow on machine A and want to access it from machine B, see the guide [Remote Installation and Access](./INSTALLATION_REMOTE.md).

## 📚 Free Resources

### Official Documentation

- **Apache Airflow** : https://airflow.apache.org/docs/
  - Complete guides
  - Step-by-step tutorials
  - Code examples
  - API Reference

- **GitHub Airflow** : https://github.com/apache/airflow
  - Source code
  - DAG examples
  - Contributions

### External Resources

- **YouTube** : Airflow tutorials
- **Medium** : Articles and guides
- **Stack Overflow** : Questions and answers

## 🎓 Certifications (Optional)

### Apache Airflow (no official certification)

- **Training** : Free documentation and tutorials
- **Duration** : 2-4 weeks
- **Level** : Intermediate to advanced

## 📝 Conventions

- All examples use Python 3.8+
- DAGs are tested on Airflow 2.x
- Paths may vary by operating system
- Default ports can be modified

## 🤝 Contribution

This training is designed to be evolving. Feel free to suggest improvements or additional use cases.

## 📚 Additional Resources

- [Apache Airflow Documentation](https://airflow.apache.org/docs/)
- [GitHub Apache Airflow](https://github.com/apache/airflow)
- [Airflow Community](https://airflow.apache.org/community/)
- [Airflow Examples](https://github.com/apache/airflow/tree/main/airflow/example_dags)

