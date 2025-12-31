# Example Workflow Files

These are example workflow files that you can copy to your Spring Boot projects.

## File Structure in Your Project

```
your-spring-boot-project/
├── .github/
│   └── workflows/
│       ├── deploy-dev.yml
│       └── deploy-pro.yml
├── src/
├── build.gradle
└── ...
```

## For Development Environment

**File: `.github/workflows/deploy-dev.yml`**

```yaml
name: deploy-dev

on:
  push:
    branches:
      - dev
  workflow_dispatch:

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      java-version: '17'
      environment: dev
      project-name: my-project        # 👈 CHANGE THIS: Your project name
      application-name: api
      docker-hub-username: kayplebdev # 👈 CHANGE THIS: Your Docker Hub username
      keep-images: 4
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: dev
      project-name: my-project        # 👈 CHANGE THIS: Your project name
      application-name: api
      image-tag: ${{ needs.build.outputs.image-tag }}
      container-port: '8080'
      blue-port: '8080'
      green-port: '8081'
      domain: dev-api.my-project.com  # 👈 CHANGE THIS: Your dev domain
      health-check-path: '/actuator/health/liveness'
      health-check-attempts: 30
      java-opts: '-Xms512m -Xmx1024m'
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
      SSH_PORT: ${{ secrets.DEV_SSH_PORT }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

## For Production Environment

**File: `.github/workflows/deploy-pro.yml`**

```yaml
name: deploy-pro

