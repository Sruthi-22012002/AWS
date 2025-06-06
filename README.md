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

