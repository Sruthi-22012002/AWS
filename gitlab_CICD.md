<div align ="center"><h2>Gitlab CICD</h2></div>

* GitLab is a complete DevSecOps platform, delivered as a single application, that enables teams to collaborate on the entire software  development lifecycle—from planning and coding to testing, deployment, and monitoring.
* CI/CD is a continuous method of software development, where you continuously build, test, deploy, and monitor iterative code changes.
* This iterative process helps reduce the chance that you develop new code based on buggy or failed previous versions.
* GitLab CI/CD can catch bugs early in the development cycle, and help ensure that the code deployed to production complies with your established code standards.

## GitLab CI/CD Highlights
* Uses .gitlab-ci.yml to define pipeline stages and jobs.
* Supports multiple executors: Shell, Docker, Kubernetes, etc.
* Includes features like:
  * Parallel jobs
  * Manual approvals
  * Environment deployments (Dev, Staging, Prod)
  * Artifacts and caching

## 📚 Table of Contents

1. [Overview](#overview)  
2. [Features](#features)  
3. [Installation Methods](#installation-methods)  
   - [3.1 Method 1: WSL Installation](#manual-wsl)  
     - [Step 1: Add GitLab Runner Repository](#step-1-add-gitlab-runner-repository)  
     - [Step 2: Install GitLab Runner](#step-2-install-gitlab-runner)  
     - [Step 3: Verify Installation](#step-3-verify-installation)  
     - [Step 4: Register the Runner](#step-4-register-the-runner)  
   - [3.2 Method 2: Docker Installation](#method-2-docker-installation)  
     - [Step 1: Create Docker Volume for Persistence](#step-1-create-docker-volume-for-persistence)  
     - [Step 2: Run GitLab Container](#step-2-run-gitlab-container)  
     - [Step 3: Access GitLab](#step-3-access-gitlab)  
     - [Step 4: GitLab Admin Login](#step-4-gitlab-admin-login)  
     - [Step 5: GitLab Admin Login Error Fix](#step-5-gitlab-admin-login-error-fix)  
     - [Step 6: Install GitLab Runner](#step-6-install-gitlab-runner)  
     - [Step 7: Check GitLab Runner Version](#step-7-check-gitlab-runner-version)  
     - [Step 8: Register the Runner with GitLab CE](#step-8-register-the-runner-with-gitlab-ce)  

4. [GitLab Basics](#gitlab-basics)  
   - [How to create a project](#how-to-create-a-project)  
   - [How to create a SSH Key](#how-to-create-a-ssh-key)  
   - [How to clone the repository](#how-to-clone-the-repository)  
   - [What is GitLab Runner](#what-is-gitlab-runner)  
   - [How to create a GitLab Runner](#how-to-create-a-gitlab-runner)  
   - [How to configure the runner with GitLab](#how-to-configure-the-runner-with-gitlab)  
   - [How to create token](#how-to-create-token)  
   - [How to add variables](#how-to-add-variables)  
   - [Write a CI/CD sample file](#write-a-cicd-sample-file)  
   - [How it works](#how-it-works)  
     - [Pipeline Overview](#pipeline-overview)  
     - [Jobs Breakdown](#jobs-breakdown)  
       - [build-job](#build-job)  
       - [test-job1](#test-job1)  
       - [test-job2](#test-job2)  
       - [deploy-prod](#deploy-prod)  
   - [Check the status of the job](#check-the-status-of-the-job)  

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
| Volume Name    | Purpose                                                | Mounted Path in Container  |
|----------------|--------------------------------------------------------|-----------------------------|
| `gitlab-config`| Stores GitLab's configuration files                    | `/etc/gitlab`               |
| `gitlab-logs`  | Stores GitLab's logs (app, nginx, etc.)                | `/var/log/gitlab`           |
| `gitlab-data`  | Stores repositories, database, uploads, and artifacts  | `/var/opt/gitlab`           |


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
**Project → Settings → CI/CD → Runners
* Tag : docker-runner
* Select `Create runner `
![image](https://github.com/user-attachments/assets/01b43bfa-fce3-4b3a-b67d-57ff87b35b56)

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
![image](https://github.com/user-attachments/assets/da2d5add-81dc-4c33-a2c8-c9dba75a7830)
---

## How to create a project
* Click `project` in the right-side menu
* Click `New project` and choose a namespace
* Choose `Blank project`
* Enter the repository namespace `EMS Application`
* Choose visibility level `public,private,internal`
* Click `create project`

![projects](https://github.com/user-attachments/assets/be39e0ef-c6e9-4dee-8777-0454ca836484)

## How to create a SSH Key
* Login into the instance in WSL
* Generate a new SSH Key
```
ssh-keygen -t rsa -b 4096 -C sruthi@jumisa.io
```
* Copy the public Key
```
cat ~/.ssh/id_ed25519.pub
```
* Search `User setting` in navigation menu
* Click `SSH key` and paste the key
* Click `Add SSH key`.

![SSH keys](https://github.com/user-attachments/assets/827576c6-3b71-49d4-9007-0a96566c0b3f)

## How to clone the repository
* Go to `projects`
* Select the project `EMS Application`
* Copy `http : http://13.208.161.102/Sruthi22012002/ems-application.git`
* Paste in the EC2 instance.

![clone](https://github.com/user-attachments/assets/4b54abf5-e730-45d0-a250-e1b648616735)

## What is Gitlab runner
* A GitLab Runner is an application that executes CI/CD jobs defined in your GitLab CI/CD pipelines. 
* It acts as a worker machine, picking up jobs and running them based on the configuration defined in the .gitlab-ci.yml file. 
* Runners can run jobs in various environments, including Docker, shell, and Kubernetes. 

## How to create a gitlab runner
* Search for `Settings -> CI/CD`
* Tags : choose `Run untagged jobs`
* Runner name : (anything)
* select create runner
* Copy URL, Registeration token in the view runners page.

![runner](https://github.com/user-attachments/assets/31a8a889-d813-49d5-a051-e6add16f17be)

## How to configure the runner with gitlab 
* CLI command
```
sudo gitlab-runner register
```
* Enter the GitLab instance URL (for example, https://gitlab.com/):
* Enter the registration token: paste the token
* Enter a name for the runner. (anything)
* Enter an executor: docker+machine, custom, shell, ssh, parallels, virtualbox, docker, docker-windows, docker-autoscaler, instance, kubernetes:
**Choose `shell`**
> Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!

## How to create token 
* Search for `user setting` 
* Select `Access Token` and select `Add Token`
* Enter the `Token name`
* select scope `API, Write repository, Read repository`
* click `create personal access token.`

![PAT](https://github.com/user-attachments/assets/9977d9f1-3a60-49e7-8893-ec6fa8d2f8e5)

## How to add variables
* Click `setting` and select `CI/CD`
* Select `variables` and click `Add Variable`.
* Enter:
  * Key (e.g., AWS_ACCESS_KEY_ID)
  * Value (e.g., AKIA...)
  * Optionally mark it as Protected (for protected branches/tags only).
  * Optionally mark it as Masked (will be hidden in job logs).
* click `Add`

![variables](https://github.com/user-attachments/assets/0c7f1dfe-8a02-4a63-9809-ae19f721047d)

## write a CI/CD sample file
```
nano .gitlab-ci.yml
```
```
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"

test-job1:
  stage: test
  script:
    - echo "This job test something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20

deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
  environment: production
```

## How it works
### Pipeline Overview
```
stages:
  - build
  - test
  - deploy
```
### Jobs Breakdown
#### build-job
This job runs in the build stage. It greets the GitLab user who triggered the pipeline.
```
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"
```
### test-job1
A simple test job in the test stage.
```
test-job1:
  stage: test
  script:
    - echo "This job test something"
```
### test-job2
A longer test job that simulates a test running 20 seconds.
```
test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - sleep 20
```
### deploy-prod
This job handles deployment to the production environment.
```
deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
  environment: production
```

## Check the status of the job
*  Select the project
* Select `Build` and click `jobs`

![jobs](https://github.com/user-attachments/assets/b437c4f3-d2fc-41f9-9e26-d3ab69f9f569)

---
<div align="center"><h2>Terraform CI/CD with GitLab</h2></div>

This guide explains how GitLab CI/CD works with Terraform using `.gitlab-ci.yml`.
```
stages:
  - init
  - plan
  - apply

variables:
  TF_ROOT: "."
  TF_VERSION: "1.6.6"
  AWS_ACCESS_KEY_ID: "$AWS_ACCESS_KEY_ID"
  AWS_SECRET_ACCESS_KEY: "$AWS_SECRET_ACCESS_KEY"
  AWS_DEFAULT_REGION: "$AWS_DEFAULT_REGION"

default:
  image:
    name: hashicorp/terraform:${TF_VERSION}
    entrypoint: [""]

before_script:
  - terraform --version
  - cd $TF_ROOT

init:
  stage: init
  script:
    - terraform init
  artifacts:
    paths:
      - .terraform.lock.hcl
    expire_in: 1 hour

plan:
  stage: plan
  script:
    - rm -rf .terraform .terraform.lock.hcl
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - tfplan
      - .terraform.lock.hcl
    expire_in: 1 hour

apply:
  stage: apply
  script:
    - rm -rf .terraform
    - terraform init
    - terraform apply -auto-approve tfplan
  when: manual
```
## Overview of Flow

- GitLab triggers the pipeline when you push changes to the repository.
- Jobs are executed in stages: `init` → `plan` → `apply`.
- Each job runs Terraform commands (`terraform init`, `terraform plan`, `terraform apply`) inside a Docker container using the image `hashicorp/terraform:1.6.6`.

## Breakdown of Each Section
### stages

Defines the order in which jobs will run:

```yaml
stages:
  - init
  - plan
  - apply
```
* These stages correspond to Terraform commands:
    - init: sets up Terraform (downloads providers, modules)
    - plan: shows what changes will be made
    - apply: applies those changes (manual trigger)
### variables
```
variables:
  TF_ROOT: "."
  TF_VERSION: "1.6.6"
  AWS_ACCESS_KEY_ID: "$AWS_ACCESS_KEY_ID"
  AWS_SECRET_ACCESS_KEY: "$AWS_SECRET_ACCESS_KEY"
  AWS_DEFAULT_REGION: "$AWS_DEFAULT_REGION"
```
* These provide:
    - TF_ROOT: Terraform root directory
    - TF_VERSION: used to dynamically pull the correct Docker image
    - AWS credentials: injected from GitLab CI/CD → Settings → CI/CD → Variables

![terraform variables](https://github.com/user-attachments/assets/e2838831-98bc-4833-8ef7-aa5f377e0c09)

### default
```
default:
  image:
    name: hashicorp/terraform:${TF_VERSION}
    entrypoint: [""]
```
* All jobs use `hashicorp/terraform:1.6.6` Docker image.
### before_script
```
before_script:
  - terraform --version
  - cd $TF_ROOT
```
* Ensures Terraform is installed and switches to the correct working directory.
### init Job
```
init:
  stage: init
  script:
    - terraform init
  artifacts:
    paths:
      - .terraform.lock.hcl
    expire_in: 1 hour
```
* Initializes the Terraform working directory.
* Downloads providers and modules.
* Saves .terraform.lock.hcl as an artifact for later jobs.
### plan Job
```
plan:
  stage: plan
  script:
    - rm -rf .terraform .terraform.lock.hcl
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - tfplan
      - .terraform.lock.hcl
    expire_in: 1 hour
```
* Deletes existing .terraform and lock file to start clean.
* Re-initializes and creates a plan file (tfplan).
* Saves tfplan and lock file as artifacts for the apply step.
### apply Job (Manual)
```
apply:
  stage: apply
  script:
    - rm -rf .terraform
    - terraform init
    - terraform apply -auto-approve tfplan
  when: manual
```
* Re-initializes
* Applies changes using the tfplan file
* Manual trigger: must be run manually from GitLab UI to prevent unintended deployment





