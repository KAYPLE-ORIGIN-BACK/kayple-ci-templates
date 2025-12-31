# Kayple CI/CD Templates

**Reusable GitHub Actions workflows for Spring Boot microservices at Kayple**

This repository provides centralized CI/CD templates that all Spring Boot projects can use as a dependency. Instead of copying deployment code to every project, simply reference these workflows and provide your project-specific configuration.

## 🎯 Features

- ✅ **Blue/Green Zero-Downtime Deployment** - No service interruption during updates
- ✅ **Automated Docker Image Management** - Build, tag, push, and cleanup old images
- ✅ **Health Check with Auto-Rollback** - Ensures new deployment is healthy before switching traffic
- ✅ **Nginx Configuration Updates** - Automatically switches proxy to new container
- ✅ **Multi-Environment Support** - Separate DEV and PRO configurations
- ✅ **Project-Specific Servers** - Each project can deploy to different servers
- ✅ **Flexible Environment Variables** - Pass any configuration as Docker env vars
- ✅ **Optional Prometheus Integration** - Automatic monitoring target registration

## 📦 Available Workflows

### 1. `spring-boot-build.yml`
Builds Spring Boot application, creates Docker image, and pushes to Docker Hub.

**Inputs:**
- `java-version` - Java version (default: '17')
- `environment` - Environment name: dev, pro, etc. (required)
- `project-name` - Project name for Docker image (required)
- `application-name` - Application name (default: 'api')
- `docker-hub-username` - Docker Hub username (required)
- `keep-images` - Number of recent images to keep (default: 4)

**Outputs:**
- `image-tag` - Built Docker image tag (e.g., kayplebdev/noriyagi-api:pro.2024.12.31.42)

**Secrets:**
- `DOCKER_HUB_ACCESS_TOKEN` - Docker Hub access token (required)
- `FIREBASE_KEY_BASE64` - Base64 encoded Firebase key file (optional)
- `APP_STORE_KEY_BASE64` - Base64 encoded App Store key file (optional)
- `IAP_PAYMENT_KEY_BASE64` - Base64 encoded IAP payment key file (optional)

### 2. `spring-boot-deploy.yml`
Deploys application to server using Blue/Green strategy with zero downtime.

**Inputs:**
- `environment` - Environment name (required)
- `project-name` - Project name (required)
- `application-name` - Application name (default: 'api')
- `image-tag` - Docker image tag to deploy (required)
- `container-port` - Container internal port (default: '8080')
- `blue-port` - Blue deployment port (required)
- `green-port` - Green deployment port (required)
- `domain` - Domain for Nginx config (required)
- `health-check-path` - Health check endpoint (default: '/actuator/health/liveness')
- `health-check-attempts` - Max health check attempts (default: 30)
- `java-opts` - JVM options (default: '-Xms512m -Xmx1024m')
- `docker-env-vars` - Environment variables as multiline string (optional)

**Secrets:**
- `SSH_HOST` - Target server IP address (required)
- `SSH_USER` - SSH username (required)
- `SSH_KEY` - SSH private key (required)
- `SSH_PORT` - SSH port (optional, default: 22)
- `DOCKER_HUB_USERNAME` - Docker Hub username (required)
- `DOCKER_HUB_ACCESS_TOKEN` - Docker Hub access token (required)

### 3. `prometheus-register.yml`
Registers deployment with Prometheus for monitoring (optional).

**Inputs:**
- `environment` - Environment name (required)
- `project-name` - Project name (required)
- `domain` - Domain to monitor (required)
- `prometheus-targets-dir` - Prometheus targets directory (default: '~/prometheus/targets')

**Secrets:**
- `SSH_HOST` - Prometheus server IP (required)
- `SSH_USER` - SSH username (required)
- `SSH_KEY` - SSH private key (optional)
- `SSH_PASSWORD` - SSH password (optional)
- `SSH_PORT` - SSH port (optional, default: 22)

## 🚀 Quick Start

### Step 1: Set Up Organization Secret

Go to your GitHub organization settings and add:
- **Settings → Secrets and variables → Actions → New organization secret**
- Name: `DOCKER_HUB_ACCESS_TOKEN`
- Value: Your Docker Hub access token
- Access: All repositories (or specific ones)

### Step 2: Configure Project Secrets

For each project repository, add these secrets:

