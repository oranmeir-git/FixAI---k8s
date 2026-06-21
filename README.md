# FixAI 🛠️✨ — Kubernetes Edition

FixAI is a web application designed to help users get instant, smart cost estimations for various repairs based on the Israeli market. Powered by Flask, MySQL, and Google Gemini AI, it allows users to enter a repair request, upload an image, and receive an estimated price range and advice.

This version of the project is fully containerized and orchestrated with **Kubernetes**, while keeping Docker Compose available for local development.

---

## 🚀 Key Features

* **AI-Powered Estimates**: Uses Google's `gemini-flash-lite-latest` model to analyze text and image inputs and generate accurate repair price ranges tailored for the Israeli market.
* **Todo/Tracking System**: Log and track repair items with their completion status and AI advice history.
* **Kubernetes Orchestration**: Production-grade deployment with Deployments, Services, ConfigMaps, Secrets, and PersistentVolumeClaims.
* **Docker Compose (Local Dev)**: Quick local setup with a single `docker compose up` command.
* **Reverse Proxy**: Nginx handles public-facing traffic and routes requests to the Flask backend.
* **Monitoring & Metrics**: Prometheus collects Flask application metrics, which are visualized using Grafana dashboards.
* **CI/CD Pipeline**: GitHub Actions running integration tests, building Docker images, and deploying to Kubernetes.

---

## 🛠️ Tech Stack

* **Backend**: Python, Flask, Flask-SQLAlchemy (MySQL)
* **AI Model**: Google GenAI SDK (`google-genai` & `gemini-flash-lite-latest`)
* **Database**: MySQL 5.7
* **Web Server / Reverse Proxy**: Nginx
* **Monitoring**: Prometheus, Grafana, `prometheus-flask-exporter`
* **Orchestration**: Kubernetes (Minikube for local, any cluster for production)
* **Containerization**: Docker & Docker Compose (local development)
* **CI/CD**: GitHub Actions

---

## 📦 Architecture

The system is split into multiple services:

| Service | Description | Port |
|---------|-------------|------|
| `flask-app` | Python Flask backend (2 replicas in K8s) | 5000 (internal) |
| `mysql` | MySQL 5.7 database with PersistentVolume | 3306 (internal) |
| `nginx` | Reverse proxy routing traffic to Flask | 80 (NodePort) |
| `prometheus` | Scrapes Flask app metrics | 9090 (NodePort) |
| `grafana` | Visualization dashboards for Prometheus | 3000 (NodePort) |
| `testserver` | Integration tests (K8s Job / Docker container) | — |

---

## ⚙️ Getting Started

