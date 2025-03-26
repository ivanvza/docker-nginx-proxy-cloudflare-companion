# docker-nginx-proxy-cloudflare-companion

[![Paypal Donate](https://img.shields.io/badge/donate-paypal-00457c.svg?logo=paypal&style=flat-square)](https://www.paypal.com/donate/?hosted_button_id=MJQ8TQ9MEWAQE)

## About

This Docker container automatically updates Cloudflare DNS records when containers start or update. It's particularly useful for Unraid users who want to maintain their DNS records across container restarts or when moving containers between different systems. The container supports multiple zones and can work with Nginx Proxy Manager (NPM) for automatic proxy host configuration.

## Features

- Automatic Cloudflare DNS record updates
- Support for multiple Cloudflare zones
- Integration with Nginx Proxy Manager (NPM)
- Docker Swarm mode support
- Automatic proxy host creation in NPM
- Support for multiple domains per container
- Debug logging for troubleshooting
- Docker secrets support for sensitive data

## Prerequisites

1. Unraid server with Docker installed
2. Cloudflare account with:
   - API token (recommended) or email/token combination
   - Domain(s) configured
   - Zone ID(s) for your domain(s)
3. Nginx Proxy Manager (NPM) container (optional, for proxy host management)
4. Docker containers with proper labels

## Installation

### Using Unraid Community Applications

1. Open Unraid WebUI
2. Go to "Apps" tab
3. Search for "nginx-proxy-cloudflare-companion"
4. Click "Install"
5. Configure the container using the template

### Manual Installation

1. Go to "Docker" tab in Unraid
2. Click "Add Container"
3. Click "Advanced View"
4. Fill in the following details:
   - Repository: `tiredofit/nginx-proxy-cloudflare-companion:latest`
   - Name: `cloudflare-companion` (or your preferred name)
   - Network Type: `Bridge`

## Configuration

### Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `CF_TOKEN` | Cloudflare API Token (preferred) | `your-api-token` |
| `CF_EMAIL` | Cloudflare Email (if not using API token) | `your@email.com` |
| `TARGET_DOMAIN` | Your main domain that containers will point to | `home.example.com` |
| `DOMAIN1` | First domain to manage | `app1.example.com` |
| `DOMAIN1_ZONE_ID` | Cloudflare Zone ID for DOMAIN1 | `abc123def456` |
| `DOMAIN1_PROXIED` | Whether to proxy DOMAIN1 through Cloudflare | `TRUE` |

### Optional Environment Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `DEFAULT_TTL` | TTL for DNS records | `1` | `1` |
| `REFRESH_ENTRIES` | Update existing records | `TRUE` | `TRUE` |
| `SWARM_MODE` | Enable Docker Swarm mode | `FALSE` | `FALSE` |
| `CONTAINER_LOG_LEVEL` | Logging level | `INFO` | `DEBUG` |
| `DEFAULT_PORT` | Default port for NPM proxy hosts | `8080` | `8080` |

### Nginx Proxy Manager Integration

To enable automatic proxy host creation in NPM, add these variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `NPM_URL` | NPM API URL | `http://npm:8181` |
| `NPM_USERNAME` | NPM admin username | `admin@example.com` |
| `NPM_PASSWORD` | NPM admin password | `your-password` |
| `DEFAULT_PROXY_IP` | Default IP for proxy hosts | `192.168.1.100` |

### Container Labels

Add these labels to your containers to enable automatic DNS and proxy host creation:

```yaml
labels:
  - "npm.frontend.rule=app1.example.com"  # Single domain
  # or
  - "npm.frontend.rule=app1.example.com,app2.example.com"  # Multiple domains
```

### Example Docker Compose Configuration

```yaml
version: '3'
services:
  cloudflare-companion:
    image: tiredofit/nginx-proxy-cloudflare-companion:latest
    container_name: cloudflare-companion
    environment:
      - CF_TOKEN=your-api-token
      - TARGET_DOMAIN=home.example.com
      - DOMAIN1=app1.example.com
      - DOMAIN1_ZONE_ID=abc123def456
      - DOMAIN1_PROXIED=TRUE
      - NPM_URL=http://npm:8181
      - NPM_USERNAME=admin@example.com
      - NPM_PASSWORD=your-password
      - DEFAULT_PROXY_IP=192.168.1.100
      - CONTAINER_LOG_LEVEL=DEBUG
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    restart: unless-stopped
```

### Example Application Container

```yaml
version: '3'
services:
  myapp:
    image: your-app-image
    container_name: myapp
    labels:
      - "npm.frontend.rule=app1.example.com"
    ports:
      - "8080:8080"
    restart: unless-stopped
```

## Usage

1. Start the container
2. Add the `npm.frontend.rule` label to your application containers
3. The container will automatically:
   - Create/update Cloudflare DNS records
   - Create proxy hosts in NPM (if configured)
   - Handle container restarts and updates

## Troubleshooting

### Enable Debug Logging

Set `CONTAINER_LOG_LEVEL=DEBUG` to see detailed logs about:
- DNS record creation/updates
- NPM proxy host creation
- Network configuration
- Container discovery

### Common Issues

1. **DNS Records Not Updating**
   - Check Cloudflare API token permissions
   - Verify zone IDs are correct
   - Check container logs for errors

2. **NPM Proxy Hosts Not Creating**
   - Verify NPM credentials
   - Check NPM API URL is accessible
   - Ensure container labels are correct

3. **Wrong IP Address Used**
   - Set `DEFAULT_PROXY_IP` to your server's IP
   - Check network configuration
   - Verify container networking mode

## Maintenance

### Updating the Container

1. Go to "Docker" tab in Unraid
2. Find the container
3. Click "Check for Updates"
4. Click "Update" if available

### Backup

No specific backup is needed as the container only manages DNS records and proxy hosts. However, it's recommended to:
- Document your environment variables
- Save your container configurations
- Keep a record of your Cloudflare zone IDs

## Support

- [GitHub Issues](https://github.com/ivanvza/docker-nginx-proxy-cloudflare-companion/issues)
- [Unraid Forums](https://forums.unraid.net/)

## License

MIT. See [LICENSE](LICENSE) for more details.

## References

- [Cloudflare API Documentation](https://api.cloudflare.com/)
- [Nginx Proxy Manager Documentation](https://nginxproxymanager.com/)
- [Docker Documentation](https://docs.docker.com/)
