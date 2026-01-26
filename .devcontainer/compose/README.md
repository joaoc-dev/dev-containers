# Database Compose Files

Shared Docker Compose configurations for common databases used across dev container templates.

## Why Docker Compose for databases/services?

We use Docker Compose fragments here because they keep **databases and infra as separate containers** (closer to production), while staying **mix-and-match** across templates.

- **Consistency**: same service names (`postgres`, `redis`, etc.) and connection details across all templates
- **Separation of concerns**: DB lifecycle/data is separate from the devcontainer image and rebuilds
- **Multi-service stacks**: easy to run Postgres + Redis + RabbitMQ together (Compose is the Dev Containers multi-container mechanism)
- **Faster devcontainer builds**: DB images aren’t baked into the devcontainer image

Dev Container **Features** are still great for installing **tools inside the devcontainer** (e.g. `psql`, `redis-cli`, `mongosh`), although you might prefer running these tools on the host anyway.

## Available Services

| Service | File | Host Ports | Default Credentials |
|---------|------|------------|---------------------|
| **SQL Server 2025** | `sqlserver.compose.yml` | 11433 | User: `sa`<br>Password: `YourStrong!Passw0rd` |
| **PostgreSQL 18** | `postgres.compose.yml` | 15432 | User: `postgres`<br>Password: `postgres`<br>DB: `devdb` |
| **MongoDB 8** | `mongo.compose.yml` | 37017 | User: `admin`<br>Password: `password`<br>DB: `devdb` |
| **Redis 8** | `redis.compose.yml` | 16379 | Password: `redispassword` |
| **RabbitMQ 4.2** | `rabbitmq.compose.yml` | 15672 (AMQP)<br>25672 (Management UI) | User: `admin`<br>Password: `password` |

**Note:** Host ports use a **+10000 offset** to avoid conflicts with locally installed services (e.g. Postgres \(5432 → 15432\)).

## How to Use

### 1. Opt-in services in your template's `devcontainer.json`

Each template has commented examples. Just uncomment the databases you need:

```json
{
  "name": "Your Dev Container",
  
  "dockerComposeFile": [
    "docker-compose.yml"
    // Uncomment databases you need:
    ,"../compose/postgres.compose.yml"
    ,"../compose/redis.compose.yml"
  ],
  "service": "devcontainer",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  
  "features": {
    // ... your features
  }
}
```

### 2. Ensure your template `docker-compose.yml` matches

Your template’s `docker-compose.yml` has a commented `depends_on` section.

- Just uncomment the same databases you uncommented above.

### 3. Rebuild the container

- Press `Ctrl+Shift+P`
- Select **"Dev Containers: Rebuild Container"**
- Wait for the rebuild to complete

### 3. Connect from your application

Services are accessible by their container names:

**SQL Server:**
```
Server: sqlserver,1433  (internal)
User: sa
Password: YourStrong!Passw0rd
```

**PostgreSQL:**
```
Host: postgres
Port: 5432  (internal)
User: postgres
Password: postgres
Database: devdb
```

**MongoDB:**
```
mongodb://admin:password@mongo:27017/devdb?authSource=admin
```

**Redis:**
```
Host: redis
Port: 6379  (internal)
Password: redispassword
```

**RabbitMQ:**
```
Host: rabbitmq
Port: 5672 (AMQP, internal)
User: admin
Password: password
Management UI: http://rabbitmq:15672 (internal)
```

**From Windows/Host:**
- SQL Server: `localhost,11433`
- PostgreSQL: `localhost:15432`
- MongoDB: `localhost:37017`
- Redis: `localhost:16379`
- RabbitMQ: `localhost:15672` (AMQP), `http://localhost:25672` (Management UI)

## Data Persistence

Each service uses named Docker volumes for data persistence:
- `sqlserver-data`
- `postgres-data`
- `mongo-data`
- `mongo-config`
- `redis-data`
- `rabbitmq-data`

**Note:** Docker Compose will prefix these volume names with the Compose project name (often the folder name), e.g. `devcont-study_postgres-data`.

**Data survives container rebuilds!** To reset, delete the volumes:

```bash
docker volume rm devcont-study_postgres-data
```

### Change Ports

If you have conflicts with existing services, edit the port mappings:

```yaml
ports:
  - "5433:5432"  # Map host port 5433 to container port 5432
```

## Health Checks

All services include health checks so your dev container waits for databases to be ready before starting.