### Prerequisites
* **Docker** and **Docker Compose** (for local development)
* **Minikube** or any Kubernetes cluster (for K8s deployment)
* **kubectl** CLI
* A Google Gemini API Key from [Google AI Studio](https://aistudio.google.com/)

---

### 🐳 Option 1: Local Development (Docker Compose)

1. **Clone the repository:**
   ```bash
   git clone https://github.com/oranmeir-git/FixAI---k8s.git
   cd FixAI---k8s
   ```

2. **Configure environment variables:**
   Create a `.env` file in the root directory:
   ```env
   MYSQL_ROOT_PASSWORD=your_secure_root_password
   MYSQL_USER=flask
   MYSQL_PASSWORD=your_secure_user_password
   GEMINI_KEY=your_gemini_api_key
   ```

3. **Start the application:**
   ```bash
   docker compose up -d --build
   ```

4. **Access the services:**
   * **Web Application**: [http://localhost](http://localhost)
   * **Prometheus Dashboard**: [http://localhost:9090](http://localhost:9090)
   * **Grafana Dashboard**: [http://localhost:3000](http://localhost:3000)

5. **Stop:**
   ```bash
   docker compose down
   ```

---

### ☸️ Option 2: Kubernetes Deployment (Minikube)

1. **Start Minikube:**
   ```bash
   minikube start
   ```

2. **Create the secrets file:**
   Copy the template and fill in your base64-encoded values:
   ```bash
   cp k8s/secrets.yaml.example k8s/secrets.yaml
   ```
   Edit `k8s/secrets.yaml` and replace placeholders:
   ```bash
   # Encode your values:
   echo -n "your_password" | base64
   echo -n "flask" | base64
   echo -n "your_gemini_api_key" | base64
   ```

3. **Deploy all resources:**
   ```bash
   # Create namespace
   kubectl apply -f k8s/namespace.yaml

   # Apply secrets and config
   kubectl apply -f k8s/secrets.yaml
   kubectl apply -f k8s/configmaps/ --recursive

   # Deploy services
   kubectl apply -f k8s/mysql/
   kubectl apply -f k8s/flask-app/
   kubectl apply -f k8s/nginx/
   kubectl apply -f k8s/prometheus/
   kubectl apply -f k8s/grafana/
   ```

4. **Check pod status:**
   ```bash
   kubectl get pods -n fixai
   ```

5. **Access the services:**
   ```bash
   # Web Application
   minikube service nginx -n fixai

   # Prometheus Dashboard
   minikube service prometheus -n fixai

   # Grafana Dashboard
   minikube service grafana -n fixai
   ```

6. **Run integration tests:**
   ```bash
   kubectl apply -f k8s/tests/test-job.yaml
   kubectl logs -f job/fixai-integration-test -n fixai
   ```

7. **Tear down:**
   ```bash
   kubectl delete namespace fixai
   ```

---

## 🧪 Testing

### Docker Compose (local)
```bash
docker compose run --rm testserver
```

### Kubernetes
```bash
kubectl apply -f k8s/tests/test-job.yaml
kubectl logs -f job/fixai-integration-test -n fixai
```

---

## 🌐 CI/CD Pipeline

This repository uses GitHub Actions for continuous integration and deployment:

### CI/CD Pipeline (`ci.yml`)
* Triggered on every push to `main`
* Spins up Docker Compose services inside the runner
* Runs the integration test suite using `testserver`
* Builds the Flask application Docker image
* Pushes it to Docker Hub with both `latest` and SHA tags

### Deploy to Kubernetes (`deploy-k8s.yml`)
* Triggered when the CI/CD Pipeline successfully completes
* Applies all Kubernetes manifests to the cluster
* Updates the Flask app deployment with the new image tag
* Verifies the rollout completes successfully

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `DOCKER_HUB_USERNAME` | Docker Hub username |
| `DOCKER_HUB_TOKEN` | Docker Hub access token |
| `DB_USER` | MySQL username |
| `DB_PASSWORD` | MySQL password |
| `GEMINI_KEY` | Google Gemini API key |
| `KUBE_CONFIG` | Base64-encoded kubeconfig for cluster access |

---

## 📁 Project Structure

```
.
├── app.py                  # Flask application
├── Dockerfile              # App container image
├── docker-compose.yml      # Local development setup
├── nginx.conf              # Nginx config (for Docker Compose)
├── requirements.txt        # Python dependencies
├── test.Dockerfile         # Test container image
├── testserver.py           # Integration tests
├── templates/              # HTML templates
├── static/                 # Static assets (CSS, JS, images)
├── db/                     # MySQL config files
├── prometheus/             # Prometheus config
├── grafana/                # Grafana provisioning
├── k8s/                    # ☸️ Kubernetes manifests
│   ├── namespace.yaml
│   ├── secrets.yaml        # (gitignored — template only)
│   ├── configmaps/         # ConfigMaps for all services
│   ├── mysql/              # MySQL Deployment + Service + PVC
│   ├── flask-app/          # Flask Deployment + Service
│   ├── nginx/              # Nginx Deployment + Service (NodePort)
│   ├── prometheus/         # Prometheus Deployment + Service
│   ├── grafana/            # Grafana Deployment + Service
│   └── tests/              # Integration test Job
└── .github/workflows/      # CI/CD pipelines
    ├── ci.yml              # Build, test, push
    └── deploy-k8s.yml      # Deploy to K8s cluster
```
