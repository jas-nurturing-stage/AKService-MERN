# 🚀 MERN Microservices Deployment on Azure Kubernetes Service (AKS)

This project demonstrates deploying a **MERN application** (React frontend + Node.js microservices + MongoDB Atlas) on **Azure Kubernetes Service (AKS)**.

---

### 📂 Project Structure
```
mern-microservices-starter
├── Jenkinsfile
├── README.md
├── backend
│   ├── helloService
│   │   ├── Dockerfile
│   │   ├── index.js
│   │   ├── package-lock.json
│   │   └── package.json
│   └── profileService
│       ├── Dockerfile
│       ├── document.txt
│       ├── index.js
│       ├── package-lock.json
│       └── package.json
├── document.txt
├── frontend
│   ├── Dockerfile
│   ├── README.md
│   ├── nginx
│   │   └── nginx.conf
│   ├── package-lock.json
│   ├── package.json
│   ├── public/
│   └── src/
├── images/
├── k8s
│   ├── frontend
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── hello
│   │   ├── deployment.yaml
│   │   ├── secret.yaml
│   │   └── service.yaml
│   ├── namespace.yaml
│   └── profile
│       ├── deployment.yaml
│       ├── secret.yaml
│       └── service.yaml
```

### ⚙️ Install Azure CLI (Ubuntu/Debian)

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

This command downloads and installs the **Azure CLI** on Ubuntu/Debian systems.

### ⚙️ Install kubectl

```bash
az aks install-cli
```

Installs **kubectl**, the Kubernetes command-line client, using the Azure CLI.

### ✅ Verify Installations

```bash
az --version
docker --version
kubectl version --client
```

- **`az --version`**: Confirms the installed Azure CLI version.  
- **`docker --version`**: Verifies Docker is installed and available (required for container builds).  
- **`kubectl version --client`**: Displays the installed `kubectl` client version. 

Here’s an icon-enhanced, copy-paste friendly block for signing in and selecting the right Azure subscription.

## 🔐 Azure Login and Subscription Setup

### 🌐 Interactive Browser Login
```bash
az login
```
- Opens a browser/device flow to authenticate with Azure and populate the local Azure CLI context.

### 📋 List subscriptions 
```bash
az account list --output table
```
- Displays available subscriptions with names and IDs in a readable table for selection.

### 🎯 Set default subscription
```bash
az account set --subscription "<subscription-id>"
```
- Sets the active subscription so subsequent az and AKS commands target the correct account.
---

### 🚀 Create Azure Kubernetes Service (AKS) Cluster

```bash
az aks create \
  --resource-group DevopsB10 \
  --name AKSmernCluster \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --generate-ssh-keys
```

### 📋 Explanation of Flags
- **`az aks create`**: Command to create a new AKS cluster in Azure.  
- **`--resource-group DevopsB10`**: The name of the Azure resource group where the cluster will be created.  
- **`--name AKSmernCluster`**: The name assigned to the AKS cluster.  
- **`--node-count 2`**: Sets the number of agent nodes (worker nodes) in the cluster to 2.  
- **`--node-vm-size Standard_DS2_v2`**: Defines the virtual machine size for each node, in this case `Standard_DS2_v2`.  
- **`--generate-ssh-keys`**: Automatically generates SSH keys if not already present, allowing secure access to the cluster nodes.  

Would you like me to also include a **diagram style workflow** showing how this cluster setup fits into Azure (Resource Group → AKS → Nodes)?

![AKS Cluter](/images/aks-create.png)

### 🔑 Get Credentials for AKS Cluster

```bash
az aks get-credentials \
  --resource-group DevopsB10 \
  --name AKSmernCluster
```

### 📋 Explanation of Flags
- **`az aks get-credentials`**: Retrieves the access credentials (kubeconfig) required to connect and manage the AKS cluster with `kubectl`.  
- **`--resource-group DevopsB10`**: Specifies the resource group where the AKS cluster is located.  
- **`--name AKSmernCluster`**: Tells Azure which AKS cluster to get the credentials for.  

