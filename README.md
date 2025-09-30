# Step-by-step guide to host a secure static website on AWS EC2 with HTTPS and Route53# Host a Static Website Securely on AWS EC2 with HTTPS

A complete step-by-step guide to deploy a static website on AWS EC2, secure it with HTTPS (Let's Encrypt SSL), and connect it to your custom domain using Route53.

---

## 🎯 What You'll Learn

- Launch and configure an EC2 instance
- Assign an Elastic IP address
- Install and configure Apache web server
- Set up Route53 for DNS management
- Obtain a free SSL certificate with Let's Encrypt
- Enable HTTPS with automatic HTTP to HTTPS redirect
- Configure auto-renewal for SSL certificates

---

## 📋 Prerequisites

- AWS account with EC2 access
- A registered domain name (Hostinger, GoDaddy, Namecheap, etc.)
- Basic knowledge of Linux commands
- SSH client (Terminal, PuTTY, or Command Prompt)

---

### Step 1: Launch EC2 Instance

1. Go to **AWS Console** → **EC2** → **Launch Instance**
2. Configure:
   - **Name:** `my-website-server` (or any name)

     <img width="940" height="207" alt="image" src="https://github.com/user-attachments/assets/49e21163-23ae-4fe5-9845-1b0fa3fc504c" />

   - **AMI:** Amazon Linux 2 or Ubuntu Server

     <img width="940" height="514" alt="image" src="https://github.com/user-attachments/assets/1d4af712-2305-42c6-8b1d-8b4edebdeb63" />

   - **Instance Type:** t2.micro (Free tier eligible)

     <img width="940" height="243" alt="image" src="https://github.com/user-attachments/assets/56def440-c5f1-445c-a6a5-73e5932b2fbc" />

   - **Key Pair:** Create new or select existing (save the `.pem` file securely)
3. **Network Settings:**
   - Create Security Group with rules:
     - SSH (22) - Your IP only
     - HTTP (80) - 0.0.0.0/0
     - HTTPS (443) - 0.0.0.0/0

       <img width="940" height="497" alt="image" src="https://github.com/user-attachments/assets/f071079e-8c32-45d6-853f-f0215018d2d2" />


4. Click **Advance Details** -> **User Data**
   # AWS EC2 User Data Script

Use the following script in the **User Data** section while launching your EC2 instance.  
This script will update packages, install Apache (httpd), Git, and Unzip, then start and enable Apache.
## Complete Example:
## 📝 Script Code
```bash
#!/bin/bash
sudo yum update -y
sudo yum upgrade -y
sudo yum install httpd -y
sudo yum install git -y
sudo yum install unzip -y
sudo systemctl start httpd
sudo systemctl enable httpd
```
5. Click **Launch Instance**
---

### Step 2: Allocate and Associate Elastic IP

1. In EC2 Console → **Elastic IPs** → **Allocate Elastic IP address**
2. Click **Allocate**
3. Select the newly created Elastic IP → **Actions** → **Associate Elastic IP address**
4. Select your instance → Click **Associate**
5. **Note down your Elastic IP** (e.g., 54.123.45.67)

<img width="940" height="379" alt="image" src="https://github.com/user-attachments/assets/c3ee35bb-0e9b-4e1d-9b18-2503fde66caf" />

<img width="940" height="350" alt="image" src="https://github.com/user-attachments/assets/763a9226-de38-4117-81a8-b0aa5199eae8" />

---

### Step 3: Connect to EC2 via SSH

**For Windows (Command Prompt):**
```bash
cd Downloads  # Navigate to where your .pem key is saved
```
```bash
ssh -i "your-key.pem" ec2-user@YOUR_ELASTIC_IP
```
<img width="940" height="376" alt="image" src="https://github.com/user-attachments/assets/d87c7053-99f3-46db-8811-718f583b776a" />

### Step 4: Install Apache Web Server mod ssl and check the status of apache server
Once connected to EC2, run:

# Install Apache and SSL module
```bash
sudo yum install httpd mod_ssl -y
```
```bash
# Check status
sudo systemctl status httpd
```

**Test:** Open `http://YOUR_ELASTIC_IP` in browser - should see Apache test page.

<img width="940" height="477" alt="image" src="https://github.com/user-attachments/assets/407eef50-e797-4699-b3c2-099b1236d71f" />

---

# Step 5: Upload Your Website Files

There are two methods to upload your website files to the EC2 instance:

---

## **Option A: Using SCP (from your local machine)**

Transfer files directly from your computer to EC2 using SCP:
```bash
scp -i your-key.pem -r /path/to/website/* ec2-user@YOUR_ELASTIC_IP:/home/ec2-user/
```
## **Option B: Using wget (download from URL)**

If your website files are hosted online (e.g., GitHub, Dropbox):
```bash
# Navigate to home directory
cd /home/ec2-user
```
```bash
# Download your website files
wget https://example.com/your-website.zip
```
```bash
# Extract the files
unzip your-website.zip
```



