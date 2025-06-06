<div align ="center"><h2>EMS OPS Application</h2></div>

## Archietecture Diagram
![image](https://github.com/user-attachments/assets/8a3adff1-e423-4ad8-8bd8-34f7ed79be5d)

## 🚀 Launching an EC2 Instance on AWS Console

This guide provides a step-by-step walkthrough for launching a virtual server (EC2 instance) on Amazon Web Services (AWS) using the AWS Management Console.

### 🖥️ Step 1: Log in to AWS Console

1. Go to [https://console.aws.amazon.com](https://console.aws.amazon.com)
2. Sign in with your AWS account credentials.

### 🏗️ Step 2: Navigate to EC2 Dashboard

1. In the top search bar, type **EC2** and select **EC2** from the services list.
2. You will be directed to the **EC2 Dashboard**.

### 🛠️ Step 3: Launch a New Instance

1. Click **Launch instance**
2. Fill in the following details:

#### 📝 Instance Configuration

- **Name**: `my-ec2-instance` (or any name)
- **Application and OS Images (AMI)**: Choose Ubuntu 22.04 LTS
- **Instance type**: `t2.micro` (Free Tier eligible)
- **Key pair (login)**:
  - Create a new key pair or use an existing one
  - If creating new: 
    - Set a name
    - Choose **.pem** format
    - Click **Create key pair** and **download the file**
- **Network settings**:
  - Select a **VPC** and **subnet**
  - Check **Auto-assign Public IP**
  - Add a **Security Group** rule:
    - Type: SSH — Port: 22 — Source: My IP
    - (Optional) HTTP — Port: 80 — Source: 0.0.0.0/0

### 📦 Step 4: Add Storage (Optional)

- Default storage is 8 GB gp3 (general purpose SSD)
- Modify only if needed
### ✅ Step 5: Launch the Instance

1. Review all settings
2. Click **Launch instance**
3. Wait for a few seconds; then click **View Instances** to monitor the status
### 🔑 Step 6: Connect to EC2 Instance

Once the instance state is **running** and status checks are passed:
![image](https://github.com/user-attachments/assets/cc213b70-4d8e-4d75-9971-5a42af3a1fc0)

## 🛠️ Create RDS Instance

1. Go to **AWS Management Console** → Navigate to **RDS**.
2. Click **Create database**.
3. Under **Database creation method**, choose:  
   ✅ **Standard create**
4. **Engine options**:  
   Select **MySQL**.
5. **Version**:  
   Choose the latest **MySQL** version (recommended).
6. **Templates**:  
   Choose **Free tier** (if eligible) or **Production/Dev**.
7. **DB Instance Identifier**:  
   Example: `database-1`
8. **Credentials**:
   - Master Username: `admin`
   - Master Password: `yourpassword`
   - Confirm Password
9. **DB instance size**:  
   Free tier: `db.t3.micro`
10. **Storage**:  
    Leave default (e.g., 20 GB)
11. **Connectivity**:
    - **VPC**: Select existing or default VPC
    - **Subnet group**: default
    - **Public access**: `Yes`
    - **VPC security group**:  
      - Select an existing security group  
      - Or create a new one and allow **port 3306 (MySQL)** inbound access from:
        - Your EC2 instance's **security group**, or  
        - Your **public IP**
12. Click **Create database**
![image](https://github.com/user-attachments/assets/5aefe76f-f8f1-41f9-b150-a82a352e2e93)

---
## Clone and run the application in docker
### Prerequities
#### 1. Docker Installation
```
# 1. Update packages
sudo apt update && sudo apt upgrade -y

# 2. Install prerequisites
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# 3. Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 4. Set up Docker repo
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# 6. Start Docker (doesn’t persist in WSL)
sudo service docker start

# 7. Run test container
sudo docker run hello-world
```
#### 2. MYSQL Installation
```
sudo apt install -y mysql-client
```
---

### Step 1 : Clone from github
```
git clone https://github.com/vignesh-0002/EMS-ops.git
```
### Step 2 : Switch branch
```
cd EMS-OPS/
git checkout phase-0
```
### Step 3 : Create a Docker Image 
> Create a frontend immage `nano Dockerfile`

#### Forntend Image
```
# Use a base image with Node.js installed
FROM node:14-alpine
 
# Set the working directory in the container
WORKDIR /react-hooks-frontend/
 
# Copy package.json
COPY package*.json ./
 
## Optional: Set environment variables (these can be overridden at runtime)
 
ARG REACT_APP_BACKEND_URL
ENV REACT_APP_BACKEND_URL=${REACT_APP_BACKEND_URL}
 
 
# Install dependencies
RUN npm install
 
# Copy the source code
COPY . .
 
# Expose port 3000
EXPOSE 3000
 
# Command to run
ENTRYPOINT ["npm","start"]
```
### Change the path to access backend
> Navigate to `cd react-hooks-frontend/src/service`

```
nano EmployeeService.js
```
> Change the variablize path

```
const EMPLOYEE_BASE_REST_API_URL = `${process.env.REACT_APP_BACKEND_URL}/api/v1/employees`;
```
### Change the RDS path in backend to access database
> Navigate to `cd springoot-backend/src/main/resources/`

```
nano application.propoerties
```
> Update this code

```
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}
 
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=${SPRING_JPA_HIBERNATE_DDL_AUTO:update}  # Default: update
```
#### Backend Image
> Create a backend immage `nano Dockerfile`

```
# Use a base image with Java and Maven installed
FROM maven:3.8.4-openjdk-17 AS build
 
# Set the working directory
WORKDIR /app
 
# Copy source code and build the application
COPY . .
 
# Optional: skip tests during build
RUN mvn clean package -DskipTests
 
# -----------------------------
# Create a new clean image just for running
FROM openjdk:17-jdk-slim
 
# Set working directory
WORKDIR /app
 
# Copy the compiled JAR file from the build stage
COPY --from=build /app/target/*.jar app.jar
 
# Optional: Set environment variables (these can be overridden at runtime)
ENV SPRING_DATASOURCE_URL=""
ENV SPRING_DATASOURCE_USERNAME=""
ENV SPRING_DATASOURCE_PASSWORD=""
ENV SPRING_JPA_HIBERNATE_DDL_AUTO=""
 
# Expose port
EXPOSE 8080
 
# Run the jar with env variables
ENTRYPOINT ["java", "-jar", "app.jar"]
```
### Database Creation
#### Login
```
mysql -h <RDS-endpoint> -P 3306 -u <username> -p
```
#### Create Database
```
create database ems;
```
### Dcokerise the image
> Create a frontend immage `nano docker-compose.yml`

```
version: '3.8'
 
services:
  frontend:
    build:
      context: ./react-hooks-frontend
      dockerfile: Dockerfile
    ports:
      - "5000:3000"
    environment:
      - REACT_APP_BACKEND_URL=http://54.165.120.121:8080
    depends_on:
      - backend
    networks:
      - ems-ops
 
  backend:
    build:
      context: ./springboot-backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://database-1.c0n8mseiazqy.us-east-1.rds.amazonaws.com:3306/ems?useSSL=false&allowPublicKeyRetrieval=true
      - SPRING_DATASOURCE_USERNAME=admin
      - SPRING_DATASOURCE_PASSWORD=admin2000
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
    networks:
      - ems-ops
 
 
networks:
  ems-ops:  # Let Docker create and manage this network
    driver: bridge
```
## Build and run the iamge
```
docker compose -f docker-compose.yml build
docker compose -f docker-compose.yml up -d
```
![image](https://github.com/user-attachments/assets/d362a1de-94b0-4cb7-8e7d-2442114b034f)

---
## 📦 Deploying EC2 Application Behind an AWS ALB

This guide walks you through setting up an **Application Load Balancer (ALB)** to route traffic to your EC2 instance running a web application (e.g., Flask, Node.js) on **port 5000**.

---

### 🛠️ Step 1: Create a Target Group

1. Go to **EC2 > Target Groups**
2. Click **Create target group**
3. Configure:
   - **Target type**: Instances
   - **Protocol**: HTTP
   - **Port**: 5000
   - **VPC**: Select the VPC where your EC2 instance is deployed
4. Click **Next**
5. In **Register targets**:
   - Select your EC2 instance
   - Click **Add to registered**
6. Click **Create target group**

---

### 🛠️ Step 2: Create an Application Load Balancer (ALB)

1. Go to **EC2 > Load Balancers**
2. Click **Create Load Balancer > Application Load Balancer**
3. Set the following:
   - **Name**: `my-alb` (or any name)
   - **Scheme**: Internet-facing
   - **IP address type**: IPv4
   - Select **2 or more Availability Zones** and corresponding subnets
4. Create or select a **Security Group** that allows:
   - **Inbound Rule**: HTTP (Port 80) from `0.0.0.0/0`
5. Under **Listeners**:
   - **Listener**: HTTP (Port 80)
   - **Forward to**: Select the **Target Group** created in Step 1
6. Click **Create Load Balancer**

---

### 🧪 Step 3: Test the Setup

1. Wait until the **Load Balancer Status** becomes **Active**
2. Ensure that the **Target Group health check** passes
3. Open your browser and go to:

```
http://ec2-alb-638808773.ap-northeast-3.elb.amazonaws.com/employees
```
![image](https://github.com/user-attachments/assets/18c6f381-8323-4f35-a1a6-24b607a5e107)