After running this command, you can directly use `kubectl get nodes` or other `kubectl` commands to manage the cluster.

![AKS ](/images/aks-switch-local-to-aks.png)

### ✅ Verify connection
```
kubectl get nodes
kubectl cluster-info
```
### 🌉 Create a Docker Network

Before running the services, create a Docker bridge network:

```bash
docker network create --driver bridge mernapp
```

### 📛 Create a Kubernetes Namespace

Before deploying your services, it's a best practice to isolate them within a dedicated Kubernetes namespace.

A predefined namespace file is available at: `k8s/namespace.yaml`
```bash
apiVersion: v1
kind: Namespace
metadata:
  name: mern-microservice
```
🚀 Apply the Namespace
Use the following command to create the namespace in your cluster:
```bash
kubectl apply -f k8s/namespace.yaml
```
![namespace k8s](/images/namespace-cli.png)
![namespace k8s](/images/namespace-ui.png)

#### 📂 Backend Repository Identification
🔗 **Repository**: `https://github.com/jas-nurturing-stage/AKService-MERN`

**Purpose:** This repository contains the backend code for the MERN application, built with Node.js and Express.js, handling API requests, MongoDB interactions, and authentication.

Technologies:

- **Node.js**: Runtime for the backend server.
- **Express.js**: Framework for building RESTful APIs.
- **MongoDB**: Database for storing application data.
- **Mongoose**: ODM for MongoDB.

```bash
git clone https://github.com/jas-nurturing-stage/AKService-MERN.git
```
For `Hello Service`
```bash
cd backend/helloService
npm install
npm start
```

For `Profile Service`
```bash
cd backend/profileService
npm install
npm start
```
#### 📂 Frontend Repository Identification
🔗 **Repository**: `https://github.com/jas-nurturing-stage/AKService-MERN`

**Purpose**: This repository contains the frontend code for the MERN application, built with React, providing a user interface for interacting with the backend APIs.

Technologies:

- **React**: Library for building the user interface.
- **Axios**: For making HTTP requests to the backend.
- **React Router**: For client-side routing.
- **Nginx**: For serving the production build.

```bash
git clone https://github.com/jas-nurturing-stage/AKService-MERN.git
cd backend/frontend
npm install
npm run dev
```

## ⚙️ Backend -  Express.js(Node Js)
### 👋 Hello Service
#### 🐳 Steps to Deploy an Application on Docker
📁 Create a `Dockerfile` inside the `backend/helloService` directory:

```Dockerfile
# Stage 1: Install Deps & build
FROM node:22-alpine AS builder

# Set the working directory in the container to /app
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy the rest of the application code to the working directory
COPY . .

# remove devDependencies
RUN npm prune --production

# Stage 2: Runtime: only prod artifacts on Alpine
#FROM node:lts-alpine as production
FROM node:22-alpine AS production

# Set the working directory in the container to /app
WORKDIR /app

# Install curl for healthchecks
RUN apk add --no-cache curl

# Copy only the necessary files from the builder stage
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app ./

# Build the application
EXPOSE 3001

# CMd to run the application
CMD ["node", "index.js"]
```
### 🔍 Local Testing & Validation
Build the Docker image:
```
docker image build --no-cache --network=mernapp -t docjas143/mern-microservices-hello .
```
![mern Hello microservices](/images/mern-hello-build.png)


Push the image to Docker Hub.
```
docker image push docjas143/mern-microservices-hello
```

Run the container:
```
docker container run -d --name mern-microservices-hello -e PORT=3001 -p 3001:3001 docjas143/mern-microservices-hello
```
**Base URL:** `http://localhost:3001`
![mern Hello microservices live](/images/mern-hello-live.png)

