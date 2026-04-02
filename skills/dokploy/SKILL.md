---
name: dokploy
description: "Manage Dokploy infrastructure: projects, applications, databases, domains, compose services, deployments, and servers via the Dokploy REST API. Use whenever the user mentions dokploy, deploying apps, managing servers, creating databases, adding domains, docker compose deployments, checking deployment status/logs, or any PaaS infrastructure management. Even if 'dokploy' isn't mentioned explicitly, trigger when the context involves their self-hosted deployment platform."
---

# Dokploy Infrastructure Management

Dokploy is a self-hosted PaaS. All management happens through its REST API using curl. No CLI tools or bash scripts — construct curl commands directly.

## Authentication & Request Pattern

**Environment variables** (must be set):
- `DOKPLOY_API_URL` — base URL of the Dokploy instance (e.g., `https://dokploy.example.com`)
- `DOKPLOY_API_KEY` — API token generated from Dokploy UI: Settings → Profile → API/CLI Section

**All endpoints** follow the pattern: `$DOKPLOY_API_URL/api/{router}.{procedure}`

### GET request (reads)

```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/{endpoint}?{param}={value}"
```

Parameters are flat query params. Do NOT use tRPC `?input=` encoding — it doesn't work on the OpenAPI surface.

### POST request (all mutations)

```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/{endpoint}" \
  -d '{"param": "value"}'
```

Every mutation (create, update, remove, deploy, start, stop, reload) uses POST. Never use PUT, PATCH, or DELETE.

Always pipe output through `jq` for readability: `| jq .`

### Response Formats

Not all endpoints return JSON objects. Be aware of these patterns:
- **`application.update`**, **`application.saveEnvironment`**, and other mutation endpoints may return a bare `true` instead of the updated object. Do not pipe these through `jq` with object filters — use `jq .` or skip `jq` entirely.
- **`application.deploy`** returns empty output on success.
- **GET endpoints** (`*.one`, `*.all`) return full JSON objects.
- When in doubt, don't assume the response shape — capture raw output first.

## Critical: The Resource Hierarchy

```
Project
  └── Environment (default: "production")
        ├── Applications
        ├── Databases (postgres, mysql, mariadb, mongo, redis)
        └── Compose services
```

- Every project has at least one environment. The default is named "production" with `isDefault: true`.
- **Creating any resource requires `environmentId`, NOT `projectId`.**
- To get `environmentId`: fetch the project with `project.one`, then read `.environments[0].environmentId` (or find the environment by name).

This is the most common source of errors. Always fetch the environment first.

## Quick Reference

### Projects & Environments

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| List all projects | `project.all` | GET | — |
| Get project details | `project.one` | GET | `projectId` |
| Create project | `project.create` | POST | `name`, `description` |
| Update project | `project.update` | POST | `projectId`, `name`, `description` |
| Delete project | `project.remove` | POST | `projectId` |

### Applications

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| Get app details | `application.one` | GET | `applicationId` |
| Create app | `application.create` | POST | `name`, `environmentId` |
| Update app | `application.update` | POST | `applicationId`, ... |
| Delete app | `application.delete` | POST | `applicationId` |
| Deploy app | `application.deploy` | POST | `applicationId` |
| Start app | `application.start` | POST | `applicationId` |
| Stop app | `application.stop` | POST | `applicationId` |
| Redeploy app | `application.redeploy` | POST | `applicationId` |
| Save env vars | `application.saveEnvironment` | POST | `applicationId`, `env`, `buildArgs`, `buildSecrets`, `createEnvFile` |
| Set build type | `application.saveBuildType` | POST | `applicationId`, `buildType`, ... |

### Databases

All database types share the same operation pattern. Replace `{type}` with: `postgres`, `mysql`, `mariadb`, `mongo`, `redis`.

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| Get DB details | `{type}.one` | GET | `{type}Id` |
| Create DB | `{type}.create` | POST | `name`, `environmentId`, `databasePassword`, ... |
| Delete DB | `{type}.remove` | POST | `{type}Id` |
| Deploy DB | `{type}.deploy` | POST | `{type}Id` |
| Start DB | `{type}.start` | POST | `{type}Id` |
| Stop DB | `{type}.stop` | POST | `{type}Id` |

Note: Each type uses its own ID field name: `postgresId`, `mysqlId`, `mariadbId`, `mongoId`, `redisId`.

