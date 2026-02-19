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

### 1. Clone or Navigate to Repository
```bash
# Clone the repository
git clone <repository-url>
cd jenkins-compose-local

# Or if you already have it locally
cd path/to/jenkins-compose-local
```

### 2. Set Up Your Projects Directory
The `docker-compose.yml` file is configured to mount a local `projects` directory:

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
      - jenkins_home:/var/jenkins_home                          # Jenkins data persistence
      - /var/run/docker.sock:/var/run/docker.sock             # Docker socket access
      - ./projects:/var/projects                               # Mount local projects directory
    restart: unless-stopped
    environment:
      - JENKINS_OPTS=--httpPort=8080

volumes:
  jenkins_home:
```

**Create the projects directory:**
```bash
# Create projects directory in the same location as docker-compose.yml
mkdir -p projects

# Or create a symbolic link to your existing projects
ln -s /path/to/your/existing/projects projects
```

**Alternative configurations:**
```bash
# Option 1: Direct path mount (edit docker-compose.yml)
- /Users/yourusername/Projects:/var/projects     # macOS
- /home/yourusername/projects:/var/projects      # Linux
- C:\Users\yourusername\Projects:/var/projects   # Windows WSL

# Option 2: Use the included projects directory (recommended)
- ./projects:/var/projects
```

## 🎛️ Jenkins Management Commands & Strategy

### Docker Setup from Terminal

#### Prerequisites Check
```bash
# Verify Docker is installed and running
docker --version
docker-compose --version

# Check Docker daemon status
docker info

# Test Docker with hello-world (optional)
docker run hello-world
```

#### Initial Setup
```bash
# 1. Navigate to project directory
cd jenkins-compose-local

# 2. Set up your projects directory (choose one option)
mkdir -p projects  # Create new projects directory
# OR
ln -s /path/to/your/existing/projects projects  # Link existing projects

# 3. Verify docker-compose.yml exists
ls -la docker-compose.yml

# 4. Create and start Jenkins (first time)
docker-compose up -d

# 5. Verify Jenkins is starting
docker-compose logs -f jenkins
```

### Starting Jenkins

#### Development Mode (with logs)
```bash
# Start with logs visible (good for development/troubleshooting)
docker-compose up

# Use Ctrl+C to stop
```

#### Production Mode (background)
```bash
# Start Jenkins in detached mode
docker-compose up -d

# Verify it's running
docker-compose ps
```

#### Selective Start (if multiple services exist)
```bash
# Start only Jenkins service
docker-compose up -d jenkins
```

### Stopping Jenkins

#### Graceful Stop (Recommended)
```bash
# Stop Jenkins gracefully (preserves all data and configuration)
docker-compose stop

# Or stop specific service
docker-compose stop jenkins
```

#### Complete Shutdown
```bash
# Stop and remove containers (data persists in volumes)
docker-compose down

# Stop and remove containers with networks
docker-compose down --remove-orphans
```

#### Nuclear Option (⚠️ Data Loss)
```bash
# Stop and remove EVERYTHING including volumes (destroys all Jenkins data)
docker-compose down -v --remove-orphans

# Use only when you want to start completely fresh
```

### Jenkins Management Strategy

#### When to Start Jenkins
✅ **Start Jenkins when:**
- Beginning a development session
- Need to run CI/CD pipelines
- Setting up new projects
- Troubleshooting build issues
- Running automated tests

#### When to Stop Jenkins
🛑 **Stop Jenkins when:**
- Ending development session (save system resources)
- Computer going to sleep/hibernation
- Need to free up port 8080
- System maintenance or updates
- Not actively developing

#### Resource Management Strategy
```bash
# Check Jenkins resource usage
docker stats jenkins-local --no-stream

# Monitor system resources
docker system df

# View Jenkins container details
docker inspect jenkins-local | grep -A 5 -B 5 "Memory\|Cpu"
```

### Restart Strategies

#### Soft Restart (Configuration Changes)
```bash
# Restart just Jenkins service (preserves all data)
docker-compose restart jenkins

# Check if restart was successful
docker-compose logs --tail=50 jenkins
```

#### Hard Restart (System Issues)
```bash
# Complete stop and start cycle
docker-compose down
docker-compose up -d

# Verify Jenkins is healthy
curl -s http://localhost:8080/login > /dev/null && echo "Jenkins is up" || echo "Jenkins is down"
```

#### Emergency Restart (Corrupted State)
```bash
# Stop everything
docker-compose down

# Remove container (keeps data)
docker rm jenkins-local

# Restart from scratch
docker-compose up -d
```

### Status Monitoring

#### Quick Health Check
```bash
# One-line health check
docker-compose ps | grep jenkins-local | grep -q "Up" && echo "✅ Jenkins Running" || echo "❌ Jenkins Stopped"

# Detailed status
docker-compose ps
```

#### Detailed Monitoring
```bash
# Real-time logs
docker-compose logs -f jenkins

