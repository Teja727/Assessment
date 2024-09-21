# Problem Statement 1:
# Title: Containerisation and Deployment of Wisecow Application on Kubernetes


#Create a Dockerfile in the root of your repository to containerize the Wisecow application.

# Use the official Python image from the Docker Hub
FROM python:3.9-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Set work directory
WORKDIR /usr/src/app

# Install dependencies
COPY app/requirements.txt .
RUN pip install --upgrade pip
RUN pip install -r requirements.txt

# Copy project
COPY app/ .

# Expose the port the app runs on
EXPOSE 5000

# Define the default command to run the application
CMD ["python", "app.py"]


#Create a k8s directory to store all Kubernetes manifest files.
#a. Deployment Manifest (k8s/deployment.yaml)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
  labels:
    app: wisecow
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
        image: <your-dockerhub-username>/wisecow:latest
        ports:
        - containerPort: 5000
        env:
        - name: ENVIRONMENT
          value: "production"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"


#Service Manifest (k8s/service.yaml)

apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: ClusterIP


#Ingress Manifest with TLS (k8s/ingress.yaml)

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - yourdomain.com
    secretName: wisecow-tls
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-service
            port:
              number: 80

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.1/cert-manager.yaml
# k8s/cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com  # Replace with your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

#Create a GitHub Actions workflow to automate building, pushing, and deploying the Docker image.

name: CI/CD Pipeline

on:
  push:
    branches:
      - main

env:
  REGISTRY: docker.io
  IMAGE_NAME: <your-dockerhub-username>/wisecow

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r app/requirements.txt

    - name: Lint with flake8
      run: |
        pip install flake8
        flake8 app/ --count --select=E9,F63,F7,F82 --show-source --statistics
        flake8 app/ --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics

    - name: Build Docker Image
      run: docker build -t $IMAGE_NAME:latest .

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Push Docker Image
      run: docker push $IMAGE_NAME:latest

    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'latest'

    - name: Configure Kubeconfig
      env:
        KUBECONFIG_DATA: ${{ secrets.KUBECONFIG_DATA }}
      run: |
        echo "$KUBECONFIG_DATA" | base64 --decode > $HOME/.kube/config

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/deployment.yaml
        kubectl apply -f k8s/service.yaml
        kubectl apply -f k8s/ingress.yaml

#To encode your kubeconfig:
cat ~/.kube/config | base64

#TLS Implementation

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.1/cert-manager.yaml

#Create ClusterIssuer:

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com  # Replace with your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx






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

