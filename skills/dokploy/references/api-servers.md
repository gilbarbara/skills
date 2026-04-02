# Servers API Reference

## server.all (GET)

List all servers.

**Parameters:** None

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/server.all" | jq .
```

## server.one (GET)

Get server details.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `serverId` | string | yes |

## server.create (POST)

Register a new server.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Server name |
| `description` | string | no | Description |
| `ipAddress` | string | yes | Server IP address |
| `port` | number | yes | SSH port (usually 22) |
| `username` | string | yes | SSH username |
| `sshKeyId` | string | yes | SSH key ID (from ssh-key router) |
| `serverType` | string | no | Server type |

## server.update (POST)

Update server details.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `serverId` | string | yes | The server ID |
| `name` | string | no | Server name |
| `description` | string | no | Description |
| `ipAddress` | string | no | Server IP |
| `port` | number | no | SSH port |
| `username` | string | no | SSH username |
| `sshKeyId` | string | no | SSH key ID |

## server.remove (POST)

Delete a server. Irreversible.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `serverId` | string | yes |

## server.setup (POST)

Run initial setup on a server (installs Docker, configures networking).

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `serverId` | string | yes |

## server.validate (GET)

Check if a server is properly configured and reachable.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `serverId` | string | yes |

## Other Server Endpoints

| Endpoint | Method | Params | Description |
|----------|--------|--------|-------------|
| `server.count` | GET | — | Get total server count |
| `server.withSSHKey` | GET | — | List servers with their SSH keys |
| `server.buildServers` | GET | — | List servers available for builds |
| `server.publicIp` | GET | — | Get the public IP of the Dokploy server |
| `server.security` | GET | `serverId` | Get server security config |
| `server.getDefaultCommand` | GET | `serverId` | Get default command for server |
| `server.getServerTime` | GET | — | Get server time |
| `server.getServerMetrics` | GET | `url`, `token`, `dataPoints` | Get server metrics |
| `server.setupMonitoring` | POST | `serverId`, `metricsConfig` | Configure monitoring |
