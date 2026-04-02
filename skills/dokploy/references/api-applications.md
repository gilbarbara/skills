# Applications API Reference

## application.one (GET)

Get detailed application information.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `applicationId` | string | yes |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/application.one?applicationId=APP_ID" | jq .
```

## application.create (POST)

Create a new application in an environment.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name |
| `environmentId` | string | yes | Environment to create in (get from project.one) |
| `appName` | string | no | Internal app name (auto-generated if omitted) |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/application.create" \
  -d '{"name": "My App", "environmentId": "ENV_ID"}' | jq .
```

## application.update (POST)

Update application settings. Only include the fields you want to change.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `applicationId` | string | yes | The application ID |
| `name` | string | no | Display name |
| `description` | string | no | Description |
| `memoryLimit` | number | no | Memory limit in bytes |
| `memoryReservation` | number | no | Memory reservation in bytes |
| `cpuLimit` | number | no | CPU limit |
| `cpuReservation` | number | no | CPU reservation |
| `replicas` | number | no | Number of replicas |
| `autoDeploy` | boolean | no | Auto-deploy on push |
| `command` | string | no | Custom start command |

## application.delete (POST)

Delete an application. Irreversible.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `applicationId` | string | yes |

## Lifecycle Operations

All take `applicationId` as the only required parameter (POST):

| Endpoint | Description |
|----------|-------------|
| `application.deploy` | Trigger a new deployment |
| `application.redeploy` | Force redeploy |
| `application.start` | Start a stopped application |
| `application.stop` | Stop a running application |
| `application.reload` | Reload application (also requires `appName`) |
| `application.cancelDeployment` | Cancel an in-progress deployment |
| `application.killBuild` | Kill a running build |
| `application.cleanQueues` | Clear deployment queue |
| `application.markRunning` | Mark application as running |

## Environment Variables

### application.saveEnvironment (POST)

Set environment variables. Env vars are a single newline-separated string, not key-value pairs.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `applicationId` | string | yes | The application ID |
| `env` | string | yes | Newline-separated env vars: `KEY=value\nKEY2=value2` |
| `buildArgs` | string | no | Build arguments (same format) |
| `buildSecrets` | string | no | Build secrets (same format) |
| `createEnvFile` | boolean | no | Whether to create a .env file |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/application.saveEnvironment" \
  -d '{
    "applicationId": "APP_ID",
    "env": "DATABASE_URL=postgres://user:pass@host:5432/db\nNODE_ENV=production\nPORT=3000"
  }' | jq .
```

## Build Type Configuration

### application.saveBuildType (POST)

Configure how the application is built.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `applicationId` | string | yes | The application ID |
| `buildType` | string | yes | One of: `nixpacks`, `dockerfile`, `docker`, `heroku`, `paketo`, `railpack`, `static` |
| `dockerfile` | string | no | Dockerfile path (for `dockerfile` type) |
| `dockerContextPath` | string | no | Docker build context |
| `dockerBuildStage` | string | no | Multi-stage build target |
| `herokuVersion` | string | no | Heroku buildpack version |
| `railpackVersion` | string | no | Railpack version |

## Git Provider Configuration

### application.saveGithubProvider (POST)

Connect to a GitHub repository.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `applicationId` | string | yes | The application ID |
| `repository` | string | yes | Repository name |
| `branch` | string | yes | Branch to deploy |
| `owner` | string | yes | GitHub user/org |
| `buildPath` | string | no | Path to build from |
| `githubId` | string | no | GitHub integration ID |
| `triggerType` | string | no | Trigger type |

### Other Git providers

Same pattern with provider-specific fields:
- `application.saveGitlabProvider` — `gitlabBranch`, `gitlabBuildPath`, `gitlabOwner`, `gitlabRepository`, `gitlabId`
- `application.saveBitbucketProvider` — `bitbucketBranch`, `bitbucketOwner`, `bitbucketRepository`
- `application.saveGiteaProvider` — `giteaBranch`, `giteaBuildPath`, `giteaOwner`, `giteaRepository`, `giteaId`
- `application.saveGitProvider` — `customGitBranch`, `customGitUrl`, `customGitBuildPath`, `watchPaths`

### application.saveDockerProvider (POST)

Deploy from a Docker image.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `applicationId` | string | yes | The application ID |
| `dockerImage` | string | yes | Image name (e.g., `nginx:latest`) |
| `username` | string | no | Registry username |
| `password` | string | no | Registry password |
| `registryUrl` | string | no | Custom registry URL |

### application.disconnectGitProvider (POST)

Disconnect the current git provider. Takes `applicationId`.

## Other Operations

| Endpoint | Method | Params | Description |
|----------|--------|--------|-------------|
| `application.move` | POST | `applicationId`, `targetEnvironmentId` | Move app to another environment |
| `application.refreshToken` | POST | `applicationId` | Refresh webhook token |
| `application.readTraefikConfig` | GET | `applicationId` | Read Traefik configuration |
| `application.updateTraefikConfig` | POST | `applicationId`, `traefikConfig` | Update Traefik configuration |
| `application.readAppMonitoring` | GET | `appName` | Read monitoring data |
