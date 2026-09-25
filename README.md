# Capstone Project — MLOps Text Analysis

End-to-end MLOps pipeline for **binary sentiment analysis** on text reviews. The project covers experiment tracking, data versioning, model registry, CI/CD, containerization, Kubernetes deployment on AWS EKS, and production monitoring with Prometheus & Grafana.

**Repo:** [github.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis](https://github.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis)  
**MLflow (DagsHub):** [Experiment tracking UI](https://dagshub.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.mlflow)

---

## Highlights

| Area | What you get |
|------|----------------|
| **ML** | Bag-of-Words + Logistic Regression for positive / negative sentiment |
| **Pipeline** | Reproducible DVC stages from ingestion → registration |
| **Tracking** | MLflow experiments & model registry on DagsHub |
| **Serving** | Flask web UI + `/metrics` for Prometheus |
| **CI/CD** | GitHub Actions: tests → promote model → build & push to ECR → deploy to EKS |
| **Ops** | Docker, AWS ECR, EKS LoadBalancer, Prometheus & Grafana on EC2 |

---

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Raw data   │────▶│ DVC pipeline │────▶│ MLflow / DagsHub│
│  (CSV/S3)   │     │  train+eval  │     │  registry       │
└─────────────┘     └──────────────┘     └────────┬────────┘
                                                  │
                    ┌─────────────────────────────▼────────┐
                    │  GitHub Actions CI/CD                 │
                    │  tests → promote → ECR → EKS deploy   │
                    └─────────────────────────────┬────────┘
                                                  │
┌─────────────┐     ┌──────────────┐     ┌────────▼────────┐
│  Grafana    │◀────│  Prometheus  │◀────│  Flask on EKS   │
│  (EC2)      │     │  (EC2)       │     │  LoadBalancer   │
└─────────────┘     └──────────────┘     └─────────────────┘
```

---

## Tech stack

- **Language:** Python 3.10  
- **ML:** scikit-learn, NLTK, pandas, NumPy  
- **Pipeline & data:** DVC (+ S3 remote), YAML params  
- **Tracking:** MLflow + DagsHub  
- **API / UI:** Flask, Gunicorn  
- **Observability:** prometheus_client, Prometheus, Grafana  
- **Cloud:** AWS (S3, ECR, EKS, EC2), Docker, kubectl, eksctl  
- **CI/CD:** GitHub Actions  

---

## Project structure

```
├── .github/workflows/ci.yaml   # CI/CD: test, promote, ECR, EKS
├── data/                       # raw → interim → processed (DVC-managed)
├── models/                     # model.pkl, vectorizer.pkl
├── notebooks/                  # experiments (BoW vs TF-IDF, etc.)
├── reports/                    # metrics.json, experiment_info.json
├── flask_app/                  # Flask serving app + templates
├── src/
│   ├── data/                   # ingestion, preprocessing
│   ├── features/               # BoW feature engineering
│   ├── model/                  # train, evaluate, register
│   ├── connections/            # S3 / SSMS helpers
│   └── logger/
├── tests/                      # model + Flask unit tests
├── scripts/promote_model.py    # Staging → Production in MLflow
├── dvc.yaml                    # pipeline definition
├── params.yaml                 # hyperparameters
├── deployment.yaml             # Kubernetes Deployment + Service
├── Dockerfile
└── requirements.txt
```

---

## ML pipeline (DVC)

Configured in `dvc.yaml` and parameterized via `params.yaml`:

| Stage | Script | Output |
|-------|--------|--------|
| `data_ingestion` | `src/data/data_ingestion.py` | `data/raw` (train/test split) |
| `data_preprocessing` | `src/data/data_preprocessing.py` | `data/interim` (cleaned text) |
| `feature_engineering` | `src/features/feature_engineering.py` | `data/processed`, `models/vectorizer.pkl` |
| `model_building` | `src/model/model_building.py` | `models/model.pkl` |
| `model_evaluation` | `src/model/model_evaluation.py` | `reports/metrics.json`, MLflow run |
| `model_registration` | `src/model/register_model.py` | Model registered → **Staging** |

**Default params** (`params.yaml`):

```yaml
data_ingestion:
  test_size: 0.25
feature_engineering:
  max_features: 50
```

**Model:** Logistic Regression (`C=1`, `liblinear`, L2).  
**Task:** Binary sentiment — `positive → 1`, `negative → 0`.  
**Metrics logged:** accuracy, precision, recall, AUC.

---

## Getting started

### 1. Clone & environment

```bash
git clone https://github.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.git
cd Capstone-Project-MLOps-Text-Analysis

conda create -n atlas python=3.10 -y
conda activate atlas
pip install -r requirements.txt
pip install -e .
```

### 2. DagsHub / MLflow auth

Create a token in DagsHub (**Settings → Tokens**) and export it (never commit the token):

```bash
# Linux / macOS
export CAPSTONE_TEST="<your-dagshub-token>"

# Windows PowerShell
$env:CAPSTONE_TEST = "<your-dagshub-token>"
```

Tracking URI used by the project:

```
https://dagshub.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.mlflow
```

### 3. Run the DVC pipeline

```bash
dvc repro
dvc status
```

Optional — configure remotes:

```bash
# Local remote (dev)
dvc remote add -d mylocal local_s3

# S3 remote (shared artifacts)
dvc remote add -d myremote s3://<your-bucket-name>
aws configure   # if not already configured
dvc push
```

### 4. Run the Flask app locally

```bash
cd flask_app
# Ensure CAPSTONE_TEST is set and models/vectorizer.pkl exists
python app.py
```

Open [http://localhost:5000](http://localhost:5000).  
Prometheus metrics: [http://localhost:5000/metrics](http://localhost:5000/metrics).

---

## Docker

From the **repo root**:

```bash
docker build -t capstone-app:latest .

docker run -p 8888:5000 -e CAPSTONE_TEST="<your-dagshub-token>" capstone-app:latest
```

App: [http://localhost:8888](http://localhost:8888)  
Image runs **Gunicorn** on port `5000` inside the container.

---

## CI/CD (GitHub Actions)

Workflow: `.github/workflows/ci.yaml` (runs on every **push**).

1. Install dependencies  
2. `dvc repro`  
3. Model tests (`tests/test_model.py`)  
4. Promote Staging → Production (`scripts/promote_model.py`)  
5. Flask app tests  
6. Build Docker image → push to **AWS ECR**  
7. Create/update K8s secret → `kubectl apply -f deployment.yaml` on **EKS**

### Required GitHub secrets

| Secret | Purpose |
|--------|---------|
| `CAPSTONE_TEST` | DagsHub token (MLflow auth) |
| `AWS_ACCESS_KEY_ID` | AWS access |
| `AWS_SECRET_ACCESS_KEY` | AWS secret |
| `AWS_REGION` | e.g. `us-east-1` |
| `AWS_ACCOUNT_ID` | AWS account ID |
| `ECR_REPOSITORY` | ECR repo name (e.g. `flask-app`) |

IAM user should include **AmazonEC2ContainerRegistryFullAccess** (and EKS/S3 permissions as needed).

---

## Deploy on AWS EKS

### Prerequisites (Windows notes)

Prefer the **MSI AWS CLI** over a pip-installed `awscli` to avoid PATH conflicts. Install `kubectl` and `eksctl` and ensure they are on your `PATH`.

```bash
aws --version
kubectl version --client
eksctl version
```

### Create cluster

```bash
eksctl create cluster \
  --name flask-app-cluster \
  --region us-east-1 \
  --nodegroup-name flask-app-nodes \
  --node-type t3.small \
  --nodes 1 --nodes-min 1 --nodes-max 1 \
  --managed

aws eks --region us-east-1 update-kubeconfig --name flask-app-cluster
kubectl get nodes
```

Allow inbound **TCP 5000** on the node / LoadBalancer security group so the service is reachable.

### Kubernetes manifests

`deployment.yaml` defines:

- **Deployment** `flask-app` — 2 replicas, image from ECR, env from secret `capstone-secret`
- **Service** `flask-app-service` — `LoadBalancer` on port `5000`

After deploy:

```bash
kubectl get pods
kubectl get svc flask-app-service
# Open http://<EXTERNAL-IP>:5000
```

---

## Monitoring

### Prometheus (EC2)

1. Launch Ubuntu EC2 (`t3.medium`, ~20 GB), open ports **9090** and **22**.  
2. Install Prometheus and point scrape config at the Flask LoadBalancer:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "flask-app"
    static_configs:
      - targets: ["<ELB-DNS-OR-IP>:5000"]
```

3. Start Prometheus with that config and open `http://<prometheus-ec2-ip>:9090`.

Custom app metrics include request count, latency, and prediction class counts (`/metrics`).

### Grafana (EC2)

1. Launch Ubuntu EC2, open ports **3000** and **22**.  
2. Install Grafana, start `grafana-server`.  
3. UI: `http://<grafana-ec2-ip>:3000` (default user `admin`).  
4. Add Prometheus as a data source (`http://<prometheus-ec2-ip>:9090`) and build dashboards.

---

## Tests

```bash
# With CAPSTONE_TEST set and pipeline artifacts available
python -m unittest tests/test_model.py
python -m unittest tests/test_flask_app.py
```

Model tests cover loading from the MLflow registry, prediction signature, and holdout performance thresholds.

---

## AWS cleanup

To avoid ongoing charges:

```bash
kubectl delete deployment flask-app
kubectl delete service flask-app-service
kubectl delete secret capstone-secret
eksctl delete cluster --name flask-app-cluster --region us-east-1
eksctl get cluster --region us-east-1   # confirm gone
```

Also remove unused **ECR** images, **S3** artifacts, Prometheus/Grafana **EC2** instances, and verify related **CloudFormation** stacks are deleted (`eksctl` creates stacks for the control plane and node group).

---

## Experiments

Notebooks / scripts under `notebooks/` explore feature representations (e.g. BoW vs TF-IDF) and Logistic Regression setups before locking the production DVC stages.

---

## License

MIT © AtifMazhar-01 — see [LICENSE](LICENSE).
