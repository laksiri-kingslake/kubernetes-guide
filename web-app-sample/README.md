# Deploying a Simple Web Application on Kubernetes

## Prerequisites
- A running Kubernetes cluster (minikube)
- kubectl  installed and configured
- A Docker image of the web application/service

## Steps
### Step 1: Create the Deployment
Here we use nginx to represent our web app. Replace 'nginx:1.7.9' with the docker image of web app/service<br>
1. Create my-webapp.yaml deployment file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-webapp
  template:
    metadata:
      labels:
        app: my-webapp
    spec:
      containers:
      - name: my-webapp
        image: my-app:1.7.9
        ports:
        - containerPort: 80
```

2. Apply deployment
```bash
kubectl apply -f my-webapp.yaml
```

### Step 2: Create the Service
1. Exposing the Nginx Deployment<br>
To expose the my-webapp Deployment, create a Service manifest (my-webapp-service.yaml):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-webapp-service
spec:
  selector:
    app: my-webapp
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007
  type: NodePort
```

2. Apply configuration
```bash
kubectl apply -f my-webapp-service.yaml
```

### Expose the Application to External Users
To make your application accessible from outside the cluster, you need to expose the Service. We can use NodePort or LoadBalancer

1. Using **NodePort**<br>
You can then access your application using http://[Node-IP]:[port]
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-webapp-service
spec:
  selector:
    app: my-webapp
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007
  type: NodePort
```

2. Using **LoadBalancer** <br>
In cloud environments like EKS we can use LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-webapp-service
spec:
  selector:
    app: my-webapp
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

After creating the Service, we can get the external IP address of the LoadBalancer using
```bash
kubectl get service my-webapp-service
```

### Step 3: Verify the Deployment and Service
To ensure that the application is deployed and accessible, we can check the status of the Deployment and Service.

1. Check Deployment Status
```bash
kubectl get deployments
```

2. Check Pod status
```bash
kubectl get pods
```

3. Check service status
```bash
kubectl get service my-webapp-service
```

### Scaling the Deployment
```bash
kubectl scale deployment my-webapp --replicas=5
```

### Updating the Deployment
To update the Deployment with a new version, image or configuration, we can modify the Deployment manifest and apply the changes.<br>
Example: Updating the Nginx Image<br>

1. Update the my-webapp.yaml file to use a different version of the web app:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-webapp
  template:
    metadata:
      labels:
        app: my-webapp
    spec:
      containers:
      - name: my-webapp
        image: my-app:1.7.10
        ports:
        - containerPort: 80
```

2. Apply the updated configuration:
```bash
kubectl apply -f my-webapp.yaml
```

## Prepare container image

```bash
docker build -t my-app:1.7.9 .
docker run -d -p 45678:80 my-app:1.7.9
```


## Reference
- https://medium.com/@platform.engineers/deploying-a-simple-web-application-on-kubernetes-43bbf724c23d