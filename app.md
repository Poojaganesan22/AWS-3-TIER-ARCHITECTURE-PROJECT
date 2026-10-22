# Flask Application Setup on Ubuntu EC2

## Project Overview

This guide explains how to set up and run a Python Flask application on an Ubuntu EC2 instance.

The Flask application runs on the private Application Tier and connects to Amazon RDS MySQL to store feedback submitted by users.

## Step 1: Update Ubuntu

Update the system packages:

```bash
sudo apt update
```

## Step 2: Install Python Packages

Install Python, pip, and virtual environment support:

```bash
sudo apt install python3-pip python3-venv python3-full -y
```

Check the Python version:

```bash
python3 --version
```

## Step 3: Create the Application Folder

Create a folder for the Flask application:

```bash
mkdir -p ~/myapp
cd ~/myapp
```

## Step 4: Create a Virtual Environment

Create a virtual environment:

```bash
python3 -m venv venv
```

Activate it:

```bash
source venv/bin/activate
```

## Step 5: Install Required Packages

Install Flask, PyMySQL, and Gunicorn:

```bash
pip install Flask PyMySQL gunicorn
```

These packages are used to run the web application and connect to the MySQL database.

## Step 6: Install the MySQL Client

Install the MySQL client:

```bash
sudo apt install default-mysql-client -y
```

Verify the installation:

```bash
mysql --version
```

## Step 7: Create the Flask Application

Create the Python file:

```bash
nano app.py
```

Add the Flask application code to this file and save it.

The application should:

* Display a message when the application is running.
* Accept feedback through the `/submit` API endpoint.
* Connect to Amazon RDS MySQL.
* Store feedback in the database.

Configure the database connection using environment variables. Do not store database passwords directly in the source code.

## Step 8: Configure Database Environment Variables

Set the database connection details:

```bash
export DB_HOST="YOUR-RDS-ENDPOINT"
export DB_USER="admin"
export DB_PASSWORD="YOUR-DB-PASSWORD"
export DB_NAME="feedbackdb1"
```

Replace the example values with your actual RDS configuration.

Keep your database password private. Do not commit it to GitHub.

## Step 9: Run the Flask Application

Ensure the virtual environment is active.

Start the application:

```bash
python app.py
```

The application should listen on port 5000.

Keep the terminal session running while testing.

## Step 10: Test the Application

Open another terminal session on the Application EC2 instance and run:

```bash
curl http://localhost:5000/
```

Expected response for the example Flask application:

```json
{
  "message": "Flask application is running"
}
```

If the application returns a response, the local Flask service is running.

## Step 11: Check the Application Port

Run:

```bash
sudo ss -lntp | grep 5000
```

This command checks whether a process is listening on port 5000.

## Step 12: Verify Connectivity

Check that the Web Tier can reach the Application Tier through the Internal Application Load Balancer.

Make sure the relevant security groups allow the required traffic.

The Application Tier should accept traffic only from the intended sources, such as the Internal ALB.

## Important Notes

* Run the Flask application inside the virtual environment.
* Use the correct Amazon RDS endpoint and database name.
* Ensure the RDS security group allows MySQL traffic from the intended application source.
* Ensure the Application Load Balancer target group uses the correct port and health check path.
* For production deployments, use Gunicorn with an appropriate service manager instead of relying on a manually started Flask development server.

## Request Flow

Browser → Public ALB → Web EC2 (Nginx) → Internal ALB → Application EC2 (Flask) → Amazon RDS MySQL
