# Deployments API Reference

## deployment.all (GET)

List deployments for an application.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `applicationId` | string | yes |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/deployment.all?applicationId=APP_ID" | jq .
```

**Response shape:**
```json
[
  {
    "deploymentId": "string",
    "title": "string",
    "status": "running" | "done" | "error",
    "logPath": "string",
    "applicationId": "string",
    "createdAt": "ISO-8601",
    "description": "string"
  }
]
```

Deployments are returned most-recent-first. The first item `.[0]` is the latest deployment.

## deployment.allByCompose (GET)

List deployments for a compose service.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `composeId` | string | yes |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/deployment.allByCompose?composeId=COMPOSE_ID" | jq .
```

## deployment.allByServer (GET)

List deployments for a server.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `serverId` | string | yes |

## deployment.allByType (GET)

List deployments by resource type. Useful when you don't know which specific endpoint to use.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Resource ID (applicationId, composeId, serverId, etc.) |
| `type` | string | yes | One of: `application`, `compose`, `server`, `schedule`, `previewDeployment`, `backup`, `volumeBackup` |

**Example:**
```bash
# Get deployments for a compose service
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/deployment.allByType?id=COMPOSE_ID&type=compose" | jq .
```

## deployment.killProcess (POST)

Kill a running deployment process.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `deploymentId` | string | yes |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/deployment.killProcess" \
  -d '{"deploymentId": "DEPLOY_ID"}' | jq .
```

## Deployment Status Values

| Status | Meaning |
|--------|---------|
| `running` | Deployment is in progress |
| `done` | Deployment completed successfully |
| `error` | Deployment failed |

## Checking Latest Deployment Status

```bash
# Get the status of the most recent deployment for an app
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/deployment.all?applicationId=APP_ID" \
  | jq '.[0] | {status, title, createdAt}'
```