### ☸️ Steps to Deploy an Application on Kubernetes
##### 🗂️ 1. Create Secret
📄 **File**: `k8s/hello/secret.yaml`
```yml
apiVersion: v1
kind: Secret
metadata:
  name: hello-backend-secrets
  namespace: mern-microservice
type: Opaque
data:
  PORT: MzAwMQo=
```
📌 **Apply Secret**

```bash
kubectl apply -f k8s/hello/secret.yaml
```
🔍 **Verify Secret**
```bash
kubectl get secret -n mern-microservice
```
![secret](/images/hello-service-secret.png)
#### 📄 2 Create Deployment
**File:** `k8s/hello/deployment.yaml`
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-backend
  labels:
    app: hello-backend
    tier: hello-backend
    environment: production
  namespace: mern-microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-backend
  template:
    metadata:
      labels:
        app: hello-backend
        tier: hello-backend
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: mern-microservices-hello
        image: docjas143/mern-microservices-hello:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3001
        envFrom:
        - secretRef:
            name: hello-backend-secrets
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 40
          periodSeconds: 5
```

📌 **Apply Deployment**
```bash
kubectl apply -f k8s/hello/deployment.yaml
```
🔍 **Verify Pods**
```bash
kubectl get pods -n mern-microservice
```
📜 **View Logs to Confirm Communication**
```
kubectl logs deploy/hello-backend -n mern-microservice
```
OR
```
kubectl logs pod/<pod_name> -n mern-microservice
```
![secret](/images/hello-deploy-logs.png)
![Deploy](/images/hello-deploy-ui.png)
#### 📄 3 Create Service
**File:** `k8s/hello/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-backend
  namespace: mern-microservice
  labels:
    app: hello-backend
    tier: hello-backend
    environment: production
spec:
  selector:
    app: hello-backend
  ports:
  - port: 3001
    targetPort: 3001
  type: LoadBalancer
```

📌 **Apply Service**
```bash
kubectl apply -f k8s/hello/service.yaml
```
🔍 **Verify Service**
```bash
kubectl get svc -n mern-microservice
```

🔁 **Test Inter-Service Communication Using cURL**

Run a shell inside the user pod:
```bash
kubectl exec -it deploy/hello-backend -n mern-microservice -- sh
```
From inside the pod, test communication:
```bash
curl http://hello-backend.mern-microservice.svc.cluster.local:3001/health
```
![Hello service](/images/hello-service.png)
![Hello service UI](/images/service-ui.png)
#### 🧪 4 Test with Port Forwarding (For Localhost)
```bash
kubectl port-forward service/backend -n=learner-insights 3001:3001 --address=0.0.0.0
```
🌍 Access the service in your browser or tool like Postman:
run it
```bash
http://localhost:3001/health
```

---

### 👤 Profile Service
#### 🐳 Steps to Deploy an Application on Docker
📁 Create a `Dockerfile` inside the `backend/profileService` directory:
```Dockerfile
# Stage 1: Install Deps & build
FROM node:22-alpine AS builder

# Set the working directory in the container to /app
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy the rest of the application code to the working directory
COPY . .

# remove devDependencies
RUN npm prune --production

# Stage 2: Runtime: only prod artifacts on Alpine
#FROM node:lts-alpine as production
FROM node:22-alpine AS production

# Set the working directory in the container to /app
WORKDIR /app

# Install curl for healthchecks
RUN apk add --no-cache curl

# Copy only the necessary files from the builder stage
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app ./

# Build the application
EXPOSE 3002

# CMd to run the application
CMD ["node", "index.js"]
```

### 🔍 Local Testing & Validation
Build the Docker image:
```
docker image build --no-cache -t docjas143/mern-microservices-profile .
```
![mern Profile microservices](/images/mern-profile-build.png)

Push the image to Docker Hub.
```
docker image push docjas143/mern-microservices-profile
```

Run the container:
```
docker container run -d --name mern-microservices-profile --network=mernapp -e PORT=3002 -e MONGO_URL="mongodb://root:root@192.168.31.100:27017/mern-microservices?authSource=admin" -p 3002:3002 docjas143/mern-microservices-profile
```
**Base URL:** `http://localhost:3002`
![mern profile microservices live](/images/mern-profile-live.png)

