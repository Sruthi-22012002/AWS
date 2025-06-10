<div align ="center"><h2>Gitlab CICD</h2></div>

## 📑 Table of Contents

1. [Overview](#overview)  
2. [Features](#features)  
3. [Installation Methods](#installation-methods)  
   - [3.1 Method 1: WSL Installation](#manual-wsl)  
     - [Step 1: Add GitLab Runner Repository](#step-1-add-the-official-gitlab-runner-repository)  
     - [Step 2: Install GitLab Runner](#step-2-install-gitlab-runner)  
     - [Step 3: Verify Installation](#step-3-verify-installation-and-check-version)  
     - [Step 4: Register the Runner](#step-4-register-the-runner)  
   - [3.2 Method 2: Docker Installation](#method-1-install-gitlab-in-a-docker-container)  
     - [Step 1: Create Docker Volume for Persistence](#step-1-create-docker-volume-for-persistence)  
     - [Step 2: Run GitLab Container](#step-2-run-gitlab-container)  
     - [Step 3: Access GitLab](#step-3-access-gitlab)  
     - [Step 4: GitLab Admin Login](#step-5-gitlab-admin-login)  
     - [Step 5: GitLab Admin Login Error Fix](#solution--login-manually-by-rails)  
     - [Step 6: Install GitLab Runner](#step-6-install-gitlab-runner)  
     - [Step 7: Check GitLab Runner Version](#step-7-check-gitlab-runner-version)  
     - [Step 8: Register the Runner with GitLab CE](#step-8-register-the-runner-with-gitlab-ce)  


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
### Manual: WSL
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
### Cloud provider(AWS) 
> you need to set up GitLab after your instance is created and ready and check with the browser
![image](https://github.com/user-attachments/assets/f9fff3dc-07a5-4bf6-a480-de849191b6fa)

> Login your instance in wsl
```
ssh -i "aws-ec2-key.pem" ubuntu@ec2-13-208-161-102.ap-northeast-3.compute.amazonaws.com
```
#### Method 1: Install GitLab in a Docker container
##### Step 1: Create Docker Volume for Persistence
```
sudo docker volume create gitlab-config
sudo docker volume create gitlab-logs
sudo docker volume create gitlab-data
```
##### Step 2: Run GitLab Container
```
sudo docker run --detach \
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

![image](https://github.com/user-attachments/assets/7bca612c-d7db-485c-a39d-c86e904aee6d)

* check disk space
```
 df -h
```
* Check the folder consuming large amount of disk spcae and remove the unwanted image, volume, or container
```
sudo docker system prune -a --volumes
```
* Modify the volume size in EC2
* Expand the partition to use full size
```
sudo growpart /dev/xvda 1`
```
* Expand the file system
```
sudo resize2fs /dev/xvda1`
```
* confirm the space
```
df -h
```
##### Step 3: Access GitLab
* Open your browser and navigate to:
> http://<your-server-ip> or https://<your-server-domain>

![image](https://github.com/user-attachments/assets/3e7819ca-1a92-48e1-b513-a1e0005fd050)

##### Register now
![image](https://github.com/user-attachments/assets/e6b4a33a-9d44-48db-89b2-2c59369e0673)

* The first time you access GitLab, you'll be prompted to set the root password.
##### Step 5: GitLab Admin Login
###### Login error
> Your account is pending approval from your GitLab administrator and hence blocked. Please contact your GitLab administrator if you think this is an error.
**Solution : Login manually by Rails**
> gitlab-rails is a command-line tool used inside the GitLab container to interact with the GitLab Rails application backend (since GitLab is built using Ruby on Rails).

> Disable if admin enable
```
sudo docker exec -it gitlab-new gitlab-rails runner "ApplicationSetting.last.update(require_admin_approval_after_user_signup: false)"
```
> one-line command to approve user manually :
```
sudo docker exec -it gitlab-new gitlab-rails runner "user = User.find_by(email: 'sruthi@jumisa.io'); user.confirmed_at = Time.now; user.save!"
```
> Active user
```
sudo docker exec -it gitlab-new gitlab-rails runner "user = User.find_by(email: 'sruthi@jumisa.io'); user.state = 'active'; user.save!"
```
![image](https://github.com/user-attachments/assets/ef183d61-611b-4cdb-83a8-ea2f6b58da2f)

> Change yourself to admin to access the page
```
sudo docker exec -it gitlab-new gitlab-rails runner "user = User.find_by(email: 'sruthi@jumisa.io'); user.admin = true; user.save!"
```
![image](https://github.com/user-attachments/assets/a790bd64-af71-42d7-8dca-9cf01d74b9a5)

> Access URL
```
http://15.152.37.166/admin
```
![image](https://github.com/user-attachments/assets/03378e06-08a7-466f-9e7d-286926cd8cd9)

##### Step 6: Install GitLab Runner
```
# 1. Install required packages
sudo apt-get update
sudo apt-get install -y curl gnupg ca-certificates

# 2. Add the GitLab official GPG key
curl -fsSL https://packages.gitlab.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/gitlab-runner-archive-keyring.gpg

# 3. Add GitLab repository to APT
echo "deb [signed-by=/usr/share/keyrings/gitlab-runner-archive-keyring.gpg] https://packages.gitlab.com/runner/gitlab-runner/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gitlab-runner.list

# 4. Update and install GitLab Runner
sudo apt-get update
sudo apt-get install gitlab-runner -y
```
##### Step 7: Check GitLab Runner Version
```
gitlab-runner --version
```
![image](https://github.com/user-attachments/assets/ae578e85-1a4a-4047-be73-a0f60b9ca450)

##### Step 8: Register the Runner with GitLab CE
* Go to your GitLab CE web interface
* Navigate to:
**Project → Settings → CI/CD → Runners → Expand**
* Copy the registration token
* Run this command on the EC2 instance:
```
sudo gitlab-runner register
```
> Provide these details when prompted:

   > GitLab URL: http://<your-gitlab-server> (use your GitLab CE IP/domain)

> Registration token: (paste from GitLab)

> Description: e.g., ec2-runner

> Tags: ec2

> Executor: Choose shell

