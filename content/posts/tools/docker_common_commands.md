---
title: "Docker common commands"
summary: "Docker command cheatsheet"
date: 2025-12-25
tags: ["Docker"]
author: ["JC"]
draft: true
weight: 0
ShowToc: true
---

# Postgres pgvector:
```bash{linenos=true}
docker pull pgvector/pgvector:pg16
docker run --name testdb \
--expose 5432 \
-p 5432:5432 \
-e POSTGRES_USER=user \
-e POSTGRES_PASSWORD=1234 \
-e POSTGRES_HOST=localhost \
-e POSTGRES_DB=L2DB \
-d pgvector/pgvector:pg16
```

# Postgres:



# MySQL
``` bash{linenos=true}
docker pull mysql:8.0
docker run --name mysqldb \
--expose 3306 \
-p 3306:3306 \
-e MYSQL_USER=user \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_PASSWORD=1234 \
-e MYSQL_DATABASE=db \
-d mysql:8.0
```