# Last 100 log lines
docker-compose logs --tail=100 jenkins

# Jenkins system info
curl -s http://localhost:8080/systemInfo
```

### Daily Workflow Examples

#### Morning Startup
```bash
cd jenkins-compose-local
docker-compose up -d
echo "Jenkins starting... Access at http://localhost:8080"
docker-compose logs -f jenkins | head -20  # Watch startup
```

#### Evening Shutdown
```bash
cd jenkins-compose-local
docker-compose stop
echo "Jenkins stopped gracefully. Data preserved."
```

#### Weekend Maintenance
```bash
# Navigate to project directory
cd jenkins-compose-local

# Stop Jenkins
docker-compose down

# Clean unused Docker resources
docker system prune -f

# Update Jenkins image
docker-compose pull

# Restart Jenkins
docker-compose up -d
```

## 🔧 Initial Setup

### 1. Start Jenkins
```bash
cd jenkins-compose-local
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

### Local Projects Mount
Your projects are automatically mounted at `/var/projects` inside Jenkins:

```
Host Path: ./projects/ (relative to docker-compose.yml)
Container Path: /var/projects/
```

This means any projects you place in the `projects` directory will be accessible to Jenkins at `/var/projects/`.

### Setting Up Your Projects

#### Option 1: Use Local Projects Directory (Recommended)
```bash
# Create projects directory
mkdir -p projects

# Copy or move your projects
cp -r /path/to/your/project projects/my-project

# Or create new projects directly
cd projects
git clone https://github.com/username/your-project.git
```

#### Option 2: Symbolic Link to Existing Projects
```bash
# Create symbolic link to your existing projects directory
ln -s /path/to/your/existing/projects projects

# Examples:
ln -s ~/Developer projects                    # macOS
ln -s ~/workspace projects                    # Linux
ln -s /c/Users/username/Projects projects     # Windows WSL
```

#### Option 3: Direct Path Mount (Advanced)
Edit `docker-compose.yml` to mount a specific directory:

```yaml
volumes:
  # Replace with your actual path
  - /Users/username/Projects:/var/projects     # macOS
  - /home/username/projects:/var/projects      # Linux  
  - C:\Users\username\Projects:/var/projects   # Windows
```

Then restart Jenkins:
```bash
docker-compose down
docker-compose up -d
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

#### Docker Not Running
```bash
# Check if Docker Desktop is running
docker info
# If error: "Cannot connect to the Docker daemon"
# → Start Docker Desktop application

# For macOS, you can also use:
open /Applications/Docker.app
```

#### Port 8080 Already in Use
```bash
# Find what's using port 8080
lsof -i :8080

# Kill the process using the port (replace PID)
kill -9 <PID>

# Or use different port in docker-compose.yml
ports:
  - "8081:8080"  # Access Jenkins at http://localhost:8081
```

#### Jenkins Container Won't Start
```bash
# Check detailed error logs
docker-compose logs jenkins

# Common solutions:
# 1. Restart Docker Desktop
# 2. Check disk space: df -h
# 3. Remove and recreate container
docker-compose down
docker-compose up -d --force-recreate
```

#### Permission Issues with Docker Socket
```bash
# On macOS, this usually isn't needed, but if you get permission errors:
# Check Docker socket permissions
ls -la /var/run/docker.sock

# If needed, fix permissions (rarely required on macOS)
sudo chmod 666 /var/run/docker.sock
```

#### Swift Projects Not Visible in Jenkins
```bash
# 1. Verify the projects directory exists
ls -la projects/

# 2. Check if mounted correctly inside container
docker-compose exec jenkins ls -la /var/projects/

# 3. If projects directory doesn't exist, create it
mkdir -p projects

# 4. Add some test content
echo "Test project" > projects/test.txt
docker-compose exec jenkins cat /var/projects/test.txt
```

#### Jenkins Takes Too Long to Start
```bash
# Monitor startup progress
docker-compose logs -f jenkins

# Typical startup takes 1-3 minutes
# Look for "Jenkins is fully up and running" message
```

### Advanced Troubleshooting

#### Container Health Check
```bash
# Check container health
docker inspect jenkins-local --format='{{.State.Health.Status}}'

# View resource usage
docker stats jenkins-local --no-stream
```

#### Network Issues
```bash
# Test Jenkins connectivity
curl -I http://localhost:8080

# Check container networking
docker-compose exec jenkins ping google.com
```

#### Reset Jenkins (⚠️ Destroys All Data)
```bash
# Complete reset - removes all Jenkins configuration, jobs, and data
docker-compose down -v
docker rmi jenkins/jenkins:lts
docker-compose up -d

# This will require going through initial setup again
```

## � Automation & Scripts

### Quick Commands Aliases
Add these to your shell profile (`~/.bashrc`, `~/.zshrc`, or `~/.profile`) for quick Jenkins management:

```bash
# Jenkins aliases - Update the path to match your repository location
JENKINS_DIR="$(pwd)"  # Assumes you're in the jenkins-compose-local directory
# Or set absolute path: JENKINS_DIR="/path/to/jenkins-compose-local"

alias jenkins-start="cd $JENKINS_DIR && docker-compose up -d"
alias jenkins-stop="cd $JENKINS_DIR && docker-compose stop"
alias jenkins-restart="cd $JENKINS_DIR && docker-compose restart jenkins"
alias jenkins-logs="cd $JENKINS_DIR && docker-compose logs -f jenkins"
alias jenkins-status="cd $JENKINS_DIR && docker-compose ps"
alias jenkins-health='curl -s http://localhost:8080/login > /dev/null && echo "✅ Jenkins is running" || echo "❌ Jenkins is down"'
```

**For a more robust setup, use absolute paths:**
```bash
# Replace with your actual path to the repository
JENKINS_DIR="/Users/yourusername/path/to/jenkins-compose-local"
# or
JENKINS_DIR="$HOME/Developer/jenkins-compose-local"
```

After adding these, reload your shell:
```bash
source ~/.zshrc    # for zsh
source ~/.bashrc   # for bash
```

Then you can use simple commands:
```bash
jenkins-start    # Start Jenkins
jenkins-stop     # Stop Jenkins
jenkins-restart  # Restart Jenkins
jenkins-logs     # View logs
jenkins-status   # Check status
jenkins-health   # Quick health check
```

### Automated Startup Script
Create a script for automated Jenkins startup with health checks:

```bash
# Create bin directory if it doesn't exist
mkdir -p ~/bin

# Create the script
cat > ~/bin/jenkins-dev-start.sh << 'EOF'
#!/bin/bash

# Update this path to match your jenkins-compose-local directory
JENKINS_DIR="$HOME/path/to/jenkins-compose-local"
MAX_WAIT=180  # 3 minutes timeout

echo "🚀 Starting Jenkins development environment..."

# Navigate to Jenkins directory
cd "$JENKINS_DIR" || {
    echo "❌ Jenkins directory not found: $JENKINS_DIR"
    echo "Please update JENKINS_DIR in this script to match your setup"
    exit 1
}

# Check if Docker is running
if ! docker info >/dev/null 2>&1; then
    echo "❌ Docker is not running. Please start Docker Desktop."
    exit 1
fi

# Start Jenkins
echo "📦 Starting Jenkins container..."
docker-compose up -d

# Wait for Jenkins to be ready
echo "⏳ Waiting for Jenkins to start..."
START_TIME=$(date +%s)

while ! curl -s http://localhost:8080/login > /dev/null; do
    CURRENT_TIME=$(date +%s)
    ELAPSED=$((CURRENT_TIME - START_TIME))
    
    if [ $ELAPSED -gt $MAX_WAIT ]; then
        echo "❌ Jenkins failed to start within ${MAX_WAIT} seconds"
        echo "📋 Check logs with: docker-compose logs jenkins"
        exit 1
    fi
    
    echo "   Still starting... (${ELAPSED}s elapsed)"
    sleep 5
done

echo "✅ Jenkins is ready!"
echo "🌐 Access Jenkins at: http://localhost:8080"
echo "📊 View logs with: docker-compose logs -f jenkins"
echo "🛑 Stop with: docker-compose stop"
EOF

# Make executable
chmod +x ~/bin/jenkins-dev-start.sh

# Add to PATH if needed
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
# or for zsh users:
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
```

**Important**: Update the `JENKINS_DIR` variable in the script to match your actual repository location.

### Daily Development Workflow Scripts

#### Morning Development Start
```bash
# Create morning startup script
cat > ~/bin/dev-morning.sh << 'EOF'
#!/bin/bash
echo "🌅 Good morning! Starting development environment..."

# Start Jenkins
jenkins-start

# Show status
echo "📊 Development Environment Status:"
jenkins-status
jenkins-health

# Open Jenkins in browser (optional)
# open http://localhost:8080
EOF

chmod +x ~/bin/dev-morning.sh
```

#### Evening Development End
```bash
# Create evening shutdown script
cat > ~/bin/dev-evening.sh << 'EOF'
#!/bin/bash
echo "🌙 Ending development session..."

# Stop Jenkins gracefully
jenkins-stop

# Show Docker resource cleanup
echo "🧹 Cleaning up Docker resources..."
docker system prune -f > /dev/null 2>&1

echo "✅ Development environment stopped cleanly"
echo "💾 All Jenkins data preserved for tomorrow"
EOF

chmod +x ~/bin/dev-evening.sh
```

## �📊 Monitoring & Maintenance

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
📅 **Last Updated**: February 19, 2026  
🏷️ **Version**: 2.0  
👨‍💻 **Repository**: jenkins-compose-local  
🐳 **Docker Compose**: Generic setup for local development
