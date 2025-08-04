Deploy Flask + MySQL App via EC2, Docker, and Jenkins CI/CD
🎯 Objective:
You are required to set up a Flask application with a MySQL database, first running on an EC2 instance, then Dockerizing it, and finally deploying everything using a Jenkins pipeline. This represents a real-world production-ready DevOps deployment workflow.

✅ Part 1: Run Flask App with MySQL on EC2 Server (Manual Setup)
📌 Tasks:
Install MySQL on EC2 and configure it to allow remote connections.

Create Database and Table:

Database: mydb

Table: users

Install Python packages: flask, mysql-connector-python, werkzeug

Start the Flask application and ensure it connects to the MySQL database successfully.

Access your app via http://<ec2-ip>:5000

✅ Part 2: Dockerize the Application
📌 Tasks:
Create a Docker network called my-network.

Create a MySQL Docker container using the mysql:8.0 image.

Update app.py to connect to the MySQL container by hostname (mysql_container).

Write a Dockerfile for the Flask app.

Build and run the Flask container inside the same Docker network.

Confirm both services run and communicate correctly.

Test at: http://<ec2-ip>:5000

✅ Part 3: Use Docker Compose (Optional but Recommended)
📌 Tasks:
Create a docker-compose.yml file for both services.

Run the app using:

Copy
Edit
docker-compose up -d
Stop it using:

Copy
Edit
docker-compose down
✅ Part 4: CI/CD Deployment with Jenkins (Important ✅)
📌 New Task:
⚙️ Automate the Full Deployment via Jenkins Pipeline

❗ Add this important line in your assignment instructions:

🛠️ “You must also deploy the entire Flask + MySQL application stack using a Jenkins pipeline. This includes cloning the source code, building Docker images, pushing them to DockerHub, and running the containers on your EC2 instance.”

📌 Jenkins Pipeline Requirements:
Pull source code from GitHub.

Build the Flask Docker image using a Dockerfile.

Use Jenkins credentials to push the image to DockerHub.

Deploy the application using either:

Docker CLI (docker run)

or Docker Compose (docker-compose up)

Jenkinsfile example should include stages:

Checkout

Docker Build

Docker Push

Docker Run / Compose Up








# Part -1. Run in Local Server or EC2
## Install MySQL on the EC2 Instance
```
sudo apt update
sudo apt install mysql-server -y
```

## Configure MySQL
- Start the MySQL service
```
sudo systemctl start mysql
sudo systemctl enable mysql
```
- Set the MySQL root password:
```
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
```
- Create the database, user and table:
```
CREATE DATABASE mydb;
USE mydb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);

CREATE USER 'root'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'root'@'%';
FLUSH PRIVILEGES;

```
## Allow Remote Connections to MySQL
- Edit the MySQL configuration file:
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
- Change it to:
```
bind-address = 0.0.0.0
```
- Restart MySQL:
```
sudo systemctl restart mysql
```



## Run Your Flask App
- Install the required Python packages:
```
pip install flask mysql-connector-python werkzeug

```
- Start the Flask application:
```
python3 app.py
```
---

# Part-2. Dockerize
## Step 1: Create a Docker Network
- First, create a custom Docker network to allow communication between the MySQL and Flask containers.
```
docker network create my-network
```
## Step 2: Create and Run the MySQL Container
```
docker run -d \
  --name mysql_container \
  --network my-network \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=mydb \
  -p 3306:3306 \
  mysql:8.0
```
### Verify MySQL Setup
- Check if the MySQL container is running:
```
docker ps -a
```
- Enter the MySQL container to check if the database was created:
```
docker exec -it mysql_container mysql -u root -p
```
- Use `password` when prompted and run:
```
SHOW DATABASES;
USE mydb;
```
## Step 3: Update Flask App's `app.py`
```
db = mysql.connector.connect(
    host="mysql_container",  # MySQL container's name
    user="root",
    password="password",
    database="mydb"
)
```
- Create Table is not present
```
CREATE DATABASE mydb;

USE mydb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);

CREATE USER 'root'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'root'@'%';
FLUSH PRIVILEGES;

```
## Step 4: Create Dockerfile for Flask App
```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python3","app.py"]
```
## Step 5: Build and Run the Flask App Container
- Build the Flask app image:
```
docker build -t flask-app .
```
- Run the Flask app container connected to the same Docker network:
```
docker run -d \
  --name flask_app_container \
  --network my-network \
  -p 5000:5000 \
  flask-app
```
## Step 6: Test the Application
```
http://<your-ec2-public-ip>:5000
```
# Run Using Docker-Compose File
- Install Docker Compose
```
apt install docker-compose
```
- Run the Application
```
docker-compose up -d
```
- Down the Container
```
docker-compose down 
```