**Development Environment:**
```
DEV_SSH_HOST          - Development server IP
DEV_SSH_USER          - SSH username (e.g., ubuntu)
DEV_SSH_KEY           - SSH private key
DEV_DB_URL            - Database connection URL
DEV_DB_USERNAME       - Database username
DEV_DB_PASSWORD       - Database password
DEV_JWT_SECRET        - JWT secret key (optional)
DEV_SLACK_WEBHOOK_URL - Slack webhook URL (optional)
```

**Production Environment:**
```
PRO_SSH_HOST          - Production server IP
PRO_SSH_USER          - SSH username
PRO_SSH_KEY           - SSH private key
PRO_DB_URL            - Database connection URL
PRO_DB_USERNAME       - Database username
PRO_DB_PASSWORD       - Database password
PRO_JWT_SECRET        - JWT secret key (optional)
PRO_SLACK_WEBHOOK_URL - Slack webhook URL (optional)
```

### Step 3: Create Workflow Files in Your Project

In your Spring Boot project, create two workflow files:

#### `.github/workflows/deploy-dev.yml`

```yaml
name: deploy-dev

on:
  push:
    branches: [dev]
  workflow_dispatch:

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      java-version: '17'
      environment: dev
      project-name: my-project        # 👈 Change to your project name
      application-name: api
      docker-hub-username: kayplebdev # 👈 Change to your Docker Hub username
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: dev
      project-name: my-project        # 👈 Change to your project name
      application-name: api
      image-tag: ${{ needs.build.outputs.image-tag }}
      container-port: '8080'
      blue-port: '8080'
      green-port: '8081'
      domain: dev-api.my-project.com  # 👈 Change to your domain
      health-check-path: '/actuator/health/liveness'
      docker-env-vars: |
        DB_URL=${{ secrets.DEV_DB_URL }}
        DB_USERNAME=${{ secrets.DEV_DB_USERNAME }}
        DB_PASSWORD=${{ secrets.DEV_DB_PASSWORD }}
        JWT_SECRET=${{ secrets.DEV_JWT_SECRET }}
        SLACK_WEBHOOK_URL=${{ secrets.DEV_SLACK_WEBHOOK_URL }}
    secrets:
      SSH_HOST: ${{ secrets.DEV_SSH_HOST }}
      SSH_USER: ${{ secrets.DEV_SSH_USER }}
      SSH_KEY: ${{ secrets.DEV_SSH_KEY }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

#### `.github/workflows/deploy-pro.yml`

```yaml
name: deploy-pro

on:
  push:
    branches: [pro]
  workflow_dispatch:

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      java-version: '17'
      environment: pro
      project-name: my-project        # 👈 Change to your project name
      application-name: api
      docker-hub-username: kayplebdev # 👈 Change to your Docker Hub username
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: my-project        # 👈 Change to your project name
      application-name: api
      image-tag: ${{ needs.build.outputs.image-tag }}
      container-port: '8080'
      blue-port: '8080'
      green-port: '8081'
      domain: api.my-project.com      # 👈 Change to your domain
      health-check-path: '/actuator/health/liveness'
      java-opts: '-Xms1g -Xmx2g'      # More memory for production
      docker-env-vars: |
        DB_URL=${{ secrets.PRO_DB_URL }}
        DB_USERNAME=${{ secrets.PRO_DB_USERNAME }}
        DB_PASSWORD=${{ secrets.PRO_DB_PASSWORD }}
        JWT_SECRET=${{ secrets.PRO_JWT_SECRET }}
        SLACK_WEBHOOK_URL=${{ secrets.PRO_SLACK_WEBHOOK_URL }}
    secrets:
      SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
      SSH_USER: ${{ secrets.PRO_SSH_USER }}
      SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

### Step 4: Deploy

Push to your `dev` or `pro` branch:

```bash
git add .github/workflows/
git commit -m "Add CI/CD workflows"
git push origin dev   # Triggers dev deployment
git push origin pro   # Triggers pro deployment
```

## 📖 Usage Examples

### Basic Deployment

Minimal configuration for a simple project:

