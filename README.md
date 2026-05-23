# AWS Auto Scaling Group Project

## Introduction

This project demonstrates how to design and deploy a **highly available, 
fault-tolerant web infrastructure on Amazon Web Services (AWS)** using core 
cloud networking and compute services. 

In real-world cloud environments, applications must handle unpredictable 
traffic — sometimes a hundred users, sometimes ten thousand. Manually 
spinning up servers every time traffic spikes is not scalable. This project 
solves that problem by building an infrastructure that **automatically scales 
up when demand increases and scales back down when it drops**, all while 
distributing traffic evenly so no single server gets overwhelmed.

This was built entirely through the AWS Management Console without 
Infrastructure as Code tools, making it an accessible but production-relevant 
demonstration of cloud architecture fundamentals.

---

## Architecture Overview

Internet
|
Application Load Balancer (ALB)
|              |
Public Subnet AZ-1   Public Subnet AZ-2
EC2 Instance       EC2 Instance
\               /
Auto Scaling Group
(min: 2, desired: 2, max: 4)
|
VPC (asg-project-vpc)

**Services Used:**
- Amazon VPC (networking foundation)
- EC2 (compute instances)
- Application Load Balancer (traffic distribution)
- Auto Scaling Group (automatic scaling)
- Security Groups (access control)

---

## Steps Taken

### Step 1 — Created a VPC and Network Infrastructure
- Created a custom VPC named `asg-project-vpc` using the "VPC and more" option
- Configured 2 public subnets spread across 2 Availability Zones for high availability
- AWS automatically provisioned route tables and an Internet Gateway

![Image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/1.VPC%20resource%20map%20showing%20subnets%20and%20internet%20gateway.png)
---

### Step 2 — Configured a Security Group
- Created `asg-sg` to control inbound traffic to EC2 instances
- Allowed HTTP (port 80) from anywhere so the web app is publicly accessible
- Allowed SSH (port 22) from my IP only for secure instance access

![Image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/2.Security%20Groups.png)
---

### Step 3 — Created a Launch Template
- Defined the blueprint for all instances the ASG would launch
- Used Amazon Linux 2023 (free tier), t3.micro instance type
- Added a User Data script to automatically install and start Apache HTTP Server 
  on every new instance, serving a custom webpage showing the instance hostname

[image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/3.%20Created%20Lunch%20Template.png)
---

### Step 4 — Created a Target Group
- Set up `asg-target-group` to register instances and monitor their health
- Configured health checks on port 80 to ensure only healthy instances 
  receive traffic

[image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/4.%20Created%20Target%20Groups.png)

---

### Step 5 — Deployed an Application Load Balancer
- Created an internet-facing ALB named `asg-alb`
- Attached it to both public subnets across both Availability Zones
- Configured a listener on port 80 to forward all traffic to the target group

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/5.%20Load%20Balancer%20Active%20with%20DNS%20name.png)

---

### Step 6 — Created the Auto Scaling Group
- Created `asg-project-group` using the launch template
- Attached it to both public subnets and connected it to the ALB target group
- Set capacity: Minimum 2, Desired 2, Maximum 4
- Applied a Target Tracking scaling policy set to 50% CPU utilization
  — the ASG automatically adds instances when CPU exceeds 50% and 
  removes them when load drops

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/6.1%20ASG%20created%20with%20Desired%20Capacity.png)

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/6.2.%20Scaling%20Policy.png)
---

### Step 7 — Verified Instances Launched Successfully
- Confirmed 2 EC2 instances were launched and managed by the ASG
- Checked the ASG Instance Management tab showing both instances 
  with "InService" health status

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/7.%20Created%20ASG%20EC2%20Instances.png)
---

### Step 8 — Tested the Load Balancer
- Copied the ALB DNS name and accessed it in a browser
- Successfully loaded the Apache webpage showing the instance hostname
- Used curl in terminal to confirm traffic was being distributed across 
  both instances

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/8.1.%20Hello%20World.png)

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/8.2.%20Hello%20World2.png)
---

## What I Learned

- How a VPC creates an isolated, secure network environment inside AWS
- How subnets across multiple Availability Zones enable high availability
- How Launch Templates define a repeatable, consistent blueprint for instances
- How an Application Load Balancer distributes traffic and monitors instance health
- How Auto Scaling Groups maintain desired capacity and respond to real load
- How Target Tracking policies make scaling decisions automatically based on metrics
- Why setting a minimum of 2 instances across 2 AZs is a best practice for 
  fault tolerance — if one Availability Zone goes down, the other keeps serving traffic

---

## Cleanup

All resources were deleted after the project to avoid AWS charges, 
in the following order:
1. Auto Scaling Group (automatically terminates EC2 instances)
2. Application Load Balancer
3. Target Group
4. Launch Template
5. Security Group
6. VPC

![image](https://github.com/SimonEdafe/AWS-ASG-Project/blob/c821639c14f9f8a8ba11d763f58aba8f3aa1a415/images/Clean%20up.png)