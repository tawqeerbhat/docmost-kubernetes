<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Docmost Deployment on Kubernetes - Complete Step-by-Step Guide</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 20px;
        }
        h1, h2, h3 {
            color: #333;
        }
        code {
            background-color: #f4f4f4;
            padding: 2px 5px;
            border-radius: 3px;
            font-family: monospace;
        }
        pre {
            background-color: #f4f4f4;
            padding: 10px;
            border-radius: 5px;
            overflow-x: auto;
        }
    </style>
</head>
<body>

<h1>Docmost Deployment on Kubernetes - Complete Step-by-Step Guide</h1>

<p>This guide provides detailed instructions to deploy the <strong>Docmost</strong> app on a Kubernetes cluster. It includes all the necessary steps, commands, and configurations.</p>

<h2>Step 1: Clone the Repository</h2>
<ol>
    <li>Open your terminal.</li>
    <li>Run the following command to clone the repository:
        <pre><code>git clone https://github.com/your-username/docmost-k8s.git</code></pre>
    </li>
    <li>Navigate to the repository folder:
        <pre><code>cd docmost-k8s</code></pre>
    </li>
</ol>

<h2>Step 2: Log in to Docker Hub</h2>
<ol>
    <li>Log in to Docker Hub using your credentials:
        <pre><code>docker login</code></pre>
    </li>
    <li>Enter your Docker Hub username and password when prompted.</li>
</ol>

<h2>Step 3: Build and Push the Docker Image</h2>
<ol>
    <li>Build the Docker image for the Docmost app:
        <pre><code>docker build -t tawkeer/docmost:latest .</code></pre>
    </li>
    <li>Push the Docker image to Docker Hub:
        <pre><code>docker push tawkeer/docmost:latest</code></pre>
    </li>
</ol>

<h2>Step 4: Update Docker Compose File</h2>
<p>We modified the <code>docker-compose.yml</code> file to include PostgreSQL and Redis. Here’s the updated file:</p>
<pre><code>
version: '3'

services:
  docmost:
    image: docmost/docmost:latest
    depends_on:
      - db
      - redis
    environment:
      APP_URL: 'http://localhost:3000'
      APP_SECRET: 'supersecretkey123'
      DATABASE_URL: 'postgresql://docmost:mypassword123@db:5432/docmost?schema=public'
      REDIS_URL: 'redis://redis:6379'
    ports:
      - "3000:3000"
    restart: unless-stopped
    volumes:
      - docmost:/app/data/storage

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: docmost
      POSTGRES_USER: docmost
      POSTGRES_PASSWORD: mypassword123
    restart: unless-stopped
    volumes:
      - db_data:/var/lib/postgresql/data

  redis:
    image: redis:7.2-alpine
    restart: unless-stopped
    volumes:
      - redis_data:/data

volumes:
  docmost:
  db_data:
  redis_data:
</code></pre>

<h2>Step 5: Deploy Redis</h2>
<ol>
    <li>Create a file named <code>redis-deployment.yaml</code> and add the following content:
        <pre><code>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7.2-alpine
        ports:
        - containerPort: 6379
        </code></pre>
    </li>
    <li>Create a file named <code>redis-service.yaml</code> and add the following content:
        <pre><code>
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
        </code></pre>
    </li>
    <li>Apply the Redis manifests:
        <pre><code>
kubectl apply -f redis-deployment.yaml
kubectl apply -f redis-service.yaml
        </code></pre>
    </li>
</ol>

<h2>Step 6: Deploy PostgreSQL</h2>
<ol>
    <li>Create a file named <code>postgres-deployment.yaml</code> and add the following content:
        <pre><code>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16-alpine
        env:
        - name: POSTGRES_DB
          value: "docmost"
        - name: POSTGRES_USER
          value: "docmost"
        - name: POSTGRES_PASSWORD
          value: "mypassword123"
        ports:
        - containerPort: 5432
        </code></pre>
    </li>
    <li>Create a file named <code>postgres-service.yaml</code> and add the following content:
        <pre><code>
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
        </code></pre>
    </li>
    <li>Apply the PostgreSQL manifests:
        <pre><code>
