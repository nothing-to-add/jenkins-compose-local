# Jenkins Local Development Environment

A Docker-based Jenkins setup for managing local development services and CI/CD pipelines.

## 📋 Prerequisites

### Install Docker Desktop
1. **Download Docker Desktop** for macOS from [docker.com](https://www.docker.com/products/docker-desktop)
2. **Install and run** Docker Desktop
3. **Verify installation**:
   ```bash
   docker --version
   docker-compose --version
   ```

### System Requirements
- **macOS**: 10.14 or later
- **RAM**: 4GB minimum (8GB recommended)
- **Storage**: 2GB free space for Jenkins container and volumes

## 🚀 Quick Start

### 1. Create Jenkins Directory
```bash
mkdir -p ~/Development/jenkins-local
cd ~/Development/jenkins-local
```

### 2. Create docker-compose.yml
Create a file named `docker-compose.yml` with the following content:

```yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins-local
    ports:
      - "8080:8080"      # Jenkins Web UI
      - "50000:50000"    # Jenkins Agent Port
    volumes:
      - jenkins_home:/var/jenkins_home                           # Jenkins data persistence
      - /var/run/docker.sock:/var/run/docker.sock              # Docker socket access
      - /Users/$USER/SwiftProjects:/var/projects               # Mount your projects
    restart: unless-stopped
    environment:
      - JENKINS_OPTS=--httpPort=8080
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false      # Skip setup wizard (optional)
    user: "1000:1000"                                          # Use your user ID

volumes:
  jenkins_home:
    name: jenkins-local-home
```

## 🎛️ Jenkins Management Commands

### Start Jenkins
```bash
# Start Jenkins in background
docker-compose up -d

# Start Jenkins with logs visible
docker-compose up
```

### Stop Jenkins
```bash
# Stop Jenkins (preserves data)
docker-compose down

# Stop and remove volumes (⚠️ destroys all data)
docker-compose down -v
```

### Check Status
```bash
# View running containers
docker-compose ps

# View Jenkins logs
docker-compose logs jenkins

# Follow logs in real-time
docker-compose logs -f jenkins
```

### Restart Jenkins
```bash
# Restart Jenkins container
docker-compose restart jenkins

# Or stop and start
docker-compose down && docker-compose up -d
```

## 🔧 Initial Setup

### 1. Start Jenkins
```bash
cd ~/Development/jenkins-local
docker-compose up -d
```

### 2. Access Jenkins Web UI
- Open browser to: **http://localhost:8080**
- Wait for Jenkins to start (may take 2-3 minutes)

### 3. Get Initial Admin Password
```bash
# Get the initial admin password
docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 4. Complete Setup Wizard
1. **Enter the admin password** from step 3
2. **Install suggested plugins** (recommended)
3. **Create admin user** (or continue as admin)
4. **Configure instance URL**: `http://localhost:8080`

## 📁 Project Integration

### Mount Your Projects
Your projects are automatically mounted at `/var/projects` inside Jenkins:

```
Host Path: /Users/[username]/SwiftProjects/
Container Path: /var/projects/
```

### Create Pipeline Jobs
1. **New Item** → **Pipeline**
2. **Pipeline Definition**: Pipeline script from SCM
3. **SCM**: None (use local files)
4. **Script Path**: `/var/projects/your-project/Jenkinsfile`

## 🛠️ Useful Docker Commands

### Container Management
```bash
# View all containers
docker ps -a

# Enter Jenkins container shell
docker-compose exec jenkins bash

# View container resource usage
docker stats jenkins-local
```

### Volume Management
```bash
# List Docker volumes
docker volume ls

# Inspect Jenkins home volume
docker volume inspect jenkins-local-home

# Backup Jenkins data
docker run --rm -v jenkins-local-home:/data -v $(pwd):/backup alpine tar czf /backup/jenkins-backup.tar.gz -C /data .
```

### Cleanup Commands
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes (⚠️ careful!)
docker volume prune
```

## 🔍 Troubleshooting

### Common Issues

#### Port 8080 Already in Use
```bash
# Find what's using port 8080
lsof -i :8080

# Use different port in docker-compose.yml
ports:
  - "8081:8080"  # Changed from 8080:8080
```

#### Permission Denied Errors
```bash
# Fix Docker socket permissions
sudo chmod 666 /var/run/docker.sock

# Or add user to docker group
sudo usermod -aG docker $USER
# Then logout and login again
```

#### Jenkins Won't Start
```bash
# Check logs for errors
docker-compose logs jenkins

# Check disk space
df -h

# Restart Docker Desktop and try again
```

### Reset Jenkins (⚠️ Destroys All Data)
```bash
# Stop and remove everything
docker-compose down -v

# Remove Jenkins image to force re-download
docker rmi jenkins/jenkins:lts

# Start fresh
docker-compose up -d
```

## 📊 Monitoring & Maintenance

### Check System Resources
```bash
# Docker system info
docker system df

# Container resource usage
docker stats --no-stream

# Jenkins container logs size
docker-compose exec jenkins du -sh /var/jenkins_home/logs
```

### Regular Maintenance
```bash
# Clean old Jenkins builds (inside Jenkins container)
docker-compose exec jenkins find /var/jenkins_home/jobs -name builds -exec rm -rf {}/[0-9]* \;

# Clean Docker system
docker system prune -f
```

## 🔐 Security Notes

### Production Considerations
- **Change default ports** if Jenkins will be accessible from network
- **Set up SSL/TLS** for external access
- **Configure firewall rules** appropriately
- **Regular backups** of Jenkins home volume
- **Update Jenkins regularly** for security patches

### Local Development
- Jenkins runs on `localhost:8080` by default
- Docker socket access allows container management
- Projects mounted read-only for safety (add `:ro` to volume if needed)

## 📚 Next Steps

1. **Create Jenkinsfile** in your project repositories
2. **Set up pipeline jobs** for each project
3. **Configure webhooks** for automatic builds
4. **Add monitoring** and notifications
5. **Create shared libraries** for common pipeline functions

## 🆘 Support

### Getting Help
- **Jenkins Documentation**: https://www.jenkins.io/doc/
- **Docker Compose Reference**: https://docs.docker.com/compose/
- **Docker Desktop Issues**: https://docs.docker.com/desktop/troubleshoot/

### Useful Resources
- Jenkins Pipeline Syntax: https://www.jenkins.io/doc/book/pipeline/syntax/
- Docker Best Practices: https://docs.docker.com/develop/dev-best-practices/
- Jenkins Blue Ocean: Modern Jenkins UI

---
📅 **Last Updated**: February 2026  
🏷️ **Version**: 1.0  
👨‍💻 **Author**: Local Development Setup
