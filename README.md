# Kubernetes Microservices Deployment Using Ingress

## Overview

This project shows how to deploy a real stateless microservices-style application on Kubernetes.
The goal is to practice how applications are actually deployed in production Kubernetes clusters.

This task focuses on deploying frontend and backend services, managing configuration and secrets, and exposing applications using an Ingress controller.

This project assumes a working Kubernetes cluster is already running.

---

## Step 1: Verify Namespace

Check namespaces.

```bash
kubectl get namespaces
```

Create the dev namespace if not present.

```bash
kubectl create namespace dev
```

---

## Step 2: Deploy Backend API

Create backend deployment.

```bash
kubectl create deployment backend-api --image=nginx -n dev
```

Check pods.

```bash
kubectl get pods -n dev
```

Expose backend as an internal service.

```bash
kubectl expose deployment backend-api \
--name=backend-service \
--port=80 \
--type=ClusterIP \
-n dev
```

Verify service.

```bash
kubectl get svc -n dev
```

---

## Step 3: Deploy Frontend Service

Create frontend deployment.

```bash
kubectl create deployment frontend --image=nginx -n dev
```

Expose frontend service.

```bash
kubectl expose deployment frontend \
--name=frontend-service \
--port=80 \
--type=ClusterIP \
-n dev
```

Check services.

```bash
kubectl get svc -n dev
```

---

## Step 4: Create ConfigMap

Create application configuration.

```bash
kubectl create configmap app-config \
--from-literal=APP_ENV=dev \
--from-literal=APP_NAME=frontend-app \
-n dev
```

Verify ConfigMap.

```bash
kubectl get configmap -n dev
```

---

## Step 5: Create Secret

Create secret for sensitive values.

```bash
kubectl create secret generic app-secret \
--from-literal=DB_PASSWORD=secret123 \
-n dev
```

Verify secret.

```bash
kubectl get secrets -n dev
```

---

## Step 6: Update Deployments to Use Config and Secret

Edit frontend deployment.

```bash
kubectl edit deployment frontend -n dev
```

Add environment variable from ConfigMap.

```yaml
env:
- name: APP_ENV
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_ENV
```

Edit backend deployment.

```bash
kubectl edit deployment backend-api -n dev
```

Add environment variable from Secret.

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: DB_PASSWORD
```

Check rollout status.

```bash
kubectl rollout status deployment frontend -n dev
kubectl rollout status deployment backend-api -n dev
```

---

## Step 7: Install NGINX Ingress Controller

Install ingress controller.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Wait for ingress pods.

```bash
kubectl get pods -n ingress-nginx
```

---

## Step 8: Create Ingress Resource

Create ingress.yaml.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: dev
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

Apply ingress.

```bash
kubectl apply -f ingress.yaml
```

Check ingress.

```bash
kubectl get ingress -n dev
```

---

## Step 9: Access Application

Get ingress controller service details.

```bash
kubectl get svc -n ingress-nginx
```

Use the external IP or NodePort to access the frontend application.

---

## Step 10: Scale Frontend Application

Scale frontend replicas.

```bash
kubectl scale deployment frontend --replicas=3 -n dev
```

Verify pods.

```bash
kubectl get pods -n dev
```

---
