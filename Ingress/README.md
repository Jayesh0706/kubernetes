Here's a step-by-step guide summarizing everything we did to set up the NGINX Ingress Controller and configure an Ingress resource in Kubernetes using Killercoda. This will serve as a detailed reference for future use.

* * * * *

### **1\. Set Up NGINX Ingress Controller**

#### **Install NGINX Ingress Controller**

1.  Use the official Kubernetes YAML file to deploy the NGINX Ingress Controller:

    `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

2.  Verify the deployment of NGINX Ingress Controller:


    `kubectl get pods -n ingress-nginx`

    Ensure all pods are in the `Running` state.

3.  Check the `ingress-nginx-controller` service:


    `kubectl get svc -n ingress-nginx`

    Note the `NodePort` for HTTP (e.g., `32209`) and HTTPS (e.g., `31990`).

* * * * *

### **2\. Deploy a Backend Application**

We deployed a sample backend to test the Ingress.

#### **Create a Deployment and Service**

1. Define a simple Deployment and Service (app-deployment.yml):

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80```
2.  Apply the YAML file:


    `kubectl apply -f app-deployment.yml`

3.  Verify the deployment:

    `kubectl get pods
    kubectl get svc`

* * * * *

### **3\. Configure the Ingress Resource**

1.  Create an Ingress YAML file (`ingress.yml`):


    `apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - host: eg.first.com
        http:
          paths:
          - path: /first
            pathType: Prefix
            backend:
              service:
                name: myservice
                port:
                  number: 80`

2.  Apply the Ingress resource:

    `kubectl apply -f ingress.yml`

3.  Verify the Ingress configuration:


    `kubectl get ingress
    kubectl describe ingress my-ingress`

* * * * *

### **4\. Access the Application**

#### **Add Hostname to `/etc/hosts`**

Since Killercoda does not support a LoadBalancer service, map the Node's IP address to the hostname specified in the Ingress (`eg.first.com`).

1.  Find the Node IP:


    `kubectl get nodes -o wide`

    Example Node IP: `172.30.2.2`.

2.  Add the mapping:

    `echo "172.30.2.2 eg.first.com" | sudo tee -a /etc/hosts`

#### **Test Access**

1.  Access the application using the NodePort (HTTP):

    `curl http://eg.first.com:32209/first`

    This should return the NGINX welcome page.

* * * * *

### **5\. Troubleshooting**

1.  **Check Service NodePort:**

    `kubectl get svc -n ingress-nginx ingress-nginx-controller`

2.  **Inspect Logs for NGINX Controller:**


    `kubectl logs -n ingress-nginx <ingress-nginx-pod-name>`

3.  **Verify Backend Service:**


    `kubectl get endpoints myservice`

4.  **Verify Ingress Resource:**

    `kubectl describe ingress my-ingress`

* * * * *

### **6\. Customizing NodePort**

If you want to assign a custom NodePort, edit the NGINX service:

1.  Edit the service:


    `kubectl edit svc -n ingress-nginx ingress-nginx-controller`

2.  Modify the `nodePort` under the `ports` section:


    `ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      nodePort: 30443`

* * * * *

### **7\. Cleanup**

To clean up resources when you're done:

1.  Delete the Ingress:

    `kubectl delete ingress my-ingress`

2.  Delete the application:


    `kubectl delete -f app-deployment.yml`

3.  Uninstall the NGINX Ingress Controller:


    `kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/dep`