# Infrastructure CI/CD Components

Shared GitHub Actions and reusable workflows for deployment and infrastructure management.

## 📦 Composite Actions

### Deployment & Rollback
- **deploy-service** - Universal deployment action for Docker-based services
- **rollback-service** - Universal rollback action with automatic config extraction
- **backup-version** - Save current deployment version for rollback

### Monitoring & Cleanup
- **cleanup-images** - Cleanup old Docker images after successful deployment
- **full-cleanup** - Weekly Docker system cleanup (images, volumes, networks)
- **grafana-annotation** - Send deployment/rollback annotations to Grafana

## 🔄 Reusable Workflows

### deploy-reusable.yml
Universal deployment workflow with the following steps:
1. Backup current version
2. Deploy service (pull image, extract configs, docker compose up)
3. Run smoke tests
4. Send Grafana annotation
5. Cleanup old images

**Inputs:**
- `deploy_path` - Deployment path on VPS (e.g., `/app/currency-bot`)
- `docker_image` - Docker image to deploy
- `cleanup_image_pattern` - Pattern for image cleanup
- `commit_sha` - Commit SHA
- `commit_message` - Commit message
- `wait_time` - Time to wait after starting services (default: 20s)

**Secrets:**
- `ssh_host`, `ssh_user`, `ssh_key`, `ssh_port` - SSH credentials
- `env_content` - Environment variables content
- `grafana_api_key` - Grafana API key (optional)

### rollback-reusable.yml
Universal rollback workflow with verification:
1. Rollback to previous version
2. Run smoke tests to verify rollback
3. Send Grafana annotation

**Inputs:**
- `deploy_path` - Deployment path on VPS
- `commit_sha` - Failed deployment commit SHA
- `wait_time` - Time to wait after starting services (default: 20s)

**Secrets:** Same as deploy-reusable.yml

## 📋 Requirements

Each repository using these workflows must have:
- `.github/actions/smoke-tests` - Repository-specific smoke tests action
- `docker-compose.yml` and configs embedded in Docker image
- Docker entrypoint that extracts configs when `/output` volume is mounted

## 🚀 Usage Examples

### Deploy workflow
```yaml
deploy:
  uses: StanislavPlotnikov/infra-ci/.github/workflows/deploy-reusable.yml@master
  with:
    deploy_path: /app/my-service
    docker_image: ghcr.io/${{ github.repository }}:latest
    cleanup_image_pattern: my-service
    commit_sha: ${{ github.sha }}
    commit_message: ${{ github.event.head_commit.message }}
    wait_time: '20'
  secrets:
    ssh_host: ${{ secrets.SSH_HOST }}
    ssh_user: ${{ secrets.SSH_USER }}
    ssh_key: ${{ secrets.SSH_PRIVATE_KEY }}
    ssh_port: ${{ secrets.SSH_PORT }}
    env_content: |
      KEY1=${{ secrets.KEY1 }}
      KEY2=${{ secrets.KEY2 }}
    grafana_api_key: ${{ secrets.GRAFANA_API_KEY }}
```

### Cleanup workflow
```yaml
jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - uses: StanislavPlotnikov/infra-ci/.github/actions/full-cleanup@master
        with:
          ssh_host: ${{ secrets.SSH_HOST }}
          ssh_user: ${{ secrets.SSH_USER }}
          ssh_key: ${{ secrets.SSH_PRIVATE_KEY }}
          ssh_port: ${{ secrets.SSH_PORT }}
```

## 🏗️ Architecture

All services follow the same deployment pattern:
1. ✅ Configs embedded in Docker image (single source of truth)
2. ✅ Image pushed to GHCR
3. ✅ On VPS: pull image → extract configs → docker compose up
4. ✅ No SCP, no base64 encoding, no file reading in workflows

## 📝 License

MIT