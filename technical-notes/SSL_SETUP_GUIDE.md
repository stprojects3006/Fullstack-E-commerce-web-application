# PURELY E-commerce Application - SSL Certificate Setup Guide

## 🔒 Overview

This guide provides comprehensive instructions for setting up SSL certificates for the PURELY E-commerce application. We support both self-signed certificates (for testing) and Let's Encrypt certificates (for production).

## 📋 Prerequisites

- Root access to the server
- Domain name pointing to your server (for Let's Encrypt)
- Ports 80 and 443 open in firewall
- Docker and Docker Compose installed

## 🚀 Quick Setup Options

### Option 1: Self-Signed Certificates (Testing)

For development and testing environments:

```bash
# Make scripts executable
chmod +x ssl-setup.sh
chmod +x generate-selfsigned-cert.sh

# Generate self-signed certificate
sudo ./generate-selfsigned-cert.sh 18.217.148.69

# Create SSL directory for Docker
mkdir -p ssl
sudo cp /etc/nginx/ssl/nginx-selfsigned.* ssl/

# Deploy with SSL
./deploy.sh
```

### Option 2: Let's Encrypt Certificates (Production)

For production environments with a domain name:

```bash
# Run the SSL setup script
sudo ./ssl-setup.sh yourdomain.com your-email@example.com

# Update docker-compose.yml to use Let's Encrypt certificates
# Uncomment the Let's Encrypt volume mounts in nginx service

# Deploy with SSL
./deploy.sh
```

## 🔧 Detailed Setup Instructions

### 1. Self-Signed Certificate Setup

#### Step 1: Generate Self-Signed Certificate
```bash
# Run the certificate generator
sudo ./generate-selfsigned-cert.sh 18.217.148.69
```

#### Step 2: Prepare for Docker
```bash
# Create SSL directory in project
mkdir -p ssl

# Copy certificates to project directory
sudo cp /etc/nginx/ssl/nginx-selfsigned.* ssl/

# Set proper permissions
sudo chown $USER:$USER ssl/*
chmod 600 ssl/nginx-selfsigned.key
chmod 644 ssl/nginx-selfsigned.crt
```

#### Step 3: Deploy with SSL
```bash
# Build and deploy
./build.sh
./deploy.sh
```

### 2. Let's Encrypt Certificate Setup

#### Step 1: Prepare Domain
Ensure your domain points to your server:
```bash
# Check DNS resolution
nslookup yourdomain.com
ping yourdomain.com
```

#### Step 2: Run SSL Setup Script
```bash
# Run the comprehensive SSL setup
sudo ./ssl-setup.sh yourdomain.com your-email@example.com
```

#### Step 3: Update Docker Configuration
Edit `docker-compose.yml` and uncomment Let's Encrypt volumes:
```yaml
nginx:
  volumes:
    - ./nginx-ssl.conf:/etc/nginx/nginx.conf
    - ./frontend/dist:/usr/share/nginx/html
    # SSL Certificates (uncomment when using Let's Encrypt)
    - /etc/letsencrypt:/etc/letsencrypt:ro
    - /var/www/html:/var/www/html:ro
    # Comment out self-signed certificates
    # - ./ssl:/etc/nginx/ssl:ro
```

#### Step 4: Update Nginx Configuration
Edit `nginx-ssl.conf` and uncomment Let's Encrypt certificate paths:
```nginx
# SSL Certificate Configuration
ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

# Comment out self-signed certificate paths
# ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
# ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
```

#### Step 5: Deploy
```bash
# Deploy with Let's Encrypt certificates
./deploy.sh
```

## 🔍 SSL Configuration Details

### Nginx SSL Configuration

The SSL configuration includes:

- **TLS 1.2 and 1.3** support
- **Strong cipher suites** for security
- **HTTP/2** support
- **Security headers** (HSTS, X-Frame-Options, etc.)
- **OCSP stapling** for performance
- **Automatic HTTP to HTTPS redirect**

### Security Headers

```nginx
# Security headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Frame-Options DENY always;
add_header X-Content-Type-Options nosniff always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

### CORS Configuration

SSL-enabled CORS headers:
```nginx
# CORS headers for HTTPS
add_header Access-Control-Allow-Origin "https://yourdomain.com" always;
add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
add_header Access-Control-Allow-Headers "*" always;
add_header Access-Control-Allow-Credentials "true" always;
```

## 🛠️ Management Commands

### Certificate Status
```bash
# Check self-signed certificate
sudo /usr/local/bin/cert-info.sh

# Check Let's Encrypt certificate
sudo /usr/local/bin/ssl-status.sh yourdomain.com
```

### SSL Troubleshooting
```bash
# Comprehensive SSL diagnostics
sudo /usr/local/bin/ssl-troubleshoot.sh yourdomain.com
```

### Certificate Renewal
```bash
# Manual renewal (Let's Encrypt)
sudo /usr/local/bin/renew-ssl.sh

# Check renewal logs
sudo journalctl -u certbot.timer
```

### Nginx SSL Testing
```bash
# Test SSL configuration
sudo nginx -t

# Test SSL connection
curl -k https://yourdomain.com

# Check SSL certificate
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com
```

## 🔧 Troubleshooting

### Common Issues

#### 1. Certificate Not Found
```bash
# Check certificate location
ls -la /etc/letsencrypt/live/yourdomain.com/
ls -la ssl/

# Verify nginx configuration
nginx -t
```

#### 2. SSL Connection Failed
```bash
# Check port 443 is open
sudo netstat -tulpn | grep :443

# Check firewall
sudo ufw status
sudo iptables -L
```

#### 3. Let's Encrypt ACME Challenge Failed
```bash
# Check webroot directory
ls -la /var/www/html/.well-known/acme-challenge/

# Test ACME challenge
curl http://yourdomain.com/.well-known/acme-challenge/test
```

#### 4. Certificate Expired
```bash
# Check certificate expiry
openssl x509 -in /etc/letsencrypt/live/yourdomain.com/fullchain.pem -noout -dates

# Force renewal
sudo certbot renew --force-renewal
```

### SSL Testing Tools

#### Online SSL Checkers
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [SSL Checker](https://www.sslshopper.com/ssl-checker.html)

#### Command Line Tools
```bash
# Test SSL configuration
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com

# Check certificate chain
openssl s_client -connect yourdomain.com:443 -showcerts

# Test specific cipher
openssl s_client -connect yourdomain.com:443 -cipher ECDHE-RSA-AES128-GCM-SHA256
```

## 📊 SSL Performance Optimization

### OCSP Stapling
```nginx
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/letsencrypt/live/yourdomain.com/chain.pem;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

### Session Caching
```nginx
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

### HTTP/2 Support
```nginx
listen 443 ssl http2;
```

## 🔄 Automatic Renewal

### Let's Encrypt Auto-Renewal
The setup script configures automatic renewal:
```bash
# Check renewal cron job
sudo crontab -l | grep certbot

# Test renewal process
sudo certbot renew --dry-run
```

### Renewal Monitoring
```bash
# Check renewal logs
sudo journalctl -u certbot.timer -f

# Monitor certificate expiry
sudo /usr/local/bin/ssl-status.sh yourdomain.com
```

## 🚨 Security Best Practices

### 1. Certificate Security
- Use strong private keys (2048-bit RSA or ECDSA)
- Store private keys securely with proper permissions
- Regularly rotate certificates
- Monitor certificate expiry

### 2. SSL Configuration
- Disable weak protocols (SSLv2, SSLv3, TLS 1.0, TLS 1.1)
- Use strong cipher suites
- Enable HSTS
- Implement proper CORS policies

### 3. Monitoring
- Monitor certificate expiry
- Set up alerts for failed renewals
- Regular security audits
- SSL configuration testing

## 📋 Checklist

### Pre-Setup
- [ ] Domain DNS configured
- [ ] Ports 80 and 443 open
- [ ] Root access available
- [ ] Email address for Let's Encrypt

### Self-Signed Setup
- [ ] Generated self-signed certificate
- [ ] Copied certificates to project directory
- [ ] Updated nginx configuration
- [ ] Tested SSL connection

### Let's Encrypt Setup
- [ ] Domain resolves to server
- [ ] Ran SSL setup script
- [ ] Updated docker-compose.yml
- [ ] Updated nginx configuration
- [ ] Tested certificate renewal

### Post-Setup
- [ ] SSL connection working
- [ ] HTTP to HTTPS redirect working
- [ ] CORS headers configured
- [ ] Automatic renewal configured
- [ ] Monitoring set up

## 🔗 Related Documentation

- [PORT_SETUP_GUIDE.md](./PORT_SETUP_GUIDE.md) - Port configuration
- [TROUBLESHOOTING_GUIDE.md](./TROUBLESHOOTING_GUIDE.md) - SSL troubleshooting
- [COMPREHENSIVE_DEPLOYMENT_GUIDE.md](./COMPREHENSIVE_DEPLOYMENT_GUIDE.md) - Full deployment guide

---

*For additional help, refer to the troubleshooting guide or check the SSL scripts in the project root.* 