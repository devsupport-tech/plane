# Plane Deployment Guide

This guide covers deploying Plane using Docker Compose or Coolify.

## Deployment Options

### Option 1: Using Pre-built Images (Recommended)

Use the production docker-compose file from `deployments/cli/community/docker-compose.yml` which uses pre-built images from `artifacts.plane.so`.

### Option 2: Building from Source

Use the root `docker-compose.yml` to build images from source code.

## Environment Variables

### Core Database Settings
```bash
# PostgreSQL Configuration
POSTGRES_USER=plane              # Database username
POSTGRES_PASSWORD=<secure-pass>  # Database password (CHANGE THIS!)
POSTGRES_DB=plane                 # Database name
POSTGRES_HOST=plane-db            # Database host
POSTGRES_PORT=5432                # Database port
PGDATA=/var/lib/postgresql/data  # PostgreSQL data directory
DATABASE_URL=postgresql://plane:<secure-pass>@plane-db:5432/plane
```

### Redis Configuration
```bash
REDIS_HOST=plane-redis            # Redis host
REDIS_PORT=6379                   # Redis port
REDIS_URL=redis://plane-redis:6379/
```

### RabbitMQ Configuration
```bash
RABBITMQ_HOST=plane-mq            # RabbitMQ host
RABBITMQ_PORT=5672                # RabbitMQ port
RABBITMQ_USER=plane               # RabbitMQ username
RABBITMQ_PASSWORD=<secure-pass>  # RabbitMQ password (CHANGE THIS!)
RABBITMQ_VHOST=plane              # RabbitMQ virtual host
AMQP_URL=amqp://plane:<secure-pass>@plane-mq:5672/plane
```

### Storage Configuration

#### Option A: Using MinIO (Self-hosted S3)
```bash
USE_MINIO=1                       # Enable MinIO
AWS_ACCESS_KEY_ID=<access-key>   # MinIO access key (CHANGE THIS!)
AWS_SECRET_ACCESS_KEY=<secret>   # MinIO secret key (CHANGE THIS!)
AWS_S3_ENDPOINT_URL=http://plane-minio:9000
AWS_S3_BUCKET_NAME=uploads        # Bucket name for uploads
AWS_REGION=                       # Leave empty for MinIO
FILE_SIZE_LIMIT=5242880          # Max file size (5MB default)
MINIO_ENDPOINT_SSL=0             # SSL for MinIO endpoint
```

#### Option B: Using AWS S3
```bash
USE_MINIO=0                       # Disable MinIO
AWS_ACCESS_KEY_ID=<your-key>     # AWS access key
AWS_SECRET_ACCESS_KEY=<secret>   # AWS secret key
AWS_REGION=us-east-1             # AWS region
AWS_S3_BUCKET_NAME=plane-uploads # S3 bucket name
AWS_S3_ENDPOINT_URL=             # Leave empty for AWS S3
```

### Application URLs
```bash
# Domain Configuration
WEB_URL=https://plane.yourdomain.com     # Main application URL
APP_DOMAIN=plane.yourdomain.com          # Your domain

# Internal URLs (change if using custom ports)
API_BASE_URL=http://api:8000
APP_BASE_URL=http://localhost:3000
ADMIN_BASE_URL=http://localhost:3001
SPACE_BASE_URL=http://localhost:3002
LIVE_BASE_URL=http://localhost:3100

# Base paths (usually don't need changing)
APP_BASE_PATH=""
ADMIN_BASE_PATH="/god-mode"
SPACE_BASE_PATH="/spaces"
LIVE_BASE_PATH="/live"
```

### Security & Authentication
```bash
# IMPORTANT: Generate a secure secret key!
SECRET_KEY=<generate-secure-key>  # Django secret key (CHANGE THIS!)
# Generate with: openssl rand -hex 32

# API Configuration
DEBUG=0                           # Set to 0 for production
API_KEY_RATE_LIMIT=60/minute     # API rate limiting
CORS_ALLOWED_ORIGINS=            # Leave empty or specify allowed origins
```

### SSL/TLS Configuration
```bash
# SSL Settings
SSL=false                         # Set to true for HTTPS
LISTEN_HTTP_PORT=80              # HTTP port
LISTEN_HTTPS_PORT=443            # HTTPS port

# Let's Encrypt (if SSL=true)
CERT_EMAIL=admin@yourdomain.com  # Email for Let's Encrypt
CERT_ACME_CA=https://acme-v02.api.letsencrypt.org/directory
SITE_ADDRESS=:80

# For DNS Challenge (optional)
CERT_ACME_DNS=                   # DNS provider for certificate
```

### Performance Tuning
```bash
# Gunicorn Workers
GUNICORN_WORKERS=2                # Number of API workers

# Replica Counts (for scaling)
WEB_REPLICAS=1                    # Frontend replicas
API_REPLICAS=1                    # API replicas
WORKER_REPLICAS=1                 # Background worker replicas
SPACE_REPLICAS=1                  # Space app replicas
ADMIN_REPLICAS=1                  # Admin panel replicas
BEAT_WORKER_REPLICAS=1            # Celery beat replicas
```

### Optional Services
```bash
# Analytics (optional)
NEXT_PUBLIC_PLAUSIBLE_DOMAIN=    # Plausible analytics domain
NEXT_PUBLIC_CRISP_ID=             # Crisp chat ID
NEXT_PUBLIC_POSTHOG_KEY=          # PostHog analytics key
NEXT_PUBLIC_POSTHOG_HOST=         # PostHog host

# GPT/AI Features (optional)
OPENAI_API_KEY=sk-...             # OpenAI API key
GPT_ENGINE=gpt-3.5-turbo          # GPT model to use
```

## Deployment Steps

### 1. Using Docker Compose

```bash
# Clone the repository
git clone https://github.com/makeplane/plane.git
cd plane

# Create .env file
cp .env.example .env
# Edit .env with your configuration

# For production deployment with pre-built images
cd deployments/cli/community
docker-compose up -d

# For development/custom builds
docker-compose up -d
```

### 2. Using Coolify

1. **Create a new service** in Coolify
2. **Select Docker Compose** as deployment type
3. **Add the docker-compose.yml content**
4. **Configure environment variables** in Coolify's UI:
   - Add all required environment variables
   - Ensure secure passwords are generated
   - Set your domain in `WEB_URL` and `APP_DOMAIN`
5. **Configure domains**:
   - Main app: `plane.yourdomain.com`
   - Admin: `plane.yourdomain.com/god-mode`
   - Spaces: `plane.yourdomain.com/spaces`
6. **Deploy** the application

### 3. Post-Deployment Setup

1. **Run migrations** (automatically done by migrator service)
2. **Access God Mode** at `https://yourdomain.com/god-mode`
3. **Create admin account** and configure instance settings
4. **Configure SMTP** for email notifications (in God Mode)
5. **Set up OAuth** providers if needed (Google, GitHub, etc.)

## Health Checks

Verify services are running:
```bash
# Check all services
docker-compose ps

# Check logs
docker-compose logs -f api
docker-compose logs -f web
docker-compose logs -f worker
```

## Backup Strategy

### Database Backup
```bash
# Backup PostgreSQL
docker exec plane-db pg_dump -U plane plane > backup.sql

# Restore PostgreSQL
docker exec -i plane-db psql -U plane plane < backup.sql
```

### File Storage Backup
- If using MinIO: Backup the `uploads` volume
- If using S3: Configure S3 versioning and lifecycle policies

## Troubleshooting

### Common Issues

1. **Database connection errors**
   - Verify `DATABASE_URL` is correct
   - Check if `plane-db` service is running
   - Ensure database migrations ran successfully

2. **File upload issues**
   - Check `FILE_SIZE_LIMIT` setting
   - Verify MinIO/S3 credentials
   - Ensure bucket exists and has proper permissions

3. **SSL/Certificate issues**
   - Verify domain DNS points to server
   - Check `CERT_EMAIL` is valid
   - Ensure ports 80/443 are accessible

4. **Performance issues**
   - Increase `GUNICORN_WORKERS`
   - Scale replicas if using orchestration
   - Check database connection pooling

## Security Recommendations

1. **Change all default passwords** in production
2. **Generate secure `SECRET_KEY`**: `openssl rand -hex 32`
3. **Enable SSL/TLS** for production deployments
4. **Restrict `CORS_ALLOWED_ORIGINS`** to your domains
5. **Set `DEBUG=0`** in production
6. **Use strong passwords** for database and RabbitMQ
7. **Configure firewall** to restrict access to internal services
8. **Regular backups** of database and uploads
9. **Keep images updated** with security patches

## Monitoring

Consider adding monitoring for:
- Service health (all containers running)
- Database performance
- Redis memory usage
- RabbitMQ queue sizes
- Application logs
- SSL certificate expiry

## Support

- Documentation: https://docs.plane.so/
- GitHub Issues: https://github.com/makeplane/plane/issues
- Discord: https://discord.com/invite/A92xrEGCge