kubectl apply -f postgres-deployment.yaml
kubectl apply -f postgres-service.yaml
        </code></pre>
    </li>
</ol>

<h2>Step 7: Deploy Docmost</h2>
<ol>
    <li>Create a file named <code>docmost-deployment.yaml</code> and add the following content:
        <pre><code>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docmost
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docmost
  template:
    metadata:
      labels:
        app: docmost
    spec:
      containers:
      - name: docmost
        image: tawkeer/docmost:latest
        env:
        - name: DATABASE_URL
          value: "postgresql://docmost:mypassword123@postgres:5432/docmost?schema=public"
        - name: REDIS_URL
          value: "redis://redis:6379"
        - name: APP_SECRET
          value: "supersecretkey123"
        ports:
        - containerPort: 3000
        </code></pre>
    </li>
    <li>Create a file named <code>docmost-service.yaml</code> and add the following content:
        <pre><code>
apiVersion: v1
kind: Service
metadata:
  name: docmost
spec:
  type: LoadBalancer
  selector:
    app: docmost
  ports:
  - port: 3000
    targetPort: 3000
        </code></pre>
    </li>
    <li>Apply the Docmost manifests:
        <pre><code>
kubectl apply -f docmost-deployment.yaml
kubectl apply -f docmost-service.yaml
        </code></pre>
    </li>
</ol>

<h2>Step 8: Access the App</h2>
<ol>
    <li>Run the following command to get the external IP or NodePort:
        <pre><code>kubectl get services</code></pre>
    </li>
    <li>Access the app:
        <ul>
            <li>If using a cloud provider, access the app at:
                <pre><code>http://&lt;external-ip&gt;:3000</code></pre>
            </li>
            <li>If using Minikube, run:
                <pre><code>minikube service docmost</code></pre>
            </li>
            <li>If using a local cluster, access the app at:
                <pre><code>http://&lt;node-ip&gt;:30100</code></pre>
            </li>
        </ul>
    </li>
</ol>

<h2>Step 9: Expose the App to the Internet (Optional)</h2>
<p>To expose the app to the internet, you can:</p>
<ol>
    <li><strong>Use a LoadBalancer</strong>:
        Update the <code>docmost-service.yaml</code> file to use <code>LoadBalancer</code> and apply the changes.
    </li>
    <li><strong>Set up an Ingress Controller</strong>:
        Install an Ingress Controller (e.g., NGINX Ingress) and create an Ingress resource for Docmost.
    </li>
</ol>

<h2>Step 10: Troubleshooting</h2>
<p>If you encounter issues, check the following:</p>
<ol>
    <li><strong>Pod Logs</strong>:
        Check the logs of the <code>docmost</code> pod:
        <pre><code>kubectl logs &lt;pod-name&gt;</code></pre>
    </li>
    <li><strong>Service Endpoints</strong>:
        Verify that the service is correctly mapped to the pod:
        <pre><code>kubectl describe service docmost</code></pre>
    </li>
    <li><strong>Firewall Rules</strong>:
        Ensure the required ports (e.g., <code>3000</code>, <code>30100</code>) are open on your Kubernetes node.
    </li>
</ol>

<h2>Step 11: Clean Up</h2>
<p>To delete all resources created by this deployment, run:</p>
<pre><code>
kubectl delete -f docmost-deployment.yaml
kubectl delete -f docmost-service.yaml
kubectl delete -f postgres-deployment.yaml
kubectl delete -f postgres-service.yaml
kubectl delete -f redis-deployment.yaml
kubectl delete -f redis-service.yaml
</code></pre>

<h2>Contributing</h2>
<p>Contributions are welcome! If you find any issues or have suggestions for improvement, please open an issue or submit a pull request.</p>

<h2>License</h2>
<p>This project is licensed under the MIT License. See the <a href="LICENSE">LICENSE</a> file for details.</p>

<h2>Acknowledgments</h2>
<ul>
    <li><a href="https://github.com/docmost/docmost">Docmost</a> for the open-source web app.</li>
    <li>Kubernetes for the orchestration platform.</li>
</ul>

</body>
</html>
