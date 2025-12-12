# Deployment Guide

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Start with Docker](#quick-start-with-docker)
- [Manual Deployment](#manual-deployment)
- [Production Deployment](#production-deployment)
- [Environment Configuration](#environment-configuration)
- [Troubleshooting](#troubleshooting)
- [Monitoring](#monitoring)
- [Backup and Recovery](#backup-and-recovery)
- [Security Considerations](#security-considerations)

## Prerequisites

### For Docker Deployment
- Docker 20.10+ 
- Docker Compose 2.0+
- At least 4GB RAM
- 10GB free disk space

### For Manual Deployment
- Python 3.10 or higher
- Node.js 16+ and npm
- uv (Python package manager)
- At least 4GB RAM
- 10GB free disk space

### API Keys Required
- Google Gemini API key OR OpenAI API key
- (Optional) MinerU API token for advanced file parsing

## Quick Start with Docker

The fastest way to get Banana Slides running is with Docker Compose.

### 1. Clone the Repository

```bash
git clone https://github.com/Anionex/banana-slides
cd banana-slides
```

### 2. Configure Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` and configure your API keys:

```bash
# Minimum required configuration
AI_PROVIDER_FORMAT=gemini  # or "openai"
GOOGLE_API_KEY=your-gemini-api-key-here
GOOGLE_API_BASE=https://generativelanguage.googleapis.com

# Or for OpenAI format
# AI_PROVIDER_FORMAT=openai
# OPENAI_API_KEY=your-openai-api-key-here
# OPENAI_API_BASE=https://api.openai.com/v1
```

**Recommended API Provider**: [AIHubMix](https://aihubmix.com/?aff=17EC) provides unified access to multiple AI models.

### 3. Start Services

```bash
docker compose up -d
```

This will:
- Build frontend and backend Docker images
- Start both services
- Frontend: http://localhost:3000
- Backend: http://localhost:5000

### 4. Verify Installation

Check service health:

```bash
# Check if services are running
docker compose ps

# Check backend health
curl http://localhost:5000/health

# Check frontend (in browser)
# Open http://localhost:3000
```

### 5. View Logs

```bash
# All services
docker compose logs -f

# Backend only
docker compose logs -f backend

# Frontend only
docker compose logs -f frontend
```

### 6. Stop Services

```bash
docker compose down
```

To remove all data (database, uploads):

```bash
docker compose down -v
```

## Manual Deployment

For development or custom deployments.

### Backend Setup

#### 1. Install Python and uv

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify installation
uv --version
```

#### 2. Install Dependencies

```bash
cd banana-slides
uv sync
```

This will create a virtual environment and install all dependencies.

#### 3. Configure Environment

```bash
cp .env.example .env
# Edit .env with your configuration
```

#### 4. Start Backend

```bash
cd backend
uv run python app.py
```

Backend will start on http://localhost:5000

### Frontend Setup

#### 1. Install Node.js Dependencies

```bash
cd frontend
npm install
```

#### 2. Configure API URL (optional)

If backend is not on localhost:5000, create `.env.local`:

```bash
VITE_API_BASE_URL=http://your-backend-url:5000
```

#### 3. Start Development Server

```bash
npm run dev
```

Frontend will start on http://localhost:3000

#### 4. Build for Production

```bash
npm run build
```

Output will be in `dist/` directory.

## Production Deployment

### Option 1: Docker Compose (Recommended)

Use the provided `docker-compose.yml` with production environment variables.

#### 1. Configure Production Environment

Create `.env.production`:

```bash
# Flask
FLASK_ENV=production
LOG_LEVEL=WARNING
PORT=5000

# AI Configuration
AI_PROVIDER_FORMAT=gemini
GOOGLE_API_KEY=your-production-api-key
TEXT_MODEL=gemini-2.5-flash
IMAGE_MODEL=gemini-3-pro-image-preview

# CORS (restrict to your domain)
CORS_ORIGINS=https://yourdomain.com,https://www.yourdomain.com

# Database (use absolute path)
DATABASE_URL=sqlite:////app/backend/instance/database.db

# Security
SECRET_KEY=your-very-long-random-secret-key-here

# Concurrency
MAX_DESCRIPTION_WORKERS=5
MAX_IMAGE_WORKERS=8
```

#### 2. Deploy

```bash
# Build with no cache
docker compose build --no-cache

# Start in production mode
docker compose up -d

# Scale services if needed
docker compose up -d --scale backend=2
```

#### 3. Setup Reverse Proxy (Nginx)

Create `/etc/nginx/sites-available/banana-slides`:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    # Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Increase timeouts for long-running AI requests
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        send_timeout 300;
    }

    # File serving
    location /files {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
    }

    # Max upload size (200MB)
    client_max_body_size 200M;
}
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/banana-slides /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

#### 4. Setup SSL with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

### Option 2: Manual Production Deployment

#### Backend with Gunicorn

Install Gunicorn:

```bash
uv add gunicorn
```

Create `gunicorn_config.py`:

```python
bind = "0.0.0.0:5000"
workers = 4
worker_class = "sync"
timeout = 300
keepalive = 2
accesslog = "-"
errorlog = "-"
loglevel = "info"
```

Start with Gunicorn:

```bash
cd backend
uv run gunicorn -c gunicorn_config.py app:app
```

#### Frontend with Nginx

Build frontend:

```bash
cd frontend
npm run build
```

Configure Nginx to serve `dist/`:

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /path/to/banana-slides/frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://localhost:5000;
        # ... (same as above)
    }
}
```

### Option 3: Cloud Platforms

#### Deploy to AWS

**Using ECS (Elastic Container Service)**:
1. Push Docker images to ECR
2. Create ECS task definitions
3. Create ECS service
4. Setup Application Load Balancer
5. Configure Route 53 for DNS

**Using Elastic Beanstalk**:
1. Package application
2. Deploy using EB CLI
3. Configure environment variables

#### Deploy to Google Cloud

**Using Cloud Run**:
```bash
# Build and push Docker images
gcloud builds submit --tag gcr.io/PROJECT_ID/banana-slides-backend
gcloud builds submit --tag gcr.io/PROJECT_ID/banana-slides-frontend

# Deploy backend
gcloud run deploy banana-slides-backend \
  --image gcr.io/PROJECT_ID/banana-slides-backend \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated

# Deploy frontend
gcloud run deploy banana-slides-frontend \
  --image gcr.io/PROJECT_ID/banana-slides-frontend \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

#### Deploy to Heroku

```bash
# Login to Heroku
heroku login

# Create apps
heroku create banana-slides-backend
heroku create banana-slides-frontend

# Set environment variables
heroku config:set GOOGLE_API_KEY=xxx -a banana-slides-backend

# Deploy
git push heroku main
```

#### Deploy to DigitalOcean

Use App Platform or Droplets with Docker.

## Environment Configuration

### Backend Environment Variables

#### Required

```bash
# AI Provider
AI_PROVIDER_FORMAT=gemini  # or "openai"

# Gemini Configuration (if using Gemini)
GOOGLE_API_KEY=your-key
GOOGLE_API_BASE=https://generativelanguage.googleapis.com

# OpenAI Configuration (if using OpenAI format)
OPENAI_API_KEY=your-key
OPENAI_API_BASE=https://api.openai.com/v1
```

#### Optional

```bash
# Models
TEXT_MODEL=gemini-2.5-flash
IMAGE_MODEL=gemini-3-pro-image-preview
IMAGE_CAPTION_MODEL=gemini-2.5-flash

# Server
PORT=5000
FLASK_ENV=production
LOG_LEVEL=INFO

# Database
DATABASE_URL=sqlite:///instance/database.db

# CORS
CORS_ORIGINS=http://localhost:3000,https://yourdomain.com

# File Upload
MAX_CONTENT_LENGTH=209715200  # 200MB in bytes

# Concurrency
MAX_DESCRIPTION_WORKERS=5
MAX_IMAGE_WORKERS=8

# MinerU (optional)
MINERU_TOKEN=your-token
MINERU_API_BASE=https://mineru.net

# Security
SECRET_KEY=your-secret-key
```

### Frontend Environment Variables

```bash
# API Base URL
VITE_API_BASE_URL=http://localhost:5000
```

## Troubleshooting

### Common Issues

#### 1. Docker Build Fails

**Problem**: Docker build fails with dependency errors

**Solution**:
```bash
# Clear Docker cache
docker system prune -a

# Rebuild with no cache
docker compose build --no-cache
```

#### 2. Port Already in Use

**Problem**: Port 3000 or 5000 already in use

**Solution**:
```bash
# Find process using port
sudo lsof -i :5000
sudo lsof -i :3000

# Kill process
kill -9 <PID>

# Or change port in docker-compose.yml
```

#### 3. Database Locked

**Problem**: SQLite database is locked

**Solution**:
- Ensure WAL mode is enabled (should be automatic)
- Check for long-running transactions
- Restart backend service

#### 4. AI API Errors

**Problem**: "Invalid API key" or rate limit errors

**Solution**:
- Verify API key in `.env`
- Check API quotas and limits
- Verify API base URL is correct
- Test API key with curl:
  ```bash
  curl -H "Authorization: Bearer YOUR_KEY" \
    https://generativelanguage.googleapis.com/v1beta/models
  ```

#### 5. File Upload Fails

**Problem**: Large file upload fails

**Solution**:
- Check `MAX_CONTENT_LENGTH` in config
- Verify disk space
- Check upload folder permissions:
  ```bash
  chmod 777 uploads/
  ```

#### 6. Frontend Cannot Connect to Backend

**Problem**: CORS errors or connection refused

**Solution**:
- Verify backend is running: `curl http://localhost:5000/health`
- Check CORS_ORIGINS in backend `.env`
- Verify VITE_API_BASE_URL in frontend

#### 7. Memory Issues

**Problem**: Container or process runs out of memory

**Solution**:
- Reduce MAX_IMAGE_WORKERS and MAX_DESCRIPTION_WORKERS
- Increase Docker memory limit
- Monitor with `docker stats`

### Log Analysis

```bash
# Backend logs
docker compose logs backend | grep ERROR

# Frontend logs
docker compose logs frontend | grep ERROR

# Database logs
docker compose logs backend | grep "database"

# AI service logs
docker compose logs backend | grep "AIService"
```

## Monitoring

### Health Checks

```bash
# Backend health
curl http://localhost:5000/health

# Check database
curl http://localhost:5000/api/projects?limit=1

# Check AI service
curl -X POST http://localhost:5000/api/projects \
  -H "Content-Type: application/json" \
  -d '{"creation_type":"idea","idea_prompt":"test"}'
```

### Metrics to Monitor

1. **Response Times**
   - API endpoint response times
   - AI generation times
   - Export generation times

2. **Resource Usage**
   - CPU usage
   - Memory usage
   - Disk usage (especially uploads/ and instance/)
   - Network bandwidth

3. **Error Rates**
   - 4xx errors (client errors)
   - 5xx errors (server errors)
   - AI API errors

4. **Queue Metrics**
   - Pending tasks
   - Failed tasks
   - Average task completion time

### Monitoring Tools

**Recommended tools**:
- **Prometheus + Grafana**: Metrics and dashboards
- **Sentry**: Error tracking
- **New Relic / Datadog**: Application monitoring
- **Uptime Robot**: Uptime monitoring

## Backup and Recovery

### Database Backup

```bash
# Backup SQLite database
cp backend/instance/database.db backend/instance/database.db.backup

# Automated backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
cp backend/instance/database.db backups/database_$DATE.db
# Keep only last 7 days
find backups/ -name "database_*.db" -mtime +7 -delete
```

### Uploads Backup

```bash
# Backup uploads folder
tar -czf uploads_backup.tar.gz uploads/

# Restore
tar -xzf uploads_backup.tar.gz
```

### Full Backup

```bash
# Backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="backups/$DATE"
mkdir -p $BACKUP_DIR

# Backup database
cp backend/instance/database.db $BACKUP_DIR/

# Backup uploads
cp -r uploads/ $BACKUP_DIR/

# Backup configuration
cp .env $BACKUP_DIR/

# Create archive
tar -czf banana-slides-backup-$DATE.tar.gz $BACKUP_DIR/
rm -rf $BACKUP_DIR/
```

### Recovery

```bash
# Extract backup
tar -xzf banana-slides-backup-YYYYMMDD.tar.gz

# Restore database
cp backup_dir/database.db backend/instance/

# Restore uploads
cp -r backup_dir/uploads/ ./

# Restart services
docker compose restart
```

## Security Considerations

### 1. API Keys

- **Never commit** API keys to version control
- Store in environment variables
- Use different keys for development and production
- Rotate keys regularly

### 2. CORS

- Restrict CORS_ORIGINS to your domain in production
- Never use `*` in production

### 3. File Uploads

- Validate file types and sizes
- Scan uploaded files for malware
- Store uploads outside web root
- Use unique filenames (UUID)

### 4. Database

- Enable WAL mode (already configured)
- Regular backups
- Limit access to database file
- Consider PostgreSQL for large deployments

### 5. HTTPS

- Always use HTTPS in production
- Use Let's Encrypt for free SSL certificates
- Force HTTPS redirect

### 6. Rate Limiting

Implement rate limiting for:
- API endpoints
- File uploads
- AI generation requests

Example with Flask-Limiter:

```python
from flask_limiter import Limiter

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["100 per hour"]
)

@app.route("/api/projects", methods=["POST"])
@limiter.limit("10 per minute")
def create_project():
    # ...
```

### 7. Authentication

Consider adding authentication for production:
- JWT tokens
- OAuth 2.0
- API keys per user

### 8. Input Validation

- Validate all user inputs
- Sanitize file uploads
- Escape HTML in user content
- Use parameterized database queries

### 9. Error Messages

- Don't expose sensitive information in errors
- Log detailed errors server-side
- Show generic errors to users

### 10. Updates

- Keep dependencies updated
- Subscribe to security advisories
- Test updates in staging environment

## Performance Optimization

### 1. Caching

Implement caching for:
- AI responses (cache identical prompts)
- Frequently accessed files
- API responses

### 2. CDN

Use CDN for:
- Frontend static assets
- Generated slide images
- Material files

### 3. Database Optimization

- Add indexes on frequently queried fields
- Consider PostgreSQL for better performance
- Implement connection pooling

### 4. Image Optimization

- Compress generated images
- Use WebP format when supported
- Implement lazy loading

### 5. Async Processing

- Use Redis + Celery for better task queue (currently using ThreadPoolExecutor)
- Implement WebSocket for real-time updates

## Scaling

### Horizontal Scaling

For high traffic:

1. **Load Balancer**: Distribute traffic across multiple backend instances
2. **Shared Storage**: Use S3 or NFS for uploads
3. **Redis**: For session storage and caching
4. **PostgreSQL**: Replace SQLite for concurrent access

### Example Architecture

```
          ┌─────────────┐
          │ Load Balancer│
          └──────┬───────┘
                 │
      ┌──────────┼──────────┐
      │          │          │
  ┌───▼──┐   ┌──▼───┐  ┌───▼──┐
  │Backend│   │Backend│  │Backend│
  └───┬──┘   └──┬───┘  └───┬──┘
      └──────────┼──────────┘
                 │
      ┌──────────▼──────────┐
      │                     │
  ┌───▼────┐          ┌─────▼───┐
  │PostgreSQL│        │  Redis  │
  └────────┘          └─────────┘
                           │
                      ┌────▼────┐
                      │   S3    │
                      └─────────┘
```

## Maintenance

### Regular Tasks

**Daily**:
- Check error logs
- Monitor disk space
- Verify backups

**Weekly**:
- Review performance metrics
- Check for updates
- Test backup restoration

**Monthly**:
- Security audit
- Dependency updates
- Performance optimization review

### Updating the Application

```bash
# Pull latest code
git pull

# Rebuild and restart
docker compose down
docker compose build --no-cache
docker compose up -d

# Check logs
docker compose logs -f
```

## Support and Resources

- **GitHub Repository**: https://github.com/Anionex/banana-slides
- **Issues**: https://github.com/Anionex/banana-slides/issues
- **Documentation**: https://github.com/Anionex/banana-slides/tree/main/docs
- **AI Provider**: [AIHubMix](https://aihubmix.com/?aff=17EC)
