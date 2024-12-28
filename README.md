# Static Site Server Setup Documentation

This document outlines the steps to set up a basic Linux server to serve a static site using Nginx. It also demonstrates how to deploy updates using a simple script with rsync.

---

## Project Overview
The goal of this project is to set up a web server capable of serving a static website using Nginx. The static site will be hosted on a remote Linux server, and deployment will be automated using a local script.

---

## Requirements

1. **Linux Server:** Create and configure a remote Linux server using a cloud provider (e.g., AWS EC2, DigitalOcean, etc.).
2. **SSH:** Set up SSH access to the server.
3. **Nginx:** Install and configure Nginx to serve a static site.
4. **Static Site:** Create a basic static site with HTML and CSS.
5. **Rsync:** Use rsync to deploy changes to the server.
6. **Domain Name:** Optionally, configure a custom domain to point to the server.

---

## Step-by-Step Guide

### 1. Create an EC2 Instance
- Launch an EC2 instance on AWS.
- Assign an Elastic IP to the instance for a fixed public IP address.

### 2. Connect to the Server via SSH
```bash
ssh -i <pem-file-path> <user>@<Elastic-IP>
```

### 3. Install Nginx
- Update package list and install Nginx:
  ```bash
  sudo apt update
  sudo apt install nginx -y
  ```
- Start and enable Nginx:
  ```bash
  sudo systemctl start nginx
  sudo systemctl enable nginx
  ```

### 4. Configure Nginx
1. **Edit Nginx Configuration File:**
   Create a new configuration file for your domain:
   ```bash
   sudo nano /etc/nginx/sites-available/janisadhikari.com.np
   ```
   Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name janisadhikari.com.np;

       root /var/www/html;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }

       access_log /var/log/nginx/janisadhikari_access.log;
       error_log /var/log/nginx/janisadhikari_error.log;

       gzip on;
       gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
   }

   server {
       listen 80;
       server_name www.janisadhikari.com.np;
       return 301 http://janisadhikari.com.np$request_uri;
   }
   ```

2. **Enable the Configuration:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/janisadhikari.com.np /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

3. **Set Permissions for the Web Root Directory:**
   ```bash
   sudo mkdir -p /var/www/html
   sudo chown -R ubuntu:ubuntu /var/www/html
   sudo chmod -R 755 /var/www/html
   ```

### 5. Create a Static Website
- Create a directory on your local machine with the static files (e.g., `index.html`):
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Test Run</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f4f4f9;
            color: #333;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        h1 {
            font-size: 2.5rem;
            margin: 0;
        }
        p {
            font-size: 1.2rem;
            margin: 10px 0 0;
        }
    </style>
</head>
<body>
    <div>
        <h1>This is a Test Run</h1>
        <p>by Janis Adhikari</p>
        <li style="color:green">test 1 ✔️</li>
    </div>
</body>
</html>
```

### 6. Deploy Using rsync

1. **Create a Deployment Script:**
   Save the following script as `deploy.sh` on your local machine:
   ```bash
   LOCAL_DIR="local dir path"
   REMOTE_USER="ubuntu"
   REMOTE_HOST="public -ip -address -of -instance"
   REMOTE_DIR="/var/www/html"
   SSH_KEY="path -to -pem file "

   # Function to run a command with error checking
   run_command() {
       if ! "$@"; then
           echo "Error: Command failed: $*"
           exit 1
       fi
   }

   # Ensure the remote directory exists and has correct permissions
   echo "Setting up remote directory..."
   run_command ssh -i "$SSH_KEY" "$REMOTE_USER@$REMOTE_HOST" "sudo mkdir -p $REMOTE_DIR && sudo chown -R $REMOTE_USER:$REMOTE_USER $REMOTE_DIR && sudo chmod -R 755 $REMOTE_DIR"

   # Sync files
   echo "Syncing files..."
   run_command rsync -avz --chmod=D755,F644 -e "ssh -i $SSH_KEY" "$LOCAL_DIR" "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR"

   # Set correct permissions after sync
   echo "Setting final permissions..."
   run_command ssh -i "$SSH_KEY" "$REMOTE_USER@$REMOTE_HOST" "sudo chown -R $REMOTE_USER:$REMOTE_USER $REMOTE_DIR && sudo chmod -R 755 $REMOTE_DIR"

   echo "Deployment completed successfully!"
   ```

2. **Run the Script:**
   ```bash
   bash deploy.sh
   ```

### 7. Configure a Domain (Optional)
- Update your domain’s DNS settings to point to the Elastic IP of the EC2 instance.
- Verify by accessing your site using the domain name.

---

This project is part of [Janis Adhikari's](https://roadmap.sh/projects/static-site-server)  DevOps projects.