# Docker Compose API Reference

## compose.one (GET)

Get compose service details.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `composeId` | string | yes |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/compose.one?composeId=COMPOSE_ID" | jq .
```

## compose.create (POST)

Create a new compose service in an environment.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in |
| `appName` | string | no | Internal name (auto-generated if omitted) |
| `composeType` | string | no | Source type |
| `description` | string | no | Description |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/compose.create" \
  -d '{"name": "My Stack", "environmentId": "ENV_ID"}' | jq .
```

## compose.update (POST)

Update a compose service. Only include fields to change.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `composeId` | string | yes | The compose ID |
| `name` | string | no | Display name |
| `description` | string | no | Description |
| `composeFile` | string | no | Docker Compose YAML content |
| `env` | string | no | Environment variables (newline-separated `KEY=value`) |
| `sourceType` | string | no | Source type |
| `autoDeploy` | boolean | no | Auto-deploy on push |
| `command` | string | no | Custom command |

**Example — update compose file:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/compose.update" \
  -d '{
    "composeId": "COMPOSE_ID",
    "composeFile": "version: \"3.8\"\nservices:\n  web:\n    image: nginx:latest\n    ports:\n      - \"80:80\""
  }' | jq .
```

## compose.delete (POST)

Delete a compose service.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `composeId` | string | yes | The compose ID |
| `deleteVolumes` | boolean | no | Also delete volumes (default: false) |

## Lifecycle Operations

All take `composeId` as the only required parameter (POST):

| Endpoint | Description |
|----------|-------------|
| `compose.deploy` | Deploy the compose stack |
| `compose.redeploy` | Force redeploy |
| `compose.start` | Start stopped services |
| `compose.stop` | Stop running services |
| `compose.cancelDeployment` | Cancel in-progress deployment |
| `compose.killBuild` | Kill a running build |
| `compose.cleanQueues` | Clear deployment queue |

## Git Provider Configuration

Same pattern as applications:
- `compose.saveGithubProvider` — connect to GitHub repo
- `compose.saveGitlabProvider` — connect to GitLab repo
- `compose.saveBitbucketProvider` — connect to Bitbucket repo
- `compose.saveGiteaProvider` — connect to Gitea repo
- `compose.saveGitProvider` — connect to any Git URL
- `compose.disconnectGitProvider` — disconnect current provider

## Other Operations

| Endpoint | Method | Params | Description |
|----------|--------|--------|-------------|
| `compose.loadServices` | GET | `composeId` | List services defined in the compose file |
| `compose.loadMountsByService` | GET | `composeId`, `serviceName` | List volume mounts for a service |
| `compose.getConvertedCompose` | GET | `composeId` | Get the processed compose file |
| `compose.getDefaultCommand` | GET | `composeId` | Get default command |
| `compose.move` | POST | `composeId`, `targetEnvironmentId` | Move to another environment |
| `compose.refreshToken` | POST | `composeId` | Refresh webhook token |
| `compose.fetchSourceType` | POST | `composeId` | Detect source type |
| `compose.randomizeCompose` | POST | `composeId` | Randomize compose config |
| `compose.isolatedDeployment` | POST | `composeId` | Deploy in isolation |

## Templates

| Endpoint | Method | Params | Description |
|----------|--------|--------|-------------|
| `compose.templates` | GET | — | List available compose templates |
| `compose.getTags` | GET | — | List template tags |
| `compose.deployTemplate` | POST | `environmentId`, `id` | Deploy a template |
| `compose.processTemplate` | POST | `base64`, `composeId` | Process a template |
| `compose.import` | POST | `base64`, `composeId` | Import compose file (base64 encoded) |
