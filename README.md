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
<img width="940" height="472" alt="image" src="https://github.com/user-attachments/assets/75b79304-3880-4be9-bc64-b86a4cb94243" />

<img width="940" height="424" alt="image" src="https://github.com/user-attachments/assets/43473cc6-b151-4291-8107-f9f08b57968f" />


```bash
# Navigate to home directory
cd /home/ec2-user
```
```bash
# Download your website files
mkdir webiste

cd website

wget https://example.com/your-website.zip
```
<img width="940" height="240" alt="image" src="https://github.com/user-attachments/assets/5dfc1f6c-a3bb-42aa-8cc1-7fb2df084394" />

```bash
# Extract the files
sudo unzip your-website.zip
```
```bash 
sudo rm -r your-website.zip
```
## **Option C: Move files to Apache directory**

Once your website files are in `/home/ec2-user/website/`, move them to the Apache web root:

```bash
sudo cp -r /home/ec2-user/website/* /var/www/html/
```
## Set proper ownership and permissions:
```bash
sudo chown -R apache:apache /var/www/html
```
```bash
sudo chmod -R 755 /var/www/html
```
## **Step 6: Configure Route53 DNS**

### **6.1 Create a Hosted Zone in AWS**

1. Go to **AWS Console → Route53 → Hosted zones**  
2. Click **Create hosted zone**
3. <img width="940" height="395" alt="image" src="https://github.com/user-attachments/assets/4079c6fe-7dc2-4c72-9262-8f9939d33d1c" />

4. Enter your domain name (e.g., `devopspedia.shop`)  
5. **Type:** Public hosted zone  
6. **Description:** Add your Elastic IP for reference (optional)
   
<img width="940" height="460" alt="image" src="https://github.com/user-attachments/assets/33ea0417-59b3-45f8-8420-d9627a6da25c" />
 
7. Click **Create hosted zone**

<img width="1181" height="692" alt="asdf" src="https://github.com/user-attachments/assets/b54f787e-4629-4924-b449-716e5c1ac17a" />

### **6.2 Copy Nameservers**

- After creating the hosted zone, you will see **4 NS (nameserver) records**.  
- Copy all **4 nameservers** (they will look like `ns-xxxx.awsdns-xx.org`, etc.).

  
### **6.3 Update Nameservers in Hostinger (or your domain provider)**

1. Log in to **Hostinger**  
2. Go to **Domains → Select your domain → DNS/Nameservers**  
3. Click **Change Nameservers**  
4. Paste all **4 Route53 nameservers**  
5. **Save changes**
   
<img width="1781" height="606" alt="hostinger" src="https://github.com/user-attachments/assets/f32edf5c-5bae-43a3-93fd-068dbdd5197b" />

> 🕒 **Note:** DNS propagation usually takes **10–30 minutes**, but in some cases it may take up to **48 hours**.

### **6.4 Create A Records in Route53**

1. In **Route53 Hosted Zone** for your domain, click **Create record**  

**First Record (Root domain):**  
- **Record name:** (leave blank)  
- **Record type:** A  
- **Value:** `YOUR_ELASTIC_IP`  
- **TTL:** 300  
- Click **Create records**
  
<img width="940" height="382" alt="image" src="https://github.com/user-attachments/assets/6114cc01-f9ba-4a40-ae30-47f14bbac4ef" />


**Second Record (www subdomain):**  
- **Record name:** www  
- **Record type:** A  
- **Value:** `YOUR_ELASTIC_IP`  
- **TTL:** 300  
- Click **Create records**
  
<img width="940" height="369" alt="image" src="https://github.com/user-attachments/assets/9a4f8162-336a-498b-b5dc-8af329c79002" />

## **Step 7: Create Apache Virtual Host**

### **7.1 Create configuration file**
```bash
sudo vi /etc/httpd/conf.d/yourdomain.conf
```
### **7.2 Paste this configuration**

*(Replace `yourdomain.shop` with your actual domain)*

```apache
<VirtualHost *:80>
    ServerName yourdomain.shop
    ServerAlias www.yourdomain.shop
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/yourdomain-error.log
    CustomLog /var/log/httpd/yourdomain-access.log combined
</VirtualHost>
```
- **Save and exit:** Type `:wq` in vi and press **Enter**

### **7.3 Test and restart Apache**

Test Apache configuration for syntax errors:

```bash
sudo apachectl configtest
```
```bash
sudo systemctl restart httpd
```
## **Step 8: Verify DNS Resolution**

- Wait **10–15 minutes** after updating nameservers for propagation.  
- Then test DNS resolution using `dig`:

```bash
dig +short yourdomain.shop
```
```bash
dig +short www.yourdomain.shop
```
- Both commands should return your Elastic IP.

## **Step 9: Test HTTP**

- Test your website using `curl`:

```bash
curl -I http://yourdomain.shop
```

- The response should return **200 OK** along with Apache headers.

