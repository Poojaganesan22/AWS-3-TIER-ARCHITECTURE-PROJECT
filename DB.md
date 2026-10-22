# Configure Amazon RDS MySQL for Flask Application

This guide explains how to configure Amazon RDS MySQL and connect it to the Flask application running on a private EC2 instance.

## Step 1: Connect to Amazon RDS

Connect to your private Application EC2 instance using Session Manager.

Run this command, replacing `<YOUR-RDS-ENDPOINT>` with your actual RDS endpoint:

```bash
mysql -h <YOUR-RDS-ENDPOINT> -u admin -p
```

Enter your RDS password when prompted.

If the MySQL client is not installed on Ubuntu, install it first:

```bash
sudo apt update
sudo apt install mysql-client -y
```

## Step 2: Create the Database

Inside the MySQL shell, create the database:

```sql
CREATE DATABASE IF NOT EXISTS feedbackdb1;
```

Select the database:

```sql
USE feedbackdb1;
```

> If your existing Flask application uses a different database name, use that same name consistently in both MySQL and Flask.

## Step 3: Create the Feedback Table

Run this SQL query:

```sql
CREATE TABLE IF NOT EXISTS feedback (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    feedback TEXT NOT NULL,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Step 4: Verify the Database and Table

Show the available tables:

```sql
SHOW TABLES;
```

Check the table structure:

```sql
DESCRIBE feedback;
```

Exit the MySQL shell:

```sql
EXIT;
```

## Step 5: Configure the Flask Application

Open your Flask application file. Use the actual path where you saved `app.py`.

For example:

```bash
nano ~/myapp/app.py
```

Configure the database connection using environment variables instead of writing your database password directly into the Python file.

Your Flask application should read the database settings like this:

```python
import os
import pymysql

DB_CONFIG = {
    "host": os.environ["DB_HOST"],
    "user": os.environ.get("DB_USER", "admin"),
    "password": os.environ["DB_PASSWORD"],
    "database": os.environ.get("DB_NAME", "feedbackdb1"),
    "cursorclass": pymysql.cursors.DictCursor
}
```

If your existing application already has `DB_CONFIG`, update the existing configuration rather than creating a duplicate.

Save the file and exit the editor.

## Step 6: Set the Database Environment Variables

On your private Application EC2 instance, set these variables:

```bash
export DB_HOST='<YOUR-RDS-ENDPOINT>'
export DB_USER='admin'
export DB_NAME='feedbackdb1'
read -s -p "Enter RDS password: " DB_PASSWORD
echo
export DB_PASSWORD
```

Replace `<YOUR-RDS-ENDPOINT>` with your actual RDS endpoint.

The password will not be displayed while you type it.

These environment variables apply to the current terminal session. If you start Flask from another session, configure the variables in that session too.

## Step 7: Start the Flask Application

Activate your existing virtual environment. For example:

```bash
cd ~/myapp
source venv/bin/activate
```

Install the required packages if necessary:

```bash
pip install Flask PyMySQL gunicorn
```

Start Flask:

```bash
python app.py
```

Keep this terminal session running.

Open another terminal session to test the application, or use Session Manager to open another session to the same EC2 instance.

## Step 8: Test the Flask Application

Check the application's health endpoint:

```bash
curl -i http://localhost:5000/
```

The expected response depends on the route defined in your `app.py`.

Now submit test feedback:

```bash
curl -i -X POST http://localhost:5000/submit \
-H "Content-Type: application/json" \
-d '{
  "name": "Test User",
  "email": "test@example.com",
  "feedback": "Testing the RDS database connection"
}'
```

If the application and database are configured correctly, the API should return a successful response.

For example:

```json
{
  "message": "Feedback submitted successfully!"
}
```

Your actual response depends on the code in `app.py`.

## Step 9: Verify Feedback in Amazon RDS

Connect to the database again:

```bash
mysql -h <YOUR-RDS-ENDPOINT> -u admin -p
```

Run:

```sql
USE feedbackdb1;

SELECT * FROM feedback;
```

If the test feedback appears, the Flask application has successfully stored data in Amazon RDS.

## Step 10: Check Security Group Connectivity

If the database connection fails, verify the following:

* The RDS instance is available.
* The RDS endpoint is correct.
* The Application EC2 instance can reach the RDS instance.
* The RDS security group's inbound rules allow MySQL traffic on port `3306` from the Application EC2 security group.
* The Application EC2 security group allows the required outbound traffic.
* The database name, username, and password are correct.

Avoid opening MySQL port `3306` to `0.0.0.0/0`.

## Architecture Flow

```text
Web Browser
    |
    v
Public Application Load Balancer
    |
    v
Web EC2 — Nginx
    |
    v
Internal Application Load Balancer
    |
    v
Private Application EC2 — Flask
    |
    | MySQL connection (port 3306)
    v
Amazon RDS — MySQL
    |
    v
feedbackdb1.feedback
```

## Project Verification Checklist

* [ ] The RDS instance is available.
* [ ] The `feedbackdb1` database exists.
* [ ] The `feedback` table exists.
* [ ] The Flask application starts successfully.
* [ ] The `/submit` endpoint accepts feedback.
* [ ] Test feedback appears in the RDS table.
* [ ] Database credentials are not committed to GitHub.
