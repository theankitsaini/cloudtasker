# CloudTasker: A Lean Kubernetes & CI/CD Pipeline on AWS

CloudTasker is a secure Task Management API I built to get hands-on experience with production DevOps practices. The goal was to build a secure, containerized app with full CI/CD and self-healing infrastructure, while keeping cloud costs strictly at **\$0/month**.

Instead of blowing my budget or AWS credits on an expensive managed service like AWS EKS, I designed this setup to run entirely within the AWS Free Tier. It uses an ultra-lightweight Kubernetes distribution (**K3s**) hosted on a single small EC2 instance.

---

## 🛠️ The Tech Stack

- **Backend API:** Python (Flask), Flask-SQLAlchemy, Flask-JWT-Extended
- **Database:** SQLite (Lightweight, file-based persistence)
- **Containerization:** Docker
- **Orchestration:** Kubernetes (via K3s)
- **Cloud Infrastructure:** AWS (EC2, ECR)
- **CI/CD Pipeline:** GitHub Actions

---

## 🚀 How It Works (The Workflow)

```text
 [ Code Commit ] ──> [ GitHub Actions Pipeline ] ──> [ Builds Docker Image ]
                                                             │
                                                             ▼
 [ Live API Live on Host ] <── [ K3s Triggers Rollout ] <── [ Pushes to AWS ECR ]
```

1. I push code updates to the `main` branch on GitHub.
2. **GitHub Actions** catches the push and automatically triggers a build runner.
3. The runner packages the app into an optimized **Docker image** and pushes it to **AWS ECR**.
4. GitHub Actions securely SSHes into my **AWS EC2** instance.
5. My **K3s cluster** detects the new image and triggers a zero-downtime rolling update to seamlessly refresh the live pods.

---

## ✨ Features I Built In

- **JWT User Authentication:** No public access. Users must register and log in to get a secure Bearer token before they can touch the task data.
- **Data Persistence:** Tasks aren't lost when containers restart. I configured SQLite to save data permanently on the host.
- **Self-Healing Infrastructure:** I hooked up custom Kubernetes `liveness` and `readiness` probes directly to a `/health` endpoint. If the Python app freezes up or crashes, Kubernetes catches it instantly and replaces the container.
- **Zero-Downtime Deployments:** Kubernetes updates my containers one by one, keeping the live API online without interruption.

---

## 💡 The Cost Optimization Trick: Why K3s?

Standard **AWS EKS** runs a bill of about **\$72/month** just to keep the control plane running, which would burn through free credits in weeks. 

To bypass this, I chose **K3s** by Rancher. K3s strips out heavy cloud-provider-specific add-ons from standard Kubernetes, drastically lowering the memory footprint. This allowed me to pack a fully operational Kubernetes master and worker node onto a single **t3.micro (1 vCPU, 1GB RAM)** instance without hitting performance bottlenecks, keeping the entire cluster 100% Free Tier compliant.

---

## 📂 Project Structure

```text
├── .github/workflows/
│   └── deploy.yml          # GitHub Actions deployment workflow
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment & health probes
│   └── service.yaml        # Kubernetes Service (NodePort setup)
├── app.py                  # Flask API with authentication & health logic
├── Dockerfile              # Docker container build script
├── requirements.txt        # App dependencies
└── README.md               # You are looking at it
```

---

## 🔧 Local Setup

### Prerequisites
- Docker installed locally
- Python 3.9+

### Running Locally with Docker
If you want to spin up the application on your machine:
```bash
git clone https://github.com
cd cloudtasker

# Build the Docker image
docker build -t cloudtasker:local .

# Run the container
docker run -p 5000:5000 cloudtasker:local
```
The local API server will be live at `http://localhost:5000`.

### Manual Cluster Deployment
To manually push configuration updates directly to the remote K3s cluster node:
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

---

## 🛡️ Next Steps for Scaling

This architecture is optimized for low-cost learning, but to scale this out for an enterprise environment, my roadmap includes:
- [ ] **External Database:** Moving from SQLite to a managed AWS RDS PostgreSQL instance.
- [ ] **Persistent Volumes:** Implementing AWS EBS CSI drivers to decouple storage lifecycles completely from the host.
- [ ] **Ingress Routing:** Replacing the `NodePort` mapping with an Nginx Ingress Controller linked to an AWS Application Load Balancer (ALB) and Route 53 for managed SSL/TLS termination.