- You can also open your browser and navigate to:
  
Open in browser: `http://yourdomain.shop `- should show your website.

## **Step 9: Install Certbot for SSL Certificate**

### **9.1 Install snapd**
```bash
sudo yum install -y snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
```
- Wait a few seconds for snap to initialize:

```bash
sleep 10
```
### **9.2 Install Certbot**
```bash
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
## **Step 10: Obtain SSL Certificate (Enable HTTPS)**

Run Certbot to get a free SSL certificate and enable HTTPS:

```bash
sudo certbot --apache \
  -d yourdomain.shop \
  -d www.yourdomain.shop \
  -m youremail@example.com \
  --agree-tos \
  --no-eff-email \
  --redirect
```
### **What this does**

✅ Gets a free SSL certificate from **Let's Encrypt**  
✅ Configures Apache to use **HTTPS (port 443)**  
✅ Automatically redirects **HTTP → HTTPS**  
✅ Sets up **auto-renewal** (every 90 days)  

### **Expected output**

```bash
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/yourdomain.shop/fullchain.pem
Congratulations! Your certificate and chain have been saved...
```

<img width="940" height="422" alt="image" src="https://github.com/user-attachments/assets/3d0c4f9c-fb91-45d8-96fa-c4ee7576fc62" />

## **Step 11: Verify HTTPS is Working**

- Test your website using `curl` in the terminal:

```bash
curl -I https://yourdomain.shop
```

## Open in browser:
<img width="940" height="477" alt="image" src="https://github.com/user-attachments/assets/f0501867-a90f-4792-a305-b06d2cb70f8c" />

   - `https://yourdomain.shop` 🔒 - Should show green padlock
   - `http://yourdomain.shop` - Should redirect to HTTPS

## ✅ Verification Checklist

After completing all steps, verify the following:

- **EC2 instance** is running  
- **Elastic IP** is associated  
- **Security Group** allows ports **22, 80, 443**  
- **Apache** is running:  
```bash
sudo systemctl status httpd
dig +short yourdomain.shop
http://yourdomain.shop
https://yourdomain.shop 🔒
```
   - HTTP automatically redirects to HTTP
   - SSL certificate auto-renewal is enabled

## 🔧 Useful Commands

### **Apache commands**
```bash
sudo systemctl status httpd             # Check Apache status
sudo systemctl restart httpd            # Restart Apache
sudo apachectl configtest               # Test Apache configuration
sudo tail -f /var/log/httpd/error_log   # View Apache error logs
```
### **Certbot commands**
```bash
sudo certbot certificates          # View certificate details
sudo certbot renew --dry-run       # Test renewal process
sudo certbot renew                 # Manually renew SSL certificate (if needed)
```
### **DNS verification**
```bash
dig +short yourdomain.shop          # Check DNS A record
nslookup yourdomain.shop            # Alternative DNS check
curl -I https://yourdomain.shop     # Test HTTPS connection
```
## 🐛 Troubleshooting

**Issue:** Certbot fails with "Challenge failed"  

**Solution:**  

- Verify that your DNS points to your Elastic IP:

```bash
dig +short yourdomain.shop
```
- Check if website loads via HTTP first:

```bash
curl -I http://yourdomain.shop
```
   - Ensure ports **80** and **443** are open:
   - Check your **Security Group** settings in the EC2 Console
**Issue:** `"localhost.crt not found"` error  

**Solution:**  

- Backup old SSL configuration:

```bash
sudo mv /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.bak
```
```bash
sudo systemctl restart httpd
```
**Issue:** DNS not resolving  

**Solution:**  
- Wait **15–30 minutes** for DNS propagation  
- Verify nameservers are updated in **Hostinger**  
- Check **A records** exist in **Route53**  
- Optionally, use an online tool: [https://dnschecker.org](https://dnschecker.org)  

**Issue:** Website shows Apache default page  

**Solution:**  
- Remove the default Apache welcome page:

```bash
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.bak
sudo systemctl restart httpd
```

## 📚 Additional Resources

- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)  
- [Route53 Documentation](https://docs.aws.amazon.com/route53/)  
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)  
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)  
- [Certbot Documentation](https://certbot.eff.org/docs/)

---

## 🎉 What You've Accomplished

✅ Deployed a static website on **AWS EC2**  
✅ Configured custom domain with **Route53**  
✅ Secured website with free **SSL certificate**  
✅ Enabled **HTTPS** with auto-redirect  
✅ Set up automatic **certificate renewal**  
✅ Created a **production-ready website hosting setup**

---

## 📝 License

This guide is **free to use and share**. If you found it helpful, please ⭐ star this repository!

---

## 🤝 Contributing

Found an error or want to improve this guide? Feel free to:  
- Fork this repository  
- Create a new branch  
- Make your changes  
- Submit a pull request

---

## 📧 Contact

For questions or feedback, open an **issue** in this repository.  

Made with ❤️ for the **DevOps community**.




