# Node.js Application Deployment Guide

This repository contains a Node.js application with full deployment instructions for AWS EC2 using PM2, NGINX, and Let's Encrypt SSL.

## Table of Contents
- [Local Development](#local-development)
- [AWS EC2 Deployment](#aws-ec2-deployment)
  - [1. Launch EC2 Instance](#1-launch-ec2-instance)
  - [2. SSH Into Your Instance](#2-ssh-into-your-instance)
  - [3. Install Node.js](#3-install-nodejs)
  - [4. Clone and Setup the Application](#4-clone-and-setup-the-application)
  - [5. Install and Configure PM2](#5-install-and-configure-pm2)
  - [6. Setup NGINX as Reverse Proxy](#6-setup-nginx-as-reverse-proxy)
  - [7. Configure Firewall](#7-configure-firewall)
  - [8. Secure with Let's Encrypt SSL](#8-secure-with-lets-encrypt-ssl)
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)

## Local Development

### Prerequisites
- Node.js (v18.x recommended)
- npm

### Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/your-project.git
   cd your-project
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```

4. The application will be available at http://localhost:8001

## AWS EC2 Deployment

### 1. Launch EC2 Instance

1. Sign up or log in to [AWS](https://aws.amazon.com/)
2. Navigate to EC2 Dashboard and click "Launch Instance"
3. Choose Ubuntu Server 22.04 LTS
4. Select t2.micro instance type (Free Tier eligible)
5. Configure Security Group to allow:
   - SSH (Port 22)
   - HTTP (Port 80)
   - HTTPS (Port 443)
6. Create or select an existing key pair
7. Launch the instance

### 2. SSH Into Your Instance

```bash
ssh -i path/to/your-key.pem ubuntu@your_ec2_public_ip
```

### 3. Install Node.js

```bash
# Update package lists
sudo apt update

# Install Node.js 18.x and npm
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version
npm --version
```

### 4. Clone and Setup the Application

```bash
# Clone the repository
git clone https://github.com/yourusername/your-project.git
cd your-project

# Install dependencies
npm install

# Set up environment variables (if needed)
nano ~/.bashrc
# Add your environment variables at the end of the file:
# export DB_URI=your_database_uri
# export SECRET_KEY=your_secret_key
# etc.

source ~/.bashrc
```

### 5. Install and Configure PM2

```bash
# Install PM2 globally
sudo npm install -g pm2

# Start your application with PM2
pm2 start index.js --name "your-app-name"

# Alternatively, create and use an ecosystem file
pm2 ecosystem
# Edit the ecosystem.config.js file:
# module.exports = {
#   apps: [{
#     name: "your-app-name",
#     script: "index.js",
#     env: {
#       NODE_ENV: "production",
#       PORT: 8001
#     }
#   }]
# }

# Then start using:
pm2 start ecosystem.config.js

# Set PM2 to start on system boot
pm2 startup
# Run the command provided by PM2

# Save the current process list
pm2 save
```

### 6. Setup NGINX as Reverse Proxy

```bash
# Install NGINX
sudo apt install -y nginx

# Configure NGINX
sudo nano /etc/nginx/sites-available/default
```

Replace the content with:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    location / {
        proxy_pass http://localhost:8001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Verify the configuration
sudo nginx -t

# Reload NGINX
sudo systemctl reload nginx
```

### 7. Configure Firewall

```bash
# Allow SSH, HTTP, and HTTPS
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'

# Enable the firewall
sudo ufw enable

# Check status
sudo ufw status
```

### 8. Secure with Let's Encrypt SSL

Ensure your domain's DNS records point to your EC2 IP address before proceeding.

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain and configure SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Follow the prompts

# Test automatic renewal
sudo certbot renew --dry-run
```

## Maintenance

### Updating the Application

```bash
# Navigate to your application directory
cd /path/to/your-project

# Pull the latest changes
git pull

# Install any new dependencies
npm install

# Restart the application
pm2 restart your-app-name
```

### Monitoring

```bash
# View logs
pm2 logs

# Monitor processes
pm2 monit

# List running processes
pm2 status
```

### System Updates

```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade
```

## Troubleshooting

### Common Issues

1. **Application not accessible**:
   - Check if application is running: `pm2 status`
   - Verify NGINX configuration: `sudo nginx -t`
   - Check firewall status: `sudo ufw status`

2. **PM2 issues**:
   - Restart PM2: `pm2 restart all`
   - View detailed logs: `pm2 logs --lines 200`

3. **SSL Certificate Problems**:
   - Check certificate status: `sudo certbot certificates`
   - Renew certificates: `sudo certbot renew`

---

For more assistance, please open an issue in this repository.
