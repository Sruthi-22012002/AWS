<div align ="center"><h2>Gitlab CICD</h2></div>

## 📑 Table of Contents

1. [Overview](#overview)  
2. [Features](#features)  
3. [Installation Methods](#installation-methods)  
   3.1 [Method 1: WSL Installation](#method-1-wsl-installation)  
   - [Step 1: Add GitLab Runner Repository](#step-1-add-gitlab-runner-repository)  
   - [Step 2: Install GitLab Runner](#step-2-install-gitlab-runner)  
   - [Step 3: Verify Installation](#step-3-verify-installation)  
   - [Step 4: Register the Runner](#step-4-register-the-runner)  
3.2 [Method 2: Docker Installation](#method-2-docker-installation)  
   - [Step 1: Create Docker Volumes](#step-1-create-docker-volumes)  
   - [Step 2: Run GitLab Container](#step-2-run-gitlab-container)  
   - [Step 3: Access GitLab in Browser](#step-3-access-gitlab-in-browser)  
   - [Step 4: GitLab Admin Login](#step-4-gitlab-admin-login)  
   - [Step 5: Stop or Remove GitLab Container](#step-5-stop-or-remove-gitlab-container)

## Overview
* CI/CD is a continuous method of software development, where you continuously build, test, deploy, and monitor iterative code changes.
* This iterative process helps reduce the chance that you develop new code based on buggy or failed previous versions.
* GitLab CI/CD can catch bugs early in the development cycle, and help ensure that the code deployed to production complies with your established code standards.

### This process is part of a larger workflow:
![image](https://github.com/user-attachments/assets/fbe92e84-5a99-44bd-815d-8fa2c088b5ec)

## Features 
1. Free & open-source
2. Manual & scheduled pipelines
3. CI/CD variables (for secrets & configurations)
4. Job artifacts & caching
5. Docker integration
6. Environments & deployment tracking
7. Auto DevOps (with customization)
8. GitOps with pull-based deploy strategies (via custom scripting)

## Installation methods 
### Method 1: WSL
#### Step 1. Add the official GitLab Runner repository
```
# Install required packages for HTTPS
sudo apt-get update
sudo apt-get install -y curl gnupg ca-certificates

# Add the GitLab official repository GPG key
curl -fsSL https://packages.gitlab.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/gitlab-runner-archive-keyring.gpg

# Add the repository to your sources list
echo "deb [signed-by=/usr/share/keyrings/gitlab-runner-archive-keyring.gpg] https://packages.gitlab.com/runner/gitlab-runner/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gitlab-runner.list

# Update package lists again
sudo apt-get update
```
#### Step 2. Install GitLab Runner
```
sudo apt-get install gitlab-runner
```
#### Step 3. Verify installation and check version
```
gitlab-runner --version
```
#### Step 4. Register the runner
```
sudo gitlab-runner register
```
### Method 2 : Install GitLab in a Docker container
> you need to set up GitLab after your instance is created and ready.
#### Step 1: Create Docker Volume for Persistence
```
docker volume create gitlab-config
docker volume create gitlab-logs
docker volume create gitlab-data
```
#### Step 2: Run GitLab Container
```
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume gitlab-config:/etc/gitlab \
  --volume gitlab-logs:/var/log/gitlab \
  --volume gitlab-data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
> Replace `gitlab.example.com` with your server's `domain` or `IP address`.
#### Step 3: Access GitLab
* Open your browser and navigate to:
> http://<your-server-ip> or https://<your-server-domain>
* The first time you access GitLab, you'll be prompted to set the root password.
#### Step 5: GitLab Admin Login
* Username: root
* Password: (set during the first-time access)
#### Step 6: Stop and Remove GitLab Container (Optional)
##### To stop GitLab:
```
docker stop gitlab
```
##### To remove gitlab
```
docker rm gitlab
```


