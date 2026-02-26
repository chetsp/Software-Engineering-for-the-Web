# SWE-645: CampusConnect Deployment
* **Student:** Chetana Patel
* **Course:** SWE-645
* **Assignment:** Assignment 1 - S3 and EC2 Deployment
* **Git Hub link:** https://github.com/chetsp/SWE-645-Spring-2026-
---

## 1. Project URLs
* **S3 Static Website:** [View Site](http://645-1-my-campus-connect-site.s3-website.us-east-2.amazonaws.com)
* **EC2 Web Application:** [View App](http://3.23.87.66)


## 2. Project Overview
This project consists of a responsive website for CampusConnect featuring:
- **Homepage (index.html):** Landing page with campus information, images, and navigation
- **Survey Form (survey.html):** Comprehensive student feedback form with validation
- **Error Page (error.html):** Custom 404 page 

The website is deployed on both AWS S3 (static hosting) and AWS EC2 (Apache web server).

## 3. Source Files

| File Name | Purpose | Key Features |
| :--- | :--- | :--- |
| `index.html` | Main landing page | Bootstrap template, campus image, navigation, responsive design |
| `survey.html` | Student survey form | Required fields, checkboxes, radio buttons, dropdown, raffle validation |
| `error.html` | Custom 404 error page | User-friendly error message, navigation back to home |
| `campus.jpg` | Campus image asset |  campus photo|


## 4. S3 Deployment Steps

### Step 1: Create S3 Bucket
```bash
1. Login to AWS Console (https://console.aws.amazon.com)
2. Navigate to S3 service
3. Click "Create bucket"
4. Configuration:
   - Bucket name: 645-1-my-campus-connect-site
   - Region: us-east-2 (default)
   - Object Ownership: ACLs disabled 
   - Block Public Access: UNCHECK "Block all public access"
   - Acknowledge the warning about public access
5. Leave other settings as default
6. Click "Create bucket"
```

### Step 2: Upload Files
```bash
1. Click on the bucket name: 645-1-my-campus-connect-site
2. Click "Upload" button
3. Click "Add files"
4. Select all files:
   - index.html
   - survey.html
   - error.html
   - campus.jpg
5. Click "Upload"
6. Wait for upload completion (green status bar)
7. Click "Close" when done
```

### Step 3: Enable Static Website Hosting
```bash
1. In the bucket, click the "Properties" tab
2. Scroll down to "Static website hosting" section
3. Click "Edit"
4. Select "Enable"
5. Hosting type: Host a static website
6. Index document: index.html
7. Error document: error.html
8. Click "Save changes"
9. Note the Bucket website endpoint URL:
   http://645-1-my-campus-connect-site.s3-website.us-east-2.amazonaws.com
```

### Step 4: Set Bucket Policy for Public Access
```bash
1. Go to the "Permissions" tab
2. Scroll to "Bucket policy" section
3. Click "Edit"
4. Paste the following JSON policy:
```

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::645-1-my-campus-connect-site/*"
        }
    ]
}
```

```bash
5. Click "Save changes"
6. You should see a "Publicly accessible" warning - this is expected
```

### Step 5: Verify S3 Deployment
```bash
1. Go back to Properties → Static website hosting
2. Click on the Bucket website endpoint URL
3. Verify:
   Homepage loads correctly
   Campus image displays
   Navigation to survey works
   Survey form is functional
   Test 404 by accessing: [endpoint]/nonexistent.html
```

---

## 5. EC2 Deployment Steps

### Step 1: Launch EC2 Instance
```bash
1. Navigate to EC2 Dashboard in AWS Console
2. Click "Launch Instance"
3. Configure instance:
   - Name: CampusConnect-WebServer
   - AMI: Ubuntu Server 24.04 LTS (Free tier eligible)
   - Architecture: 64-bit (x86)
   - Instance type: t3.micro (Free tier eligible)
   - Key pair: Proceed without a key pair (not required)
   
4. Network settings - Click "Edit":
   - Auto-assign public IP: Enable
   - Firewall (Security Groups): Create new security group
     * Security group name: campusconnect-sg
     * Description: Allow HTTP and SSH
     * Inbound rules:
       - Rule 1: Type: SSH, Port: 22, Source: Anywhere
       - Rule 2: Type: HTTP, Port: 80, Source: Anywhere (0.0.0.0/0)
   
5. Configure storage:
   - Size: 8 GB (default)
   
6. Click "Launch instance"
7. Wait for instance state to show "Running"
8. Note the Public IPv4 address: 3.23.87.66
```

### Step 2: Connect to Instance
```bash
# Use EC2 Instance Connect (No SSH key required)
1. Go to EC2 Dashboard -> Instances
2. Select your instance (CampusConnect-WebServer)
3. Click "Connect" button at the top
4. Choose "EC2 Instance Connect" tab
5. Username: ubuntu (default for Ubuntu AMI)
6. Click "Connect"
7. A browser-based terminal will open

```

### Step 3: Install and Configure Apache Web Server
```bash
# Once connected via EC2 Instance Connect, run these commands:

# Update system packages
sudo apt update -y
sudo apt upgrade -y

# Install Apache2 web server
sudo apt install apache2 -y

# Start Apache service
sudo systemctl start apache2

# Enable Apache to start on boot
sudo systemctl enable apache2

# Verify Apache is running
sudo systemctl status apache2
# (Press 'q' to exit the status view)

# Check Apache is responding
curl http://localhost
# Should show default Apache page HTML
```

### Step 4: Deploy Website Files
```bash
# Remove default Apache welcome page
sudo rm /var/www/html/index.html

# Download campus image from S3 bucket
sudo wget -O /var/www/html/campus.jpg \
  http://645-1-my-campus-connect-site.s3-website.us-east-2.amazonaws.com/campus.jpg

# Create HTML files:

# Download directly from S3 
sudo wget -O /var/www/html/index.html \
  http://645-1-my-campus-connect-site.s3-website.us-east-2.amazonaws.com/index.html

sudo wget -O /var/www/html/survey.html \
  http://645-1-my-campus-connect-site.s3-website.us-east-2.amazonaws.com/survey.html

sudo wget -O /var/www/html/error.html \
  http://645-1-my-campus-connect-site.s3-website.us-east-2.amazonaws.com/error.html

# Set proper file permissions
sudo chown -R www-data:www-data /var/www/html
sudo chmod 644 /var/www/html/*.html
sudo chmod 644 /var/www/html/campus.jpg

# Verify files are in place
ls -la /var/www/html/
# Should show: index.html, survey.html, error.html, campus.jpg
```

### Step 5: Configure Custom Error Page
```bash
# Edit Apache virtual host configuration
sudo nano /etc/apache2/sites-available/000-default.conf

# Add this line inside the  block (before ):
ErrorDocument 404 /error.html

# Your configuration should look like:
# 
#     ServerAdmin webmaster@localhost
#     DocumentRoot /var/www/html
#     ErrorDocument 404 /error.html
#     ...
# 

# Save and exit: Ctrl+X, then Y, then Enter

# Test Apache configuration for syntax errors
sudo apache2ctl configtest
# Should output: "Syntax OK"

# Restart Apache to apply changes
sudo systemctl restart apache2

# Verify Apache restarted successfully
sudo systemctl status apache2
```

### Step 6: Verify EC2 Deployment
```bash

# Test from your browser:
# http://3.23.87.66              -> Should show homepage
# http://3.23.87.66/survey.html  -> Should show survey form
# http://3.23.87.66/test.html    -> Should show custom error page
```

---