```yaml
jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      environment: pro
      project-name: simple-api
      docker-hub-username: kayplebdev
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: simple-api
      image-tag: ${{ needs.build.outputs.image-tag }}
      blue-port: '8080'
      green-port: '8081'
      domain: api.simple.com
      docker-env-vars: |
        DB_URL=${{ secrets.PRO_DB_URL }}
    secrets:
      SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
      SSH_USER: ${{ secrets.PRO_SSH_USER }}
      SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

### Advanced Configuration

With custom ports, memory settings, and additional environment variables:

```yaml
deploy:
  uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
  with:
    environment: pro
    project-name: advanced-api
    image-tag: ${{ needs.build.outputs.image-tag }}
    blue-port: '9090'
    green-port: '9091'
    domain: api.advanced.com
    java-opts: '-Xms2g -Xmx4g -XX:+UseG1GC'
    health-check-attempts: 60  # Wait longer for startup
    docker-env-vars: |
      DB_URL=${{ secrets.PRO_DB_URL }}
      DB_USERNAME=${{ secrets.PRO_DB_USERNAME }}
      DB_PASSWORD=${{ secrets.PRO_DB_PASSWORD }}
      REDIS_HOST=${{ secrets.PRO_REDIS_HOST }}
      REDIS_PORT=6379
      S3_BUCKET=${{ secrets.PRO_S3_BUCKET }}
      AWS_REGION=ap-northeast-2
      SPRING_PROFILES_ACTIVE=pro
      JWT_SECRET=${{ secrets.PRO_JWT_SECRET }}
      JWT_EXPIRATION=86400
      SLACK_WEBHOOK=${{ secrets.PRO_SLACK_WEBHOOK }}
      LOG_LEVEL=INFO
  secrets:
    SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
    SSH_USER: ${{ secrets.PRO_SSH_USER }}
    SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
    DOCKER_HUB_USERNAME: kayplebdev
    DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

### With Prometheus Monitoring

Add optional Prometheus registration after deployment:

```yaml
jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    # ... build configuration ...

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    # ... deploy configuration ...

  prometheus:
    needs: deploy
    uses: kayple/kayple-ci-templates/.github/workflows/prometheus-register.yml@main
    with:
      environment: pro
      project-name: my-project
      domain: api.my-project.com
    secrets:
      SSH_HOST: ${{ secrets.PROMETHEUS_HOST }}
      SSH_USER: ${{ secrets.PROMETHEUS_USER }}
      SSH_PASSWORD: ${{ secrets.PROMETHEUS_PASSWORD }}
```

### With Secret Key Files

If your project needs Firebase, App Store, or IAP payment keys:

```yaml
build:
  uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
  with:
    environment: pro
    project-name: my-project
    docker-hub-username: kayplebdev
  secrets:
    DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
    FIREBASE_KEY_BASE64: ${{ secrets.FIREBASE_KEY_BASE64 }}
    APP_STORE_KEY_BASE64: ${{ secrets.APP_STORE_KEY_BASE64 }}
    IAP_PAYMENT_KEY_BASE64: ${{ secrets.IAP_PAYMENT_KEY_BASE64 }}
```

The build workflow will automatically decode and place these files in `src/main/resources/key-store/`.

## 🏗️ How It Works

### Deployment Flow

```
1. Code Push (dev/pro branch)
        ↓
2. BUILD JOB
   ├─ Checkout code
   ├─ Setup Java 17
   ├─ Restore secret key files (if provided)
   ├─ Build with Gradle
   ├─ Create Docker image with timestamped tag
   ├─ Push to Docker Hub
   └─ Cleanup old images (keep 4 recent)
        ↓
3. DEPLOY JOB
   ├─ SSH to target server
   ├─ Login to Docker Hub
   ├─ Pull new Docker image
   ├─ Determine active container (Blue/Green)
   ├─ Deploy to inactive container
   ├─ Health check (up to 5 minutes)
   ├─ Update Nginx configuration
   ├─ Reload Nginx
   ├─ Verify traffic switch
   └─ Remove old container
        ↓
4. PROMETHEUS JOB (Optional)
   └─ Register with monitoring
```

### Blue/Green Deployment

The system runs two containers on different ports:
- **Blue container**: Port 8080
- **Green container**: Port 8081

**Deployment process:**
1. If Blue is active → Deploy to Green
2. Health check Green container
3. Update Nginx to point to Green
4. Remove Blue container
5. Next deployment will use Blue again

This ensures **zero downtime** during deployments.

### Docker Image Tagging

Images are tagged with environment, date, and build number:

```
kayplebdev/noriyagi-api:pro.2024.12.31.42
                         │    │         └─ GitHub run number
                         │    └─────────── Date (YYYY.MM.DD)
                         └──────────────── Environment (dev/pro)
```

This makes it easy to:
- Identify when an image was built
- Track deployments
- Rollback to previous versions

## 🔧 Server Requirements

### Docker

```bash
# Install Docker
sudo apt-get update
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

### Nginx

Install and configure Nginx:

```bash
# Install
sudo apt-get install nginx -y

