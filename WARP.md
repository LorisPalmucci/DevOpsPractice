# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a DevOps practice repository containing a simple Flask web application designed for learning containerization and deployment workflows. The application serves as a "Hello World" example to practice Docker, Docker Compose, and cloud deployment concepts.

## Architecture

### Application Stack
- **Framework**: Flask (Python web framework)
- **Python Version**: 3.11.0
- **Entry Point**: `devops.py` - contains a single Flask app with one route (`/`) that returns "Hello, World!"
- **Dependencies**: Managed in `requirements.txt` (currently only Flask)
- **Virtual Environment**: Uses `.venv` for local Python dependency isolation

### Containerization
- **Docker**: Multi-stage Dockerfile optimized for Python applications
- **Base Image**: `python:3.11.0-slim`
- **Working Directory**: `/` (root directory in container)
- **Exposed Port**: 5000
- **Security**: Runs as non-privileged user `appuser` (UID 10001)
- **Docker Compose**: Single service (`server`) configuration that maps port 5000:5000

### Project Structure
```
DevOpsPractice/
├── devops.py           # Flask application entry point
├── requirements.txt    # Python dependencies
├── Dockerfile          # Container build instructions
├── compose.yaml        # Docker Compose configuration
├── .dockerignore       # Files excluded from Docker build context
├── README.Docker.md    # Docker-specific documentation
└── .venv/             # Python virtual environment (gitignored)
```

## Common Development Commands

### Local Development
```powershell
# Activate virtual environment
.venv\Scripts\Activate.ps1

# Install dependencies
python -m pip install -r requirements.txt

# Run Flask application locally
flask --app devops.py run

# Access application
# http://localhost:5000
```

### Docker Development
```powershell
# Build and run with Docker Compose (recommended)
docker compose up --build

# Build Docker image manually
docker build -t devopspractice .

# Build for different platform (e.g., for cloud deployment)
docker build --platform=linux/amd64 -t devopspractice .

# Run container manually
docker run -p 5000:5000 devopspractice

# Stop and remove containers
docker compose down
```

### Deployment
```powershell
# Build for production (amd64 architecture)
docker build --platform=linux/amd64 -t myregistry.com/devopspractice .

# Push to registry
docker push myregistry.com/devopspractice
```

## Development Workflow

### Git Workflow
- **Main Branch**: `main` (deployment branch)
- **Commits**: Include co-author attribution when using Warp: `Co-Authored-By: Warp <agent@warp.dev>`

### Making Changes
1. Modify Python code in `devops.py` or add new modules
2. Update `requirements.txt` if adding new dependencies
3. Test locally with Flask development server
4. Test containerized version with `docker compose up --build`
5. Verify application responds correctly at `http://localhost:5000`

### Adding Dependencies
When adding new Python packages:
1. Add package name to `requirements.txt`
2. Rebuild Docker image to update dependencies
3. The Dockerfile uses Docker cache mounts for pip to speed up builds

## Key Architectural Patterns

### Docker Build Optimization
- Uses BuildKit cache mounts for pip dependencies (`--mount=type=cache,target=/root/.cache/pip`)
- Bind mounts `requirements.txt` during build to avoid copying into intermediate layers
- Multi-stage awareness (currently single stage but structured for expansion)

### Security Considerations
- Non-root user execution in container
- Minimal base image (`-slim` variant)
- `.dockerignore` excludes development files, secrets, and build artifacts
- Environment variables set for Python behavior:
  - `PYTHONDONTWRITEBYTECODE=1` - no .pyc files
  - `PYTHONUNBUFFERED=1` - immediate log output

### Port Configuration
- Flask default: 5000
- Docker container exposes: 5000
- Docker Compose maps: 5000:5000 (host:container)
- When modifying ports, update all three locations consistently

## Future Extension Points

The `compose.yaml` file includes commented examples for adding:
- PostgreSQL database service
- Volume mounts for data persistence
- Health checks
- Secrets management

When adding database or other services, uncomment and configure the relevant sections in `compose.yaml`.

## Windows-Specific Notes

This repository is developed on Windows with PowerShell:
- Use PowerShell commands for virtual environment activation (`.venv\Scripts\Activate.ps1`)
- Docker Desktop for Windows is required for containerization
- Line endings may be CRLF - Docker build handles this automatically
