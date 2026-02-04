# My Self-Hosting Services

A Docker-based homelab infrastructure for self-hosting services with centralized authentication, automated SSL certificate management, and Cloudflare integration. Services are deployed as Portainer stacks, providing a clean separation of concerns and easy management through a web UI.

The infrastructure uses Traefik as a reverse proxy with automatic HTTPS via Let's Encrypt (DNS challenge), Authentik for SSO and forward authentication, and Cloudflare for DNS management and DDoS protection.

## Main Features

**Centralized Authentication**

- **Authentik SSO**: Enterprise-grade identity provider with forward authentication
- Single sign-on across all services via Traefik middleware
- OAuth2/OIDC support for service integrations

**Network & Reverse Proxy**

- **Traefik v3**: Modern reverse proxy with automatic service discovery
- Automatic HTTPS with Let's Encrypt (DNS-01 challenge via Cloudflare)
- Wildcard SSL certificates for `*.${DOMAIN}`
- HTTP to HTTPS redirect on all services

**Cloudflare Integration**

- **traefik-cloudflare-companion**: Automatic DNS record management for Traefik services
- **cloudflare-ddns**: Dynamic DNS updates for home IP changes
- Cloudflare proxy for DDoS protection and CDN caching

**Infrastructure Management**

- **Portainer**: Web UI for managing Docker stacks and containers
- Google Drive Desktop sync for `$HOME/docker_volumes` backup
- Stack-based deployment (init, network, authentik, services)
- Environment variable configuration via `.env`

**Services**

- **n8n**: Workflow automation with task runners
- **OpenClaw**: Personal AI assistant across messaging channels
- **Monolithic backend**: Supporting microservices for other applications

## How to Setup

1. [Prerequisites](docs/prerequisites.md) - Install Docker and Docker Compose
2. [Cloudflare](docs/cloudflare.md) - Configure Cloudflare DNS, API tokens, and SSL/TLS
3. [Portainer](docs/portainer.md) - Configure environment variables and run Portainer
4. [Network](docs/network.md) - Deploy Traefik reverse proxy and Cloudflare DNS automation
5. [Authentication](docs/authentication.md) - Set up Authentik SSO and forward authentication

### Service Stack Deployment

1. [n8n](docs/n8n.md) - Workflow automation with task runners
2. [OpenClaw](docs/openclaw.md) - Personal AI assistant
3. [Monolithic](docs/monolithic.md) - Monolithic backend service

### Optional

- [Docker Volumes Backup](docs/docker-volumes-backup.md) - Back up persistent data with Google Drive Desktop
