# Configure Nginx Reverse Proxy

## Project Overview

This project uses Nginx on the Web Tier EC2 instance to forward API requests to the Flask application through an Internal Application Load Balancer.

## Architecture Flow

Browser → Public ALB → Web EC2 (Nginx) → Internal ALB → Flask Application → Amazon RDS MySQL

## Step 1: Open the Web Server

Connect to your Web Tier EC2 instance.

Check whether Nginx is installed:

```bash
nginx -v
```

## Step 2: Configure Nginx

Open the Nginx configuration file:

```bash
sudo nano /etc/nginx/nginx.conf
```

Inside the appropriate `server` block, configure the website and API reverse proxy.

Example:

```nginx
location / {
    root /usr/share/nginx/html;
    index index.html;
}

location /api/ {
    proxy_pass http://YOUR-INTERNAL-ALB-DNS/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

Replace `YOUR-INTERNAL-ALB-DNS` with your actual Internal ALB DNS name.

This configuration forwards requests to `/api/submit` to the Flask application's `/submit` endpoint.

**Important:** Add these `location` blocks inside the existing `server` block. Do not remove other required Nginx configuration.

## Step 3: Test Nginx Configuration

Run:

```bash
sudo nginx -t
```

If the configuration is correct, the output should indicate that the syntax is OK and the test is successful.

## Step 4: Restart Nginx

Run:

```bash
sudo systemctl restart nginx
```

Check the service:

```bash
sudo systemctl status nginx
```

## Step 5: Test the Application

Open your Public ALB DNS in a browser:

```text
http://YOUR-PUBLIC-ALB-DNS
```

Check the following:

* The website opens.
* API requests reach the Flask application.
* Feedback is submitted successfully.
* Feedback records are stored in Amazon RDS MySQL.

## Troubleshooting

Check Nginx configuration:

```bash
sudo nginx -t
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

View Nginx error logs:

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

Check whether the Flask application is listening on port 5000:

```bash
sudo ss -lntp | grep 5000
```

## Security Notes

* Keep the RDS database in private subnets.
* Allow only required traffic between security groups.
* Do not store database passwords or AWS credentials in GitHub.
