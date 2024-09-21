# Problem Statement 1:
# Title: Containerisation and Deployment of Wisecow Application on Kubernetes


#Create a Dockerfile in the root of your repository:
# Use an official Node.js runtime as a parent image
FROM node:14

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of your application code
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Command to run your application
CMD ["npm", "start"]

#Create a directory named k8s and add the following YAML files.
#Deployment (k8s/deployment.yaml):

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow-container
        image: <your-docker-image>:<tag>
        ports:
        - containerPort: 3000

#Service (k8s/service.yaml):

apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: wisecow


#Create a directory .github/workflows and add a workflow file named ci-cd.yml:
#CI/CD Workflow (.github/workflows/ci-cd.yml):


name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and Push Docker Image
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: <your-docker-image>:latest

    - name: Set up kubectl
      uses: azure/setup-kubectl@v1
      with:
        version: 'latest'

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/deployment.yaml
        kubectl apply -f k8s/service.yaml


#TLS Implementation

server {
    listen 80;
    server_name your_domain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your_domain.com;

    ssl_certificate /etc/ssl/certs/your_cert.crt;
    ssl_certificate_key /etc/ssl/private/your_key.key;

    location / {
        proxy_pass http://wisecow-service:3000;  # Point to your service
    }
}

#Repository Structure
#Ensure your repository contains:

#The application source code
#Dockerfile
#k8s/deployment.yaml
#k8s/service.yaml
#
.github/workflows/ci-cd.yml

#Access Control
#Make sure your GitHub repository is set to public for review.

#Secrets Configuration
#In your GitHub repository settings, add the following secrets:

#DOCKER_USERNAME
#DOCKER_PASSWORD



#Problem Statement 2:
#Please choose any two objectives from the list below and attempt to achieve them using either Bash or Python.

#System Health Monitoring Script
import psutil
import logging
import datetime

# Set up logging
logging.basicConfig(filename='system_health.log', level=logging.INFO)

# Thresholds
CPU_THRESHOLD = 80
MEMORY_THRESHOLD = 80
DISK_THRESHOLD = 90

def log_health_status():
    # Get metrics
    cpu_usage = psutil.cpu_percent()
    memory_usage = psutil.virtual_memory().percent
    disk_usage = psutil.disk_usage('/').percent

    # Log current time
    current_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    # Check CPU usage
    if cpu_usage > CPU_THRESHOLD:
        logging.warning(f"{current_time} - CPU usage is above threshold: {cpu_usage}%")

    # Check memory usage
    if memory_usage > MEMORY_THRESHOLD:
        logging.warning(f"{current_time} - Memory usage is above threshold: {memory_usage}%")

    # Check disk usage
    if disk_usage > DISK_THRESHOLD:
        logging.warning(f"{current_time} - Disk usage is above threshold: {disk_usage}%")

    # Log current status
    logging.info(f"{current_time} - CPU: {cpu_usage}%, Memory: {memory_usage}%, Disk: {disk_usage}%")

if __name__ == "__main__":
    log_health_status()


#Application Health Checker

import requests

def check_application_health(url):
    try:
        response = requests.get(url)
        if response.status_code == 200:
            print(f"The application at {url} is UP. Status code: {response.status_code}")
        else:
            print(f"The application at {url} is DOWN. Status code: {response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"Error checking application health: {e}")

if __name__ == "__main__":
    url_to_check = "http://your_application_url_here"  # Replace with your application URL
    check_application_health(url_to_check)

