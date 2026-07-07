# Dockerized Service Deployment

A Node.js/Express service with a public route and a Basic Auth–protected secret route, containerized with Docker and deployed via a GitHub Actions pipeline that builds the image, pushes it to GHCR, and deploys it to a remote server over SSH.

## Project URL
https://roadmap.sh/projects/dockerized-service-deployment

## Stack
- Node.js 20 (Alpine base image) + Express
- Docker
- GitHub Container Registry (GHCR)
- Deployed via SSH to a Terraform-provisioned AWS EC2 instance (see [devops-projects](https://github.com/WilliamFly/devops-projects/tree/main/18-dockerized-service-deployment))

## Endpoints

| Route | Behavior |
|-------|----------|
| `GET /` | Returns `Hello, world!` |
| `GET /secret` | Protected by hand-rolled HTTP Basic Auth. Returns `SECRET_MESSAGE` if credentials match `USERNAME`/`PASSWORD`; otherwise `401`. |

## Environment Variables

Copy `.env.example` to `.env` and fill in real values:

```
SECRET_MESSAGE=your_secret_message_here
USERNAME=your_username_here
PASSWORD=your_password_here
PORT=80
```

`.env` is git-ignored and Docker-ignored — it never gets committed or baked into the image. Values are injected at container runtime via `--env-file` (local) or `-e` flags (deployed).

## Local Development

```bash
npm install
PORT=3000 node index.js
```

## Docker

```bash
docker build -t dockerized-service-deployment .
docker run -d -p 3000:80 --env-file .env --name test-container dockerized-service-deployment
curl http://localhost:3000
curl -u <username>:<password> http://localhost:3000/secret
```

## CI/CD Pipeline

On every push to `main`, `.github/workflows/deploy.yml`:

1. **build-and-push** — builds the Docker image from the Dockerfile and pushes it to `ghcr.io/williamfly/dockerized-service-deployment`, authenticating with the workflow's built-in `GITHUB_TOKEN`
2. **deploy** — SSHs into the remote server (via `appleboy/ssh-action`), logs into GHCR, pulls the latest image, stops/removes any existing container, and starts a fresh one with secrets injected as environment variables

## Required GitHub Secrets

| Secret | Purpose |
|--------|---------|
| `SERVER_IP` | Public IP of the deployment target |
| `SSH_PRIVATE_KEY` | Key used to SSH into the server for deployment |
| `SECRET_MESSAGE` | Value returned by `/secret` |
| `APP_USERNAME` | Basic Auth username |
| `APP_PASSWORD` | Basic Auth password |