on:
  push:
    branches:
      - pro
  workflow_dispatch:

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      java-version: '17'
      environment: pro
      project-name: my-project        # 👈 CHANGE THIS: Your project name
      application-name: api
      docker-hub-username: kayplebdev # 👈 CHANGE THIS: Your Docker Hub username
      keep-images: 4
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: my-project        # 👈 CHANGE THIS: Your project name
      application-name: api
      image-tag: ${{ needs.build.outputs.image-tag }}
      container-port: '8080'
      blue-port: '8080'
      green-port: '8081'
      domain: api.my-project.com      # 👈 CHANGE THIS: Your prod domain
      health-check-path: '/actuator/health/liveness'
      health-check-attempts: 30
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
      SSH_PORT: ${{ secrets.PRO_SSH_PORT }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  # Optional: Register with Prometheus for monitoring
  # Uncomment this section if you have Prometheus set up
  # prometheus:
  #   needs: deploy
  #   uses: kayple/kayple-ci-templates/.github/workflows/prometheus-register.yml@main
  #   with:
  #     environment: pro
  #     project-name: my-project
  #     domain: api.my-project.com
  #   secrets:
  #     SSH_HOST: ${{ secrets.PROMETHEUS_HOST }}
  #     SSH_USER: ${{ secrets.PROMETHEUS_USER }}
  #     SSH_PASSWORD: ${{ secrets.PROMETHEUS_PASSWORD }}
  #     SSH_PORT: ${{ secrets.PROMETHEUS_SSH_PORT }}
```

## Customization Guide

### What to Change

1. **project-name**: Your project identifier (e.g., "noriyagi", "keymedia", "dangdaepyo")
2. **docker-hub-username**: Your Docker Hub username or organization
3. **domain**: Your application domain (dev-api.example.com, api.example.com)
4. **docker-env-vars**: Add any additional environment variables your app needs

### What to Keep

- All the `uses: kayple/kayple-ci-templates/...` lines (this is the dependency!)
- Secret variable names (these match what you set in GitHub Secrets)
- Port numbers (unless you have specific requirements)

### Optional Customizations

#### Different Ports

```yaml
with:
  blue-port: '9090'
  green-port: '9091'
```

#### More Memory

```yaml
with:
  java-opts: '-Xms2g -Xmx4g'  # 4GB max heap
```

#### Keep More Images

```yaml
with:
  keep-images: 10  # Keep last 10 images instead of 4
```

#### Different Health Check Path

```yaml
with:
  health-check-path: '/health'  # If not using Spring Actuator
```

#### Longer Health Check Timeout

```yaml
with:
  health-check-attempts: 60  # Try for 10 minutes instead of 5
```

#### Additional Environment Variables

```yaml
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
  MAIL_HOST=${{ secrets.PRO_MAIL_HOST }}
  MAIL_PORT=587
  SLACK_WEBHOOK=${{ secrets.PRO_SLACK_WEBHOOK }}
  LOG_LEVEL=INFO
```

## Real World Examples

### Example 1: noriyagi_back

```yaml
# .github/workflows/deploy-pro.yml
name: deploy-pro

on:
  push:
    branches: [pro]

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      environment: pro
      project-name: noriyagi
      docker-hub-username: kayplebdev
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: noriyagi
      image-tag: ${{ needs.build.outputs.image-tag }}
      blue-port: '8080'
      green-port: '8081'
      domain: api.noriyagi.com
      docker-env-vars: |
        DB_URL=${{ secrets.PRO_DB_URL }}
        DB_USERNAME=${{ secrets.PRO_DB_USERNAME }}
        DB_PASSWORD=${{ secrets.PRO_DB_PASSWORD }}
        JWT_SECRET=${{ secrets.PRO_JWT_SECRET }}
    secrets:
      SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
      SSH_USER: ${{ secrets.PRO_SSH_USER }}
      SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

### Example 2: keymedia-api (with more resources)

```yaml
# .github/workflows/deploy-pro.yml
name: deploy-pro

on:
  push:
    branches: [pro]

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      environment: pro
      project-name: keymedia
      docker-hub-username: kayplebdev
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: keymedia
      image-tag: ${{ needs.build.outputs.image-tag }}
      blue-port: '8080'
      green-port: '8081'
      domain: api.keymedia.com
      java-opts: '-Xms2g -Xmx4g'  # Large app, needs more memory
      docker-env-vars: |
        DB_URL=${{ secrets.PRO_DB_URL }}
        DB_USERNAME=${{ secrets.PRO_DB_USERNAME }}
        DB_PASSWORD=${{ secrets.PRO_DB_PASSWORD }}
        REDIS_HOST=${{ secrets.PRO_REDIS_HOST }}
        S3_BUCKET=${{ secrets.PRO_S3_BUCKET }}
        AWS_REGION=ap-northeast-2
    secrets:
      SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
      SSH_USER: ${{ secrets.PRO_SSH_USER }}
      SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

### Example 3: dang-daepyo-backend (with Prometheus)

```yaml
# .github/workflows/deploy-pro.yml
name: deploy-pro

on:
  push:
    branches: [pro]

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      environment: pro
      project-name: dangdaepyo
      docker-hub-username: kayplebdev
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: dangdaepyo
      image-tag: ${{ needs.build.outputs.image-tag }}
      blue-port: '8080'
      green-port: '8081'
      domain: api.dangdaepyo.com
      docker-env-vars: |
        DB_URL=${{ secrets.PRO_DB_URL }}
        DB_USERNAME=${{ secrets.PRO_DB_USERNAME }}
        DB_PASSWORD=${{ secrets.PRO_DB_PASSWORD }}
    secrets:
      SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
      SSH_USER: ${{ secrets.PRO_SSH_USER }}
      SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
      DOCKER_HUB_USERNAME: kayplebdev
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  prometheus:
    needs: deploy
    uses: kayple/kayple-ci-templates/.github/workflows/prometheus-register.yml@main
    with:
      environment: pro
      project-name: dangdaepyo
      domain: api.dangdaepyo.com
    secrets:
      SSH_HOST: ${{ secrets.PROMETHEUS_HOST }}
      SSH_USER: ${{ secrets.PROMETHEUS_USER }}
      SSH_PASSWORD: ${{ secrets.PROMETHEUS_PASSWORD }}
```

## Quick Copy-Paste Template

Here's a minimal template you can quickly customize:

```yaml
name: deploy-pro

on:
  push:
    branches: [pro]

jobs:
  build:
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-build.yml@main
    with:
      environment: pro
      project-name: PROJECT_NAME_HERE
      docker-hub-username: YOUR_DOCKERHUB_HERE
    secrets:
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

  deploy:
    needs: build
    uses: kayple/kayple-ci-templates/.github/workflows/spring-boot-deploy.yml@main
    with:
      environment: pro
      project-name: PROJECT_NAME_HERE
      image-tag: ${{ needs.build.outputs.image-tag }}
      blue-port: '8080'
      green-port: '8081'
      domain: YOUR_DOMAIN_HERE
      docker-env-vars: |
        DB_URL=${{ secrets.PRO_DB_URL }}
        DB_USERNAME=${{ secrets.PRO_DB_USERNAME }}
        DB_PASSWORD=${{ secrets.PRO_DB_PASSWORD }}
    secrets:
      SSH_HOST: ${{ secrets.PRO_SSH_HOST }}
      SSH_USER: ${{ secrets.PRO_SSH_USER }}
      SSH_KEY: ${{ secrets.PRO_SSH_KEY }}
      DOCKER_HUB_USERNAME: YOUR_DOCKERHUB_HERE
      DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

Replace:
- `PROJECT_NAME_HERE` → Your project name (e.g., noriyagi, keymedia)
- `YOUR_DOCKERHUB_HERE` → Your Docker Hub username
- `YOUR_DOMAIN_HERE` → Your domain (e.g., api.noriyagi.com)