# Create site configuration
sudo nano /etc/nginx/sites-available/api.my-project.com
```

Nginx configuration example:

```nginx
server {
    listen 80;
    server_name api.my-project.com;

    location / {
        proxy_pass http://127.0.0.1:8080/;  # Will be updated by CI/CD
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/api.my-project.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Spring Boot Configuration

Ensure your `application.yml` has health check endpoints:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health
  endpoint:
    health:
      probes:
        enabled: true
```

### SSH Access

Ensure GitHub Actions can SSH to your server:

```bash
# Generate SSH key pair (if needed)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/deploy_key

# Add public key to server
cat ~/.ssh/deploy_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Copy private key to GitHub Secrets (as SSH_KEY)
cat ~/.ssh/deploy_key
```

## 🔐 Security Best Practices

1. **Use separate SSH keys per environment**
   - Different keys for DEV and PRO
   - Rotate keys regularly

2. **Never commit secrets**
   - All sensitive data in GitHub Secrets
   - Never hardcode passwords in workflow files

3. **Use private Docker repositories** (if possible)
   - Protects your application images

4. **Enable HTTPS in production**
   - Use Let's Encrypt for free SSL certificates
   - Update Nginx config to redirect HTTP to HTTPS

5. **Rotate credentials regularly**
   - Database passwords
   - JWT secrets
   - API keys

6. **Use least privilege access**
   - SSH users should have minimal permissions
   - Use sudo only when necessary

## 🐛 Troubleshooting

### Check Deployment Status

```bash
# SSH to server
ssh ubuntu@your-server-ip

# List containers
docker ps -a

# Check logs
docker logs my-project-api-pro-BLUE --tail 100
docker logs my-project-api-pro-GREEN --tail 100

# Test health endpoint
curl http://localhost:8080/actuator/health/liveness
curl http://localhost:8081/actuator/health/liveness
```

### Check Nginx

```bash
# Test configuration
sudo nginx -t

# View configuration
cat /etc/nginx/sites-available/api.my-project.com

# Reload Nginx
sudo systemctl reload nginx

# Check logs
tail -f /var/log/nginx/error.log
tail -f /var/log/nginx/access.log
```

### Manual Rollback

If deployment fails and auto-rollback doesn't work:

```bash
# Find previous image
docker images | grep my-project

# Run previous image manually
docker run -d --name my-project-api-pro-BLUE \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=pro \
  -e DB_URL=jdbc:mysql://... \
  --restart always \
  kayplebdev/my-project-api:pro.2024.12.30.41

# Update Nginx
sudo sed -i 's/proxy_pass http:\/\/127\.0\.0\.1:[0-9]*\//proxy_pass http:\/\/127.0.0.1:8080\//g' \
  /etc/nginx/sites-available/api.my-project.com

sudo nginx -s reload
```

### Common Issues

**Problem: "Repository not found"**
- Solution: Ensure template repository is accessible (check org permissions)

**Problem: "Secrets not found"**
- Solution: Add required secrets to repository settings

**Problem: "Health check timeout"**
- Solution: Increase `health-check-attempts` or fix application startup issues

**Problem: "Nginx reload failed"**
- Solution: Check Nginx configuration syntax with `sudo nginx -t`

**Problem: "Container keeps restarting"**
- Solution: Check logs with `docker logs <container-name>`, verify environment variables

## 📊 Monitoring

### View Deployment History

GitHub Actions page shows all deployments:
- `https://github.com/kayple/{project-name}/actions`

Each workflow run shows:
- Build logs
- Deployment logs
- Success/failure status
- Deployment duration

### Docker Images

Check your Docker Hub repository:
- `https://hub.docker.com/r/{username}/{project-name}-api/tags`

You'll see all tagged images with timestamps.

### Server Monitoring

Check running containers:

```bash
# List all containers
docker ps -a

# Container resource usage
docker stats

# Container logs
docker logs -f <container-name>
```

## 🔄 Updating the Template

To update deployment logic for all projects:

1. Update workflow files in this repository
2. Commit and push to main branch
3. All projects automatically use new version on next deployment

For versioned releases:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

Then projects can pin to specific versions:

```yaml
uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@v1.0.0
```

## 📚 Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Nginx Documentation](https://nginx.org/en/docs/)

## 🤝 Contributing

When updating this template:

1. Test changes in a separate branch
2. Verify with at least one project
3. Document changes in commit messages
4. Consider backward compatibility

## 📄 License

Internal use only - Kayple organization.

## 💬 Support

For issues or questions:
- Open an issue in this repository
- Contact DevOps team
- Check GitHub Actions logs for deployment details
