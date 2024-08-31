

---

# Beyogo DevOps Intern Assignment

This repository contains the necessary files and instructions to deploy a Flask application connected to a MongoDB Atlas database on a Kubernetes cluster using Minikube. The setup includes resource management, autoscaling, and secure configurations.

## Building and Pushing Docker Image to DockerHub

To build the Docker image, follow the command below:

```bash
docker build -t <your_dockerhub_username>/flask-mongo-app:2 .
```

To push the built Docker image to DockerHub, use the command below:

```bash
docker push <your_dockerhub_username>/flask-mongo-app:2
```

You can verify the image on DockerHub. You should see a similar output like this:

![DockerHub Verification](https://github.com/user-attachments/assets/2e0c185f-1cc9-4ba6-99f2-4fbdfec2634f)

## Deploying the Flask Application and MongoDB on Minikube Kubernetes Cluster

To deploy on the Kubernetes cluster, you need to follow the commands below:

1. **Apply Secrets:**
   ```bash
   kubectl apply -f secrets.yml
   ```
   - This command is used to apply or create resources in your Kubernetes cluster from a YAML configuration file. Since we have some sensitive information like MongoDB URI, username, and password, it's better to store them in `secret.yml` file.

2. **Apply Deployment:**
   ```bash
   kubectl apply -f deployment.yml
   ```
   - This command creates a deployment with the configurations mentioned in the YAML file.

3. **Apply Service:**
   ```bash
   kubectl apply -f service.yml
   ```
   - This command creates a service, which allows us to access the application.

4. **Apply MongoDB Service:**
   ```bash
   kubectl apply -f mongosvc.yml
   ```
   - This command creates a MongoDB service.

5. **Apply Horizontal Pod Autoscaler (HPA):**
   ```bash
   kubectl apply -f hpa.yml
   ```
   - This command is used to scale the pods depending on the CPU usage.

You can view all the outputs by using the command below:

```bash
kubectl get all
```

This command lists all pods, services, replicasets, statefulsets, and HPAs.

![Kubernetes Output](https://github.com/user-attachments/assets/be81d687-780c-42a6-8561-f637bcf4835d)
DNS Resolution within a Kubernetes Cluster for Inter-Pod Communication
In a Kubernetes cluster, DNS (Domain Name System) resolution plays a crucial role in enabling inter-pod communication. Kubernetes includes a built-in DNS service that automatically creates DNS records for Kubernetes services, allowing pods to discover and communicate with each other by service names rather than hardcoded IP addresses.
in the given assignment wehave a Flask application deployed as a service (flask-app) and a MongoDB service (mongo-db). The Flask app needs to connect to the MongoDB database to perform operations.

The Flask pod sends a DNS query for mongo-db.default.svc.cluster.local to the cluster's DNS server.
The DNS server resolves this name to the IP address of the MongoDB service.
The Flask pod then uses this IP address to send requests to the MongoDB service, which forwards them to one of the MongoDB pods.

## Resource Requests and Limits in Kubernetes

### Overview
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
In a shared Kubernetes cluster, it's essential to manage resource allocation to ensure that no single team or application consumes all available resources, leading to potential system failures.

- **Limit:** Specifies the maximum resources (CPU, memory) that a container, pod, or namespace can use. If the limit is exceeded, the object won’t be created.
- **Request:** Specifies the minimum resources required for a container, pod, or namespace. If the request exceeds the available resources, Kubernetes won't allow the creation of the object.

### Design Choices

#### Resource Requests and Limits

- **Why Chosen:** Setting resource requests and limits ensures optimal resource usage without over-allocation, reducing the chances of cluster collapse.
- **Alternatives Considered:** Not setting resource limits can lead to resource competition, where a single container might use up all resources, starving others.

#### Horizontal Pod Autoscaler (HPA)

- **Why Chosen:** HPA automatically scales the number of pods based on CPU usage, ensuring the application can handle variable traffic levels without manual intervention.
- **Alternatives Considered:** Manually scaling pods, which is less efficient and prone to human error.

#### Secret Management

- **Why Chosen:** Using Kubernetes Secrets to manage sensitive information like database URIs keeps sensitive data out of source code and version control systems, ensuring better security.

## Testing Autoscaling and Database Interactions

### Autoscaling Test

To ensure that the Flask-MongoDB application works as expected under various conditions, I conducted thorough testing, focusing on autoscaling and database interactions.

1. **Setup:**
   - **HPA Configuration:** The HPA was configured to scale between 2 to 5 replicas based on a target CPU utilization of 70%.
   - **Deployment:** The Flask application was deployed with initial resource requests and limits set to trigger scaling under load.
   
2. **Load Generation with `wrk`:**
   - Installed `wrk`, a modern HTTP benchmarking tool, to generate a significant load on the Flask application.
   - Command used:
     ```bash
     wrk -t12 -c400 -d60s http://localhost:5000
     ```

3. **Results:**
   - The HPA successfully scaled the number of pods from 2 to 5 as CPU utilization exceeded the 70% threshold during the load test.
   - Once the load test was completed, the HPA gradually scaled down the number of replicas back to 2 as CPU utilization decreased.

4. **Issues Encountered:**
   - **Slow Initial Scale-Up:** The first scale-up took slightly longer, potentially leading to brief periods where the pods were under high load. Adjusting the `--horizontal-pod-autoscaler-sync-period` parameter in the `kube-controller-manager` could help reduce this delay.

### Database Interaction Testing

1. **Setup:**
   - **MongoDB Atlas:** The application was connected to a MongoDB Atlas cluster using a secret to store the connection URI.
   - **Deployment:** The Flask app was configured to handle multiple concurrent connections to MongoDB.

2. **Results:**
   - The Flask application handled all incoming requests and performed database operations (e.g., data retrieval, insertion) without any errors or slowdowns.

---

final results:
![image](https://github.com/user-attachments/assets/32d13580-fbaa-49bb-a464-63c07d266f6a)
![Screenshot (290)](https://github.com/user-attachments/assets/1c2fb0c3-65b0-4959-8b5e-a67956a04d96)

