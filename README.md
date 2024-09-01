---

# Buyogo DevOps Intern Assignment

This repository contains the necessary files and instructions to deploy a Flask application connected to a MongoDB Atlas database on a Kubernetes cluster using Minikube. The setup includes resource management, autoscaling, and secure configurations.

## Building and Pushing Docker Image to DockerHub

To build and push the Docker image, follow these steps:

1. **Build the Docker Image:**
   ```bash
   docker build -t <your_dockerhub_username>/flask-mongo-app:2 .
   ```

2. **Push the Docker Image to DockerHub:**
   ```bash
   docker push <your_dockerhub_username>/flask-mongo-app:2
   ```

   You can verify the image on DockerHub to ensure it has been uploaded successfully.
   ![Screenshot (287)](https://github.com/user-attachments/assets/e151a340-0c14-46d6-ac67-4efa4538469c)


## Deploying the Flask Application and MongoDB on Minikube Kubernetes Cluster

To deploy the application and MongoDB on the Kubernetes cluster, execute the following commands:

1. **Apply Secrets:**
   ```bash
   kubectl apply -f secrets.yml
   ```
   This command applies the Kubernetes secrets configuration, which stores sensitive information like MongoDB URI, username, and password.

2. **Apply Deployment:**
   ```bash
   kubectl apply -f deployment.yml
   ```
   This command deploys the Flask application with the configurations specified in the YAML file.

3. **Apply Service:**
   ```bash
   kubectl apply -f service.yml
   ```
   This command creates a service to expose the Flask application.

4. **Apply MongoDB Service:**
   ```bash
   kubectl apply -f mongosvc.yml
   ```
   This command creates a service for MongoDB.

5. **Apply Horizontal Pod Autoscaler (HPA):**
   ```bash
   kubectl apply -f hpa.yml
   ```
   This command configures the HPA to automatically scale the number of pods based on CPU utilization.

To view the status of all resources, use the following command:

```bash
kubectl get all
![Screenshot 2024-08-31 200228](https://github.com/user-attachments/assets/2e02abe1-55e7-4455-a00b-bf40d5099213)

```

This command lists all pods, services, replicasets, statefulsets, and HPAs in the cluster.

## DNS Resolution in a Kubernetes Cluster

In a Kubernetes cluster, DNS resolution is crucial for inter-pod communication. Kubernetes includes a built-in DNS service that creates DNS records for services, allowing pods to communicate using service names instead of hardcoded IP addresses.

For example, in this assignment:

- The Flask application, deployed as a service (`flask-app`), needs to connect to the MongoDB service (`mongo-db`).
- The Flask pod sends a DNS query for `mongo-db.default.svc.cluster.local`, which is resolved by the cluster's DNS server to the IP address of the MongoDB service.
- The Flask pod then communicates with the MongoDB service using this IP address.

## Resource Requests and Limits in Kubernetes

### Overview

In a shared Kubernetes cluster, managing resource allocation is essential to prevent a single application from consuming all available resources, which could lead to system failures.

- **Limits:** Specify the maximum resources (CPU, memory) a container, pod, or namespace can use. Exceeding these limits prevents the object from being created.
- **Requests:** Specify the minimum resources required. If the requests exceed available resources, Kubernetes will not allow the object to be created.

### Design Choices

#### Resource Requests and Limits
- **Rationale:** Setting resource requests and limits ensures efficient resource usage without over-allocation, minimizing the risk of cluster collapse.
- **Alternatives:** Not setting limits could lead to resource competition, where one container might consume all resources, starving others.

#### Horizontal Pod Autoscaler (HPA)
- **Rationale:** HPA automatically scales pods based on CPU usage, allowing the application to handle variable traffic levels without manual intervention.
- **Alternatives:** Manual scaling of pods, which is less efficient and prone to human error.

#### Secret Management
- **Rationale:** Using Kubernetes Secrets to manage sensitive information (e.g., database URIs) keeps this data out of source code and version control systems, enhancing security.

## Testing Autoscaling and Database Interactions

### Autoscaling Test

1. **Setup:**
   - **HPA Configuration:** The HPA was configured to scale between 2 to 5 replicas based on a target CPU utilization of 70%.
   - **Deployment:** The Flask application was deployed with initial resource requests and limits set to trigger scaling under load.
   
2. **Load Generation with `wrk`:**
   - Installed `wrk`, a modern HTTP benchmarking tool, to generate load on the Flask application.
   - Command used:
     ```bash
     wrk -t12 -c400 -d60s http://localhost:5000
     ```

3. **Results:**
   - The HPA successfully scaled the number of pods from 2 to 5 as CPU utilization exceeded the 70% threshold during the load test.
   - After the load test, the HPA gradually scaled down the replicas back to 2 as CPU utilization decreased.

4. **Issues Encountered:**
   - **Slow Initial Scale-Up:** The initial scale-up took longer, potentially causing brief periods of high load. Adjusting the `--horizontal-pod-autoscaler-sync-period` parameter in the `kube-controller-manager` could reduce this delay.

### Database Interaction Testing

1. **Setup:**
   - **MongoDB Atlas:** The application was connected to a MongoDB Atlas cluster using a secret to store the connection URI.
   - **Deployment:** The Flask app was configured to handle multiple concurrent connections to MongoDB.

2. **Results:**
   - The Flask application handled all incoming requests and performed database operations (e.g., data retrieval, insertion) without errors or slowdowns.

---

Final results:
![image](https://github.com/user-attachments/assets/32d13580-fbaa-49bb-a464-63c07d266f6a)
![Screenshot (290)](https://github.com/user-attachments/assets/1c2fb0c3-65b0-4959-8b5e-a67956a04d96)

---
