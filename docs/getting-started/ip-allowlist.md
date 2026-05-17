---
icon: octagon-check
---

# IP allowlist

Restrict API key usage to specific IP addresses. When an allowlist is configured, only requests from listed IPs or CIDR ranges can use your API keys. Requests from unlisted IPs receive a `403 Forbidden` response.

This is useful for enterprise environments where API access should be limited to office networks, datacenters, or specific server IPs.

## How it works

* The allowlist is **per account** -- it applies to all your projects and API keys.
* An **empty allowlist** (default) means all IPs are allowed.
* A **populated allowlist** means only listed IPs can make API requests.
* The check runs before rate limiting and filtering, so blocked IPs do not consume quota.
* Supports **IPv4**, **IPv6**, and **CIDR notation** (e.g. `10.0.0.0/8`).
* Maximum **50 entries** per account.

## Configuring the allowlist

### Via the dashboard

Go to **Settings > IP Allowlist** to add or remove IP entries. Changes take effect immediately.

### Via the API

#### Get current allowlist

```bash
curl https://api.collieai.com/auth/ip-allowlist \
  -H "Authorization: Bearer <session_token>"
```

**Response:**

```json
{
  "ips": ["192.168.1.0/24", "10.0.0.1"],
  "enabled": true
}
```

#### Set the allowlist

Replace the entire allowlist. Send an empty list to disable restrictions.

```bash
curl -X PUT https://api.collieai.com/auth/ip-allowlist \
  -H "Authorization: Bearer <session_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "ips": ["203.0.113.0/24", "198.51.100.42"]
  }'
```

#### Disable the allowlist

```bash
curl -X PUT https://api.collieai.com/auth/ip-allowlist \
  -H "Authorization: Bearer <session_token>" \
  -H "Content-Type: application/json" \
  -d '{"ips": []}'
```

## CIDR notation

Use CIDR ranges to allow entire subnets without listing individual IPs.

| Entry            | Matches                                         |
| ---------------- | ----------------------------------------------- |
| `192.168.1.1`    | Exactly `192.168.1.1`                           |
| `192.168.1.0/24` | `192.168.1.0` through `192.168.1.255` (256 IPs) |
| `10.0.0.0/8`     | All `10.x.x.x` addresses (16M IPs)              |
| `2001:db8::/32`  | IPv6 range                                      |

## Error response

When a request comes from an IP not in the allowlist:

```json
{
  "error": {
    "message": "IP address not allowed",
    "type": "permission_error",
    "code": 403
  }
}
```

## Reverse proxy setup

If CollieAI runs behind a reverse proxy or load balancer (Nginx, AWS ALB, Cloudflare), set the `FORWARDED_ALLOW_IPS` environment variable so that CollieAI reads the real client IP from the `X-Forwarded-For` header instead of the proxy's IP.

```bash
# Trust a specific proxy IP
FORWARDED_ALLOW_IPS=10.0.0.1

# Trust a subnet (e.g. internal LB range)
FORWARDED_ALLOW_IPS=10.0.0.0/8

# Trust all proxies (use only if you control the entire network path)
FORWARDED_ALLOW_IPS=*
```

Without this setting, the allowlist evaluates the proxy's IP, which will reject legitimate traffic.

## Next steps

* [Authentication](/broken/pages/1845ab390e5ce0610c3639336c3607da0944e5f1) -- sign-in methods and password management.
* [API Keys](/broken/pages/7feac2acca8bcde057933e84cdddfb8ff4d6f647) -- manage API keys within projects.
* [Projects](/broken/pages/12493f7755c3bc9f868977b8eb076b8598bb0ddb) -- configure per-project settings like rate limiting.