### ☸️ Steps to Deploy an Application on Kubernetes
##### 🗂️ 1. Create Secret
📄 **File**: `k8s/profile/secret.yaml`
```yml
apiVersion: v1
kind: Secret
metadata:
  name: profile-backend-secrets
  namespace: mern-microservice
type: Opaque
data:
  PORT: MzAwMg==
  MONGO_URL: bW9uZ29kYjovL3Jvb3Q6cm9vdEAxOTIuMTY4LjMxLjEwMDoyNzAxNy9tZXJuLW1pY3Jvc2VydmljZXMtc3RhcnRlcj9hdXRoU291cmNlPWFkbWlu
```
📌 **Apply Secret**

```bash
kubectl apply -f k8s/profile/secret.yaml
```
🔍 **Verify Secret**
```bash
kubectl get secret -n mern-microservice
```
![Profile secret](/images/profile-secret.png)

#### 📄 2 Create Deployment
**File:** `k8s/profile/deployment.yaml`
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: profile-backend
  labels:
    app: profile-backend
    tier: profile-backend
    environment: production
  namespace: mern-microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: profile-backend
  template:
    metadata:
      labels:
        app: profile-backend
        tier: profile-backend
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: mern-microservices-profile
        image: docjas143/mern-microservices-profile:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3002
        envFrom:
        - secretRef:
            name: profile-backend-secrets
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3002
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3002
          initialDelaySeconds: 40
          periodSeconds: 5
```

📌 **Apply Deployment**
```bash
kubectl apply -f k8s/profile/deployment.yaml
```
🔍 **Verify Pods**
```bash
kubectl get pods -n mern-microservice
```
📜 **View Logs to Confirm Communication**
```
kubectl logs deploy/profile-backend -n mern-microservice
```
OR
```
kubectl logs pod/<pod_name> -n mern-microservice
```
![Profile Deploy](/images/profile-deploy-log.png)
![Profile Deploy](/images/deploy-ui.png)
#### 📄 3 Create Service
**File:** `k8s/profile/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: profile-backend
  namespace: mern-microservice
  labels:
    app: profile-backend
    tier: profile-backend
    environment: production
spec:
  selector:
    app: profile-backend
  ports:
  - port: 3002
    targetPort: 3002
  type: LoadBalancer
```

📌 **Apply Service**
```bash
kubectl apply -f k8s/profile/service.yaml
```
🔍 **Verify Service**
```bash
kubectl get svc -n mern-microservice
```

![service CLI](/images/service-cli.png)
![service UI](/images/service-ui.png)
🔁 **Test Inter-Service Communication Using cURL**

Run a shell inside the user pod:
```
kubectl exec -it deploy/profile-backend -n mern-microservice -- sh
```
From inside the pod, test communication:
```
curl http://profile-backend.mern-microservice.svc.cluster.local:3002/health
```

#### 🧪 4 Test with Port Forwarding (For localhost)
```bash
kubectl port-forward service/backend -n=mern-microservice 3002:3002 --address=0.0.0.0
```
🌍 Access the service in your browser or tool like Postman:
run it
```bash
http://localhost:3002/health
```

---
### 🎨 Frontend Service
#### 🐳 Steps to Deploy an Application on Docker
📁 Create a `Dockerfile` inside the `frontend` directory:

```Dockerfile
# STAGE 1: Build
FROM node:22-alpine AS builder

# Accept build-time API URL
ARG REACT_APP_API_BASE_URL_HELLO
ARG REACT_APP_API_BASE_URL_PROFILE
ENV REACT_APP_API_BASE_URL_HELLO=$REACT_APP_API_BASE_URL_HELLO
ENV REACT_APP_API_BASE_URL_PROFILE=$REACT_APP_API_BASE_URL_PROFILE