### Domains

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| Get domain | `domain.one` | GET | `domainId` |
| Domains for app | `domain.byApplicationId` | GET | `applicationId` |
| Create domain | `domain.create` | POST | `host`, `applicationId`, `port` |
| Update domain | `domain.update` | POST | `domainId`, `host`, ... |
| Delete domain | `domain.delete` | POST | `domainId` |

### Compose

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| Get compose | `compose.one` | GET | `composeId` |
| Create compose | `compose.create` | POST | `name`, `environmentId` |
| Update compose | `compose.update` | POST | `composeId`, ... |
| Delete compose | `compose.delete` | POST | `composeId`, `deleteVolumes` |
| Deploy compose | `compose.deploy` | POST | `composeId` |
| Start compose | `compose.start` | POST | `composeId` |
| Stop compose | `compose.stop` | POST | `composeId` |
| List services | `compose.loadServices` | GET | `composeId` |

### Deployments

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| List deployments | `deployment.all` | GET | `applicationId` |
| By type | `deployment.allByType` | GET | `id`, `type` |

`type` values: `application`, `compose`, `server`, `schedule`, `previewDeployment`, `backup`, `volumeBackup`

### Servers

| Task | Endpoint | Method | Key Params |
|------|----------|--------|------------|
| List servers | `server.all` | GET | — |
| Get server | `server.one` | GET | `serverId` |
| Create server | `server.create` | POST | `name`, `ipAddress`, `port`, `username`, `sshKeyId` |
| Delete server | `server.remove` | POST | `serverId` |

## Common Workflows

### List all projects with their resources

```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.all" | jq .
```

The response includes nested environments with all applications, databases, and compose services summarized.

### Deploy an existing application

```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/application.deploy" \
  -d '{"applicationId": "THE_APP_ID"}' | jq .
```

### Create a new app in an existing project

Step 1 — Get the project's environmentId:
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.one?projectId=PROJECT_ID" \
  | jq '.environments[] | select(.isDefault) | .environmentId'
```

Step 2 — Create the application:
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/application.create" \
  -d '{"name": "my-app", "environmentId": "ENV_ID"}' | jq .
```

### Create a full project from scratch

Step 1 — Create the project:
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/project.create" \
  -d '{"name": "My Project", "description": "Project description"}' | jq .
```

Step 2 — Get the new project's environmentId from the response, or fetch it:
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.all" \
  | jq '.[] | select(.name == "My Project") | .environments[0].environmentId'
```

Step 3 — Create resources using that environmentId (apps, databases, compose).

### Check deployment status

For applications:
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/deployment.all?applicationId=APP_ID" | jq '.[0]'
```

For compose services:
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/deployment.allByType?id=COMPOSE_ID&type=compose" | jq '.[0]'
```

### Set environment variables on an app

Env vars are set as a single newline-separated string block, not key-value pairs.

**Required fields**: `buildArgs`, `buildSecrets`, and `createEnvFile` are mandatory (use empty strings and `false` as defaults):
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/application.saveEnvironment" \
  -d '{
    "applicationId": "APP_ID",
    "env": "DATABASE_URL=postgres://...\nREDIS_URL=redis://...\nNODE_ENV=production",
    "buildArgs": "",
    "buildSecrets": "",
    "createEnvFile": false
  }'
```

## Important Notes

- `project.all` returns a comprehensive overview — use it to discover IDs before drilling down.
- Deployment is asynchronous. After triggering deploy, poll `deployment.all` or `deployment.allByType` to check status.
- Destructive operations (`remove`, `delete`) cannot be undone. Always confirm with the user first.
- The Swagger UI is available at `$DOKPLOY_API_URL/swagger` for interactive API exploration (admin only).

## Reference Files

For full endpoint details with all parameters:

| Need details about... | Read |
|-----------------------|------|
| Projects & environments | `references/api-projects.md` |
| Applications (CRUD, deploy, build types, git providers) | `references/api-applications.md` |
| Databases (PostgreSQL, MySQL, MariaDB, MongoDB, Redis) | `references/api-databases.md` |
| Domains (create, update, certificates) | `references/api-domains.md` |
| Docker Compose services | `references/api-compose.md` |
| Deployment history & logs | `references/api-deployments.md` |
| Server management | `references/api-servers.md` |
