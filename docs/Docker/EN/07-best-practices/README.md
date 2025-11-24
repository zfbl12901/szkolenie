# 7. Docker Best Practices

## 🎯 Objectives

- Secure containers
- Optimize performance
- Organize projects
- Maintain images
- Manage resources

## 📋 Table of Contents

1. [Security](#security)
2. [Performance](#performance)
3. [Organization](#organization)
4. [Maintenance](#maintenance)
5. [Resources](#resources)

---

## Security

### Use Official Images

```dockerfile
# Good
FROM python:3.11-slim

# Avoid
FROM random-user/python:latest
```

### Don't Use Root

```dockerfile
# Create non-root user
RUN useradd -m appuser
USER appuser
```

### Limit Privileges

```bash
# Don't use --privileged
docker run --privileged my-container  # Avoid

# Use specific capabilities if needed
docker run --cap-add NET_ADMIN my-container
```

### Secrets

```yaml
# docker-compose.yml
services:
  app:
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/password.txt
```

---

## Performance

### Use .dockerignore

```
__pycache__
*.pyc
.git
.env
node_modules
*.log
```

### Optimize Layers

```dockerfile
# Bad
RUN apt update
RUN apt install -y python3
RUN apt clean

# Good
RUN apt update && \
    apt install -y python3 && \
    apt clean
```

### Build Cache

**Instruction order:**

```dockerfile
# First dependencies (change little)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Then code (changes often)
COPY . .
```

---

## Organization

### Project Structure

```
my-project/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── requirements.txt
├── src/
│   └── app.py
└── data/
    └── data.csv
```

### Image Tags

```bash
# Semantic tags
docker build -t my-app:1.0.0 .
docker build -t my-app:latest .

# Environment tags
docker build -t my-app:dev .
docker build -t my-app:prod .
```

---

## Maintenance

### Clean Resources

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Clean everything
docker system prune -a
```

### Update Images

```bash
# Update an image
docker pull python:3.11

# Rebuild
docker-compose build --no-cache
docker-compose up
```

---

## Resources

### Limit Resources

```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

---

## 📊 Key Takeaways

1. **Security** : Official images, non-root
2. **Performance** : Optimize layers, cache
3. **Organization** : Clear structure, tags
4. **Maintenance** : Clean regularly
5. **Resources** : Limit usage

## 🔗 Next Module

Proceed to module [8. Practical Projects](./08-projets/README.md) to create complete projects.

