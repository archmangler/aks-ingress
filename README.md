# Example Ingress Configuration for AKS

* This is a complete guide for setting up an Ingress Controller and Ingress Resource on Azure Kubernetes Service (AKS) to expose a ClusterIP service.
* This setup uses an NGINX Ingress Controller to expose a ClusterIP service in AKS. 
* It includes routing HTTP/HTTPS traffic using an Ingress resource and supports DNS mapping for external access. 
* For production, enable TLS for secure communication.

# Solution Overview

* Ingress Controller Deployment: Deploy an NGINX Ingress Controller in your AKS cluster. It will act as the gateway to route HTTP/HTTPS traffic.
* ClusterIP Service: Ensure the target service is of type ClusterIP.
* Ingress Resource: Define an Ingress resource to route traffic to the ClusterIP service.
* External DNS: Map a DNS name to the public IP address assigned to the Ingress Controller.

## Step 1: Deploy the NGINX Ingress Controller

*Using Helm:*

* The simplest way to deploy an NGINX Ingress Controller in AKS is by using Helm.

Add the NGINX Ingress Helm repository:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
Install the NGINX Ingress Controller:
```

```
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.replicaCount=2 \
  --set controller.service.externalTrafficPolicy=Local
```

Verify Deployment:

```
kubectl get all -n ingress-nginx
This will deploy the Ingress Controller along with a LoadBalancer service. The LoadBalancer service will allocate a public IP address.
```

Retrieve the Public IP Address:

```
kubectl get service ingress-nginx-controller -n ingress-nginx
```


## Step 2: Deploy Your Application

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx
        ports:
        - containerPort: 80

```

* Create:

```
kubectl apply -f my-app-deployment.yaml
```

* Create ClusterIP Service

```
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```


## Step 3: Configure an Ingress Resource


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: my-app.example.com  # Replace with your domain name
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
``` 


##  Step 4: Configure DNS


* Retrieve the Public IP of the Ingress Controller:

```
kubectl get service ingress-nginx-controller -n ingress-nginx
```

* Update your DNS Zone to point your domain (my-app.example.com) to the public IP of the Ingress Controller.


## Step 5: Test the Setup

* Use curl or a web browser to test access to your service:

```
curl http://my-app.example.com
```

You should see the default NGINX welcome page (or your app's welcome page if you chose to install something else as your application).


## TLS Configuration

* Secure the Ingress using HTTPS by adding a TLS certificate.
* Create a TLS Secret:

```
kubectl create secret tls my-app-tls --cert=/path/to/cert.crt --key=/path/to/cert.key
```


* Update the Ingress Resource:

```
spec:
  tls:
  - hosts:
    - my-app.example.com
    secretName: my-app-tls
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

* Apply the updated Ingress:

```
kubectl apply -f my-app-ingress.yaml
```

