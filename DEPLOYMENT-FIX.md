# Fixing 502 Bad Gateway Errors in Plane Deployment

The 502 errors indicate the proxy can't reach the API backend. Here's how to fix it:

## Quick Fix Steps

### 1. Use the Updated Docker Compose File
Use `docker-compose.production.yml` with the corrected environment variables.

### 2. Set Environment Variables
Create or update `.env.production` with your domain:

```bash
# Critical settings for your domain
APP_DOMAIN=plane.team-workspace.us
WEB_URL=https://plane.team-workspace.us
NEXT_PUBLIC_API_BASE_URL=https://plane.team-workspace.us/api

# MUST CHANGE these for production
SECRET_KEY=<generate-new-32-char-key>
POSTGRES_PASSWORD=<strong-password>
RABBITMQ_PASSWORD=<strong-password>
AWS_ACCESS_KEY_ID=<minio-access-key>
AWS_SECRET_ACCESS_KEY=<minio-secret-key>
```

### 3. Deploy with Correct Configuration

```bash
# Using docker-compose directly
docker compose -f docker-compose.production.yml --env-file .env.production up -d

# Or set environment first
export $(cat .env.production | xargs)
docker compose -f docker-compose.production.yml up -d
```

## For Coolify Deployment

### Environment Variables to Set in Coolify:

```bash
# Domain Configuration (CRITICAL)
WEB_URL=https://plane.team-workspace.us
NEXT_PUBLIC_API_BASE_URL=https://plane.team-workspace.us/api
APP_DOMAIN=plane.team-workspace.us

# Security (MUST CHANGE)
SECRET_KEY=<generate-with-openssl-rand-hex-32>
POSTGRES_PASSWORD=<strong-password>
RABBITMQ_PASSWORD=<strong-password>

# Storage
USE_MINIO=1
AWS_ACCESS_KEY_ID=<change-from-default>
AWS_SECRET_ACCESS_KEY=<change-from-default>

# SSL
SSL=true
CERT_EMAIL=your-email@domain.com
```

## Common Issues and Solutions

### Issue 1: API Returns 502
**Cause**: Frontend can't reach backend API
**Solution**: Ensure `NEXT_PUBLIC_API_BASE_URL` points to `/api` path on same domain

### Issue 2: Services Not Starting
**Cause**: Database not initialized
**Solution**: Check migrator container logs: `docker logs plane-migrator`

### Issue 3: File Uploads Failing
**Cause**: MinIO not configured
**Solution**: Verify MinIO credentials match in all services

## Verify Deployment

1. **Check all services are running:**
```bash
docker compose -f docker-compose.production.yml ps
```

2. **Check API health:**
```bash
curl https://plane.team-workspace.us/api/
```

3. **Check logs for errors:**
```bash
docker logs plane-api
docker logs plane-web
docker logs plane-proxy
```

## Nginx Proxy Configuration

The proxy service automatically routes:
- `/` → web container (port 3000)
- `/api` → api container (port 8000)
- `/god-mode` → admin container
- `/spaces` → space container
- `/live` → live container

## Important Notes

1. **Single Domain Setup**: All services run under one domain with path-based routing
2. **API Path**: API is accessed at `/api` not separate port
3. **SSL Termination**: Handled by proxy container
4. **Internal Communication**: Services communicate via Docker network, not external URLs

## Troubleshooting Commands

```bash
# Restart all services
docker compose -f docker-compose.production.yml restart

# View real-time logs
docker compose -f docker-compose.production.yml logs -f

# Check specific service
docker logs plane-api --tail 100

# Rebuild and restart
docker compose -f docker-compose.production.yml up -d --build

# Complete reset
docker compose -f docker-compose.production.yml down
docker compose -f docker-compose.production.yml up -d
```