# Projects & Environments API Reference

## project.all (GET)

List all projects with their environments and nested resources.

**Parameters:** None

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.all" | jq .
```

**Response shape:**
```json
[
  {
    "projectId": "string",
    "name": "string",
    "description": "string",
    "createdAt": "ISO-8601",
    "organizationId": "string",
    "environments": [
      {
        "environmentId": "string",
        "name": "production",
        "isDefault": true,
        "applications": [{ "applicationId": "...", "name": "...", "appName": "..." }],
        "compose": [{ "composeId": "...", "name": "...", "composeStatus": "..." }],
        "postgres": [{ "postgresId": "..." }],
        "mysql": [{ "mysqlId": "..." }],
        "mariadb": [{ "mariadbId": "..." }],
        "mongo": [{ "mongoId": "..." }],
        "redis": [{ "redisId": "..." }]
      }
    ]
  }
]
```

## project.one (GET)

Get detailed information about a single project.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `projectId` | string | yes | The project ID |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.one?projectId=abc123" | jq .
```

## project.create (POST)

Create a new project. A default "production" environment is created automatically.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Project name |
| `description` | string | no | Project description |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/project.create" \
  -d '{"name": "My Project", "description": "A new project"}' | jq .
```

## project.update (POST)

Update a project's name or description.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `projectId` | string | yes | The project ID |
| `name` | string | no | New name |
| `description` | string | no | New description |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/project.update" \
  -d '{"projectId": "abc123", "name": "New Name"}' | jq .
```

## project.remove (POST)

Delete a project and all its resources. This is irreversible.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `projectId` | string | yes | The project ID |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/project.remove" \
  -d '{"projectId": "abc123"}' | jq .
```

## project.duplicate (POST)

Duplicate a project from an existing environment.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `sourceEnvironmentId` | string | yes | Environment ID to duplicate from |
| `name` | string | yes | Name for the new project |

## Getting the environmentId

The environmentId is required for creating applications, databases, and compose services. To extract it:

```bash
# Get the default environment ID for a project
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.one?projectId=PROJECT_ID" \
  | jq -r '.environments[] | select(.isDefault) | .environmentId'

# Or get all environments for a project
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/project.one?projectId=PROJECT_ID" \
  | jq '.environments[] | {environmentId, name, isDefault}'
```
