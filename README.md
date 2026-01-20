# Self-Hosted GitHub Actions Runner (Organization Level)

This repository provides a containerized infrastructure solution for deploying self-hosted GitHub Actions runners. It is designed for organization-wide implementation, allowing multiple repositories to utilize a shared computing resource.

The configuration utilizes the **Docker-out-of-Docker (DooD)** methodology by binding the host's Docker socket to the runner container. This approach eliminates the need for privileged mode and enables the runner to spawn sibling containers directly on the host system, ensuring optimal performance and cache utilization.

## Architecture

* **Scope:** Organization Level (serves all repositories within the target GitHub Organization).
* **Isolation:** Runs within a lightweight container based on the `myoung34/github-runner` image.
* **Docker Integration:** Mounts `/var/run/docker.sock` to allow the runner to execute Docker commands on the host daemon.

## Prerequisites

1.  **Host System:** A server running Linux (Ubuntu/Debian recommended) with Docker Engine and Docker Compose installed.
2.  **GitHub Access Token:** A Personal Access Token (PAT) with the following scopes:
    * `repo` (Full control of private repositories)
    * `workflow` (Update GitHub Action workflows)
    * `admin:org` (Full control of organizations and teams - required for organization-level registration)

## Installation & Configuration

### 1. Clone the Repository
Clone this configuration to the standard `/opt` directory or your preferred infrastructure location.

```bash
git clone https://github.com/hasanalbayrak/github-organization-runner-docker.git
cd github-organization-runner-docker
```

### 2. Configure Environment Variables

Create a ```.env``` file from the example template to store sensitive credentials.
```bash
cp .env.example .env
nano .env
```

Populate the ```ACCESS_TOKEN``` variable with your GitHub Personal Access Token.

### 3. Update Service Configuration

Modify ```docker-compose.yml``` to reflect your organization's details:

* **ORG_NAME:** The target GitHub Organization name.
* **RUNNER_NAME:** A unique identifier for this runner instance.
* **RUNNER_LABELS:** Custom labels for targeting this runner in workflows (e.g., ```self-hosted, production, docker```).

### 4. Deployment

Initialize the runner service in detached mode.
```bash
docker compose up -d
```

## Usage
To utilize this runner in your CI/CD pipelines, update the ```runs-on``` property in your workflow YAML files.
```yaml
jobs:
  build_and_deploy:
    runs-on: [self-hosted, docker]
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
      
      - name: Build Docker Image
        run: docker build . -t my-application:latest
```

## Security Note

**Important:** Do not commit the ```.env``` file to version control. It contains sensitive authentication tokens. The ```.gitignore``` file included in this repository is configured to exclude it automatically.