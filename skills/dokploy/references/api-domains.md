# Domains API Reference

## domain.one (GET)

Get domain details.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `domainId` | string | yes |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/domain.one?domainId=DOMAIN_ID" | jq .
```

## domain.byApplicationId (GET)

List all domains for an application.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `applicationId` | string | yes |

**Example:**
```bash
curl -s -H "x-api-key: $DOKPLOY_API_KEY" \
  "$DOKPLOY_API_URL/api/domain.byApplicationId?applicationId=APP_ID" | jq .
```

## domain.byComposeId (GET)

List all domains for a compose service.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `composeId` | string | yes |

## domain.create (POST)

Add a domain to an application or compose service.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `host` | string | yes | Domain hostname (e.g., `app.example.com`) |
| `applicationId` | string | conditional | Required if attaching to an application |
| `composeId` | string | conditional | Required if attaching to a compose service |
| `path` | string | no | URL path (default: `/`) |
| `port` | number | no | Container port to route to |
| `https` | boolean | no | Enable HTTPS |
| `certificateType` | string | no | `letsencrypt`, `none`, or `custom` |
| `customCertResolver` | string | no | Custom cert resolver name |
| `serviceName` | string | no | Service name (for compose with multiple services) |
| `domainType` | string | no | Domain type |
| `uniqueConfigKey` | number | no | Unique config key |

**Example:**
```bash
curl -s -X POST \
  -H "x-api-key: $DOKPLOY_API_KEY" \
  -H "Content-Type: application/json" \
  "$DOKPLOY_API_URL/api/domain.create" \
  -d '{
    "host": "myapp.example.com",
    "applicationId": "APP_ID",
    "port": 3000,
    "https": true,
    "certificateType": "letsencrypt"
  }' | jq .
```

## domain.update (POST)

Update a domain's configuration.

**Parameters:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `domainId` | string | yes | The domain ID |
| `host` | string | no | New hostname |
| `path` | string | no | URL path |
| `port` | number | no | Container port |
| `https` | boolean | no | Enable HTTPS |
| `certificateType` | string | no | Certificate type |
| `customCertResolver` | string | no | Custom cert resolver |
| `serviceName` | string | no | Service name |

## domain.delete (POST)

Remove a domain.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `domainId` | string | yes |

## domain.generateDomain (POST)

Auto-generate a domain name for an application.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `appName` | string | yes |

## domain.validateDomain (POST)

Validate a domain configuration.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `domain` | string | yes |

## domain.canGenerateTraefikMeDomains (GET)

Check if Traefik.me domains can be generated for a server.

**Parameters:**
| Param | Type | Required |
|-------|------|----------|
| `serverId` | string | yes |