# Set the working directory in the container to /app
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy the rest of the application code to the working directory
COPY . .

RUN npm run build

# STAGE 2: Serve
FROM nginx:alpine

# Install curl for healthchecks
RUN apk add --no-cache curl

# Set the working directory in the container to /usr/share/nginx/html
RUN rm -rf /usr/share/nginx/html/*

# Copy only the necessary files from the builder stage
COPY --from=builder /app/build /usr/share/nginx/html

# Copy custom Nginx config
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf

# Expose
EXPOSE 80

# Build the application
CMD ["nginx", "-g", "daemon off;"]
```
### 🔍 Local Testing & Validation
Build the Docker image:
```
docker image build --build-arg REACT_APP_API_BASE_URL_HELLO=http://192.168.31.100:3001 --build-arg REACT_APP_API_BASE_URL_PROFILE=http://192.168.31.100:3002 -t docjas143/mern-microservices-frontend .
```
![mern Frontend](/images/mern-frontend-build.png)


Push the image to Docker Hub.
```
docker image push docjas143/mern-microservices-frontend
```

Run the container:
```
docker container run -d --name mern-microservices-frontend -p 3000:80 docjas143/mern-microservices-frontend
```
**Base URL:** `http://localhost:3000`
![mern Frontend live](/images/mern-frontend-live.png)


### ☸️ Steps to Deploy an Application on Kubernetes
#### 📄 2 Create Deployment
**File:** `k8s/frontend/deployment.yaml`
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
    tier: frontend
    environment: production
  namespace: mern-microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: mern-microservices-frontend
        image: docjas143/mern-microservices-frontend:latest
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 40
          periodSeconds: 5
```

📌 **Apply Deployment**
```bash
kubectl apply -f k8s/frontend/deployment.yaml
```
🔍 **Verify Pods**
```bash
kubectl get pods -n mern-microservice
```
📜 **View Logs to Confirm Communication**
```
kubectl logs deploy/frontend -n mern-microservice
```
OR
```
kubectl logs pod/<pod_name> -n mern-microservice
```
![Frontend Deploy](/images/frontend-deploy-log.png)

![Frontend Deploy](/images/deploy-ui.png)

#### 📄 3 Create Service
**File:** `k8s/frontend/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: mern-microservice
  labels:
    app: frontend
    tier: frontend
    environment: production
spec:
  selector:
    app: frontend
  ports:
  - port: 3000
    targetPort: 80
  type: LoadBalancer
```

📌 **Apply Service**
```bash
kubectl apply -f k8s/frontend/service.yaml
```
🔍 **Verify Service**
```bash
kubectl get svc -n mern-microservice
```
![Frontend Service](/images/service-cli.png)
![Frontend Service](/images/service-ui.png)
#### 🧪 4 Test with Port Forwarding (For localhost)
```bash
kubectl port-forward service/backend -n=mern-microservice 3000:80 --address=0.0.0.0
```
🌍 Access the service in your browser or tool like Postman:
run it
```bash
http://localhost:3000
```

### **🌍 Production App Live:**
![AKS ](/images/production-live.png)

---
## 📜 Project Information

### 📄 License Details
This project is released under the MIT License, granting you the freedom to:
- 🔓 Use in commercial projects
- 🔄 Modify and redistribute
- 📚 Use as educational material

## 📞 Contact

📧 Email: [Email Me](sk.jasminesaheb@gmail.com
)
🔗 LinkedIn: [LinkedIn Profile](https://www.linkedin.com/in/sagar-93-patel)  
🐙 GitHub: [GitHub Profile](https://github.com/jas-nurturing-stage)  

---

<div align="center">
  <p>Built with ❤️ by Jasmine Shaik</p>
</div>