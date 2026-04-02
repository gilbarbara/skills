# Databases API Reference

All database types follow the same operation pattern. The router name matches the database type: `postgres`, `mysql`, `mariadb`, `mongo`, `redis`.

Each type uses its own ID field: `postgresId`, `mysqlId`, `mariadbId`, `mongoId`, `redisId`.

## Common Operations (all types)

| Endpoint | Method | Params | Description |
|----------|--------|--------|-------------|
| `{type}.one` | GET | `{type}Id` | Get database details |
| `{type}.create` | POST | See below | Create database |
| `{type}.update` | POST | `{type}Id`, ... | Update database settings |
| `{type}.remove` | POST | `{type}Id` | Delete database (irreversible) |
| `{type}.deploy` | POST | `{type}Id` | Deploy/redeploy database |
| `{type}.start` | POST | `{type}Id` | Start database |
| `{type}.stop` | POST | `{type}Id` | Stop database |
| `{type}.reload` | POST | `{type}Id` | Reload database |
| `{type}.rebuild` | POST | `{type}Id` | Rebuild database container |
| `{type}.changeStatus` | POST | `{type}Id`, `applicationStatus` | Change status |
| `{type}.saveExternalPort` | POST | `{type}Id`, `externalPort` | Set external port |
| `{type}.saveEnvironment` | POST | `{type}Id`, `env` | Set env vars |
| `{type}.move` | POST | `{type}Id`, `targetEnvironmentId` | Move to another environment |

## Create Parameters by Type

### postgres.create

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in |
| `databaseName` | string | yes | Database name |
| `databaseUser` | string | yes | Database user |
| `databasePassword` | string | yes | Database password |
| `dockerImage` | string | no | Image (default: `postgres:15`) |
| `description` | string | no | Description |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/postgres.create" \
  -d '{
    "name": "Main DB",
    "environmentId": "ENV_ID",
    "databaseName": "myapp",
    "databaseUser": "myuser",
    "databasePassword": "securepassword123"
  }' | jq .
```

### mysql.create

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in |
| `databaseName` | string | yes | Database name |
| `databaseUser` | string | yes | Database user |
| `databasePassword` | string | yes | Database password |
| `databaseRootPassword` | string | no | Root password |
| `dockerImage` | string | no | Image (default: `mysql:8`) |

### mariadb.create

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in |
| `databaseName` | string | yes | Database name |
| `databaseUser` | string | yes | Database user |
| `databasePassword` | string | yes | Database password |
| `databaseRootPassword` | string | no | Root password |
| `dockerImage` | string | no | Image (default: `mariadb:11`) |

### mongo.create

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in |
| `databaseUser` | string | yes | Database user |
| `databasePassword` | string | yes | Database password |
| `dockerImage` | string | no | Image (default: `mongo:6`) |

Note: MongoDB does not take `databaseName` on create.

### redis.create

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in |
| `databasePassword` | string | yes | Password |
| `dockerImage` | string | no | Image (default: `redis:7`) |

Note: Redis does not take `databaseName` or `databaseUser` on create.

## Getting a Database

```bash
# PostgreSQL example
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/postgres.one?postgresId=PG_ID" | jq .

# MySQL example
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/mysql.one?mysqlId=MYSQL_ID" | jq .
```

## Deploying a Database

```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/postgres.deploy" \
  -d '{"postgresId": "PG_ID"}' | jq .
```

## Setting External Port

To make a database accessible from outside Docker:
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/postgres.saveExternalPort" \
  -d '{"postgresId": "PG_ID", "externalPort": 5432}' | jq .
```
