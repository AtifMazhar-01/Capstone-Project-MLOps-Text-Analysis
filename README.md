# End-to-End MLOps: Text Sentiment Analysis

An end-to-end Machine Learning Operations (MLOps) project for binary sentiment analysis, covering data and pipeline versioning, model training, experiment tracking, model registration, automated CI/CD, containerization, deployment on AWS EKS, and monitoring with Prometheus and Grafana.

The project demonstrates how a machine learning model can move beyond experimentation into a structured, reproducible, and deployable ML workflow.

<p align="center">
  <a href="https://github.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis">
    <img src="https://img.shields.io/badge/GitHub-Repository-181717?style=for-the-badge&logo=github" alt="GitHub Repository">
  </a>
  <a href="https://dagshub.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.mlflow">
    <img src="https://img.shields.io/badge/MLflow-DagsHub-blue?style=for-the-badge&logo=mlflow" alt="MLflow on DagsHub">
  </a>
  <img src="https://img.shields.io/badge/Python-3.10-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.10">
  <img src="https://img.shields.io/badge/AWS-EKS%20%7C%20ECR%20%7C%20S3-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS">
</p>

---

## Table of Contents

* [Project Overview](#project-overview)
* [Objectives](#objectives)
* [Key Features](#key-features)
* [System Architecture](#system-architecture)
* [Machine Learning Pipeline](#machine-learning-pipeline)
* [Model Development](#model-development)
* [Experiment Tracking and Model Registry](#experiment-tracking-and-model-registry)
* [Data and Pipeline Versioning](#data-and-pipeline-versioning)
* [Project Structure](#project-structure)
* [Technology Stack](#technology-stack)
* [Getting Started](#getting-started)
* [Running the Application](#running-the-application)
* [Docker](#docker)
* [CI/CD with GitHub Actions](#cicd-with-github-actions)
* [Deployment on AWS EKS](#deployment-on-aws-eks)
* [Monitoring and Observability](#monitoring-and-observability)
* [Testing](#testing)
* [Model Evaluation](#model-evaluation)
* [Security and Cost Management](#security-and-cost-management)
* [Future Improvements](#future-improvements)
* [Author](#author)

---

## Project Overview

Sentiment analysis is a Natural Language Processing (NLP) task that identifies the sentiment expressed in text. It can help analyze customer reviews and distinguish positive feedback from negative feedback.

This project implements a binary sentiment classification pipeline using Python and Scikit-learn. Logistic Regression is used as the classification algorithm, with Bag-of-Words (BoW) as the feature representation in the production DVC pipeline.

In addition to model development, the project applies MLOps practices to manage the machine learning lifecycle, including:

* Data ingestion and preprocessing
* Feature engineering
* Model training and evaluation
* Experiment tracking with MLflow
* Remote tracking through DagsHub
* Data and pipeline versioning with DVC
* Model registration and promotion
* Flask-based model serving
* Automated testing and CI/CD
* Docker containerization
* Kubernetes deployment on AWS EKS
* Monitoring with Prometheus and Grafana

The objective is to build a structured workflow that makes model development, deployment, and maintenance more manageable and reproducible.

## Objectives

1. Develop a binary sentiment classifier for positive and negative text reviews.
2. Implement text preprocessing and feature extraction.
3. Train a Logistic Regression model using Scikit-learn.
4. Track experiments, parameters, and evaluation metrics with MLflow.
5. Manage datasets and pipeline stages using DVC.
6. Register the trained model and manage its lifecycle.
7. Serve predictions through a Flask application.
8. Automate testing and deployment using GitHub Actions.
9. Containerize the application using Docker.
10. Deploy the application to AWS EKS using Amazon ECR.
11. Monitor application metrics using Prometheus and Grafana.

---

## Key Features

| Component           | Implementation                                         |
| ------------------- | ------------------------------------------------------ |
| Sentiment Analysis  | Binary classification of positive and negative reviews |
| Feature Engineering | Bag-of-Words                                           |
| Classification      | Logistic Regression                                    |
| Experiment Tracking | MLflow with DagsHub                                    |
| Model Registry      | MLflow Model Registry                                  |
| Pipeline Management | DVC                                                    |
| Data Storage        | Local data and S3-backed DVC remote                    |
| Model Serving       | Flask and Gunicorn                                     |
| CI/CD               | GitHub Actions                                         |
| Containerization    | Docker                                                 |
| Container Registry  | Amazon ECR                                             |
| Orchestration       | Amazon EKS and Kubernetes                              |
| Monitoring          | Prometheus and Grafana                                 |
| Testing             | Python unit tests                                      |

---

## System Architecture

The project connects the machine learning development workflow with automated deployment and operational monitoring.

```text
                         MACHINE LEARNING WORKFLOW

  Raw Dataset
      |
      v
  DVC Pipeline
      |
      +--> Data Ingestion
      |
      +--> Data Preprocessing
      |
      +--> Feature Engineering (Bag-of-Words)
      |
      +--> Model Building
      |
      +--> Model Evaluation
      |
      v
  MLflow / DagsHub
      |
      v
  Model Registry
      |
      v
  Staging Model
      |
      v
  GitHub Actions CI/CD
      |
      +--> Automated Tests
      |
      +--> Model Promotion
      |
      +--> Docker Build
      |
      +--> Push Image to Amazon ECR
      |
      +--> Deploy to Amazon EKS
                         |
                         v
                   Flask Application
                         |
                         v
                  Prometheus Metrics
                         |
                         v
                       Grafana
```

### Architecture Components

* **DVC:** Defines and reproduces the machine learning pipeline.
* **MLflow:** Tracks experiments and manages model artifacts and versions.
* **DagsHub:** Provides remote MLflow experiment tracking.
* **GitHub Actions:** Automates testing, model promotion, image publishing, and deployment.
* **Amazon ECR:** Stores Docker images.
* **Amazon EKS:** Runs the containerized Flask application on Kubernetes.
* **Prometheus:** Collects application metrics.
* **Grafana:** Visualizes metrics through dashboards.

---

## Machine Learning Pipeline

The machine learning pipeline is organized into modular stages defined in `dvc.yaml`, with configurable parameters stored in `params.yaml`.

### Pipeline Stages

| Stage               | Script                                | Output                                    |
| ------------------- | ------------------------------------- | ----------------------------------------- |
| Data Ingestion      | `src/data/data_ingestion.py`          | `data/raw`                                |
| Data Preprocessing  | `src/data/data_preprocessing.py`      | `data/interim`                            |
| Feature Engineering | `src/features/feature_engineering.py` | `data/processed`, `models/vectorizer.pkl` |
| Model Building      | `src/model/model_building.py`         | `models/model.pkl`                        |
| Model Evaluation    | `src/model/model_evaluation.py`       | `reports/metrics.json`, MLflow run        |
| Model Registration  | `src/model/register_model.py`         | Model registered in Staging               |

### 1. Data Ingestion

The data ingestion stage loads the source dataset and prepares the data for subsequent pipeline stages.

The documented configuration uses a 75/25 train-test split.

### 2. Data Preprocessing

The preprocessing stage prepares the text reviews for feature extraction. The project uses NLP preprocessing tools, including NLTK.

### 3. Feature Engineering

The production pipeline uses Bag-of-Words to convert text into numerical features.

The feature extraction stage also saves the fitted vectorizer as `models/vectorizer.pkl`.

### 4. Model Building

The model-building stage trains a Logistic Regression classifier using the configured parameters.

The trained model is saved as `models/model.pkl`.

### 5. Model Evaluation

The evaluation stage calculates classification metrics and records the results in `reports/metrics.json`. Experiment information is also logged to MLflow.

### 6. Model Registration

The registration stage registers the trained model in the MLflow Model Registry, with the documented workflow registering the model to the Staging stage.

This separates model training from the model lifecycle and deployment workflow.

### Pipeline Configuration

The documented default parameters in `params.yaml` are:

```yaml
data_ingestion:
  test_size: 0.25

feature_engineering:
  max_features: 50
```

The production model configuration is:

| Parameter              | Value               |
| ---------------------- | ------------------- |
| Algorithm              | Logistic Regression |
| `C`                    | `1`                 |
| Solver                 | `liblinear`         |
| Penalty                | `l2`                |
| Feature representation | Bag-of-Words        |
| Maximum features       | `50`                |
| Test size              | `25%`               |

The pipeline is designed to keep data processing, feature extraction, model training, evaluation, and registration as separate stages.

---

## Model Development

### Problem Statement

Given a text review, predict whether its sentiment is positive or negative.

This is a supervised binary classification problem.

### Label Encoding

| Sentiment | Label |
| --------- | ----: |
| Positive  |     1 |
| Negative  |     0 |

### Feature Representation: Bag-of-Words

Bag-of-Words represents a document using the frequency or presence of words from a vocabulary.

The text is converted into a numerical feature vector that can be used by a machine learning classifier.

The production pipeline uses a configured maximum vocabulary size of 50 features.

### Classification: Logistic Regression

Logistic Regression is used to predict the sentiment class from the extracted text features.

The model configuration documented for the production pipeline is:

```python
LogisticRegression(
    C=1,
    solver="liblinear",
    penalty="l2"
)
```

The project also explores alternative text feature representations, including TF-IDF, in its notebooks and experiments.

### Experiments: BoW vs. TF-IDF

The notebooks and experiment scripts explore alternative feature representations and Logistic Regression configurations before the production pipeline is defined.

* **Bag-of-Words:** Represents text using word-based features.
* **TF-IDF:** Weights words according to their importance within and across documents.

The production DVC pipeline uses Bag-of-Words; TF-IDF is part of the experimentation workflow.

---

## Experiment Tracking and Model Registry

### MLflow

MLflow is used to track machine learning experiments and organize model artifacts and evaluation results.

The project records evaluation metrics including:

* Accuracy
* Precision
* Recall
* AUC

MLflow provides a centralized record of experiment runs and supports comparison between model versions.

### DagsHub Integration

DagsHub is used as the remote MLflow tracking platform.

**Experiment Tracking UI:**

[Open MLflow Experiments on DagsHub](https://dagshub.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.mlflow)

**Tracking URI:**

```text
https://dagshub.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.mlflow
```

Authentication is configured using the `CAPSTONE_TEST` environment variable in the documented setup.

### Model Registry

The model registration stage stores the trained model in the MLflow Model Registry.

The documented workflow registers the model to Staging. The CI/CD workflow includes a promotion step from Staging to Production.

This provides a structured way to manage model versions and connect model development with deployment.

---

## Data and Pipeline Versioning

### DVC

DVC is used to define the pipeline, track dependencies, and manage data and artifact versions.

The main configuration files are:

| File          | Purpose                                  |
| ------------- | ---------------------------------------- |
| `dvc.yaml`    | Defines pipeline stages and dependencies |
| `params.yaml` | Stores configurable pipeline parameters  |
| `.dvc/`       | Stores DVC configuration and metadata    |

The documented setup includes an S3-backed DVC remote for shared artifacts.

### Reproducing the Pipeline

Run the pipeline from the repository root:

```bash
dvc repro
```

Check the current DVC state:

```bash
dvc status
```

DVC can determine which stages need to be rerun based on changes to the pipeline, dependencies, and parameters.

### Remote Storage

The project documentation includes both a local development remote and an S3 remote.

Example S3 remote configuration:

```bash
dvc remote add -d myremote s3://<your-bucket-name>
```

Configure AWS credentials before accessing the remote:

```bash
aws configure
```

Push tracked data and artifacts:

```bash
dvc push
```

Replace `<your-bucket-name>` with your own bucket. Configure the remote only once unless you need to change it.

---

## Project Structure

```text
Capstone-Project-MLOps-Text-Analysis/
│
├── .github/
│   └── workflows/
│       └── ci.yaml
│
├── data/
│   ├── raw/
│   ├── interim/
│   └── processed/
│
├── models/
│   ├── model.pkl
│   └── vectorizer.pkl
│
├── notebooks/
│   └──                 # Experiments: BoW vs. TF-IDF
│
├── reports/
│   ├── metrics.json
│   └── experiment_info.json
│
├── flask_app/
│   ├── app.py
│   └── templates/
│
├── src/
│   ├── data/
│   │   ├── data_ingestion.py
│   │   └── data_preprocessing.py
│   │
│   ├── features/
│   │   └── feature_engineering.py
│   │
│   ├── model/
│   │   ├── model_building.py
│   │   ├── model_evaluation.py
│   │   └── register_model.py
│   │
│   ├── connections/
│   └── logger/
│
├── scripts/
│   └── promote_model.py
│
├── tests/
│   ├── test_model.py
│   └── test_flask_app.py
│
├── dvc.yaml
├── params.yaml
├── deployment.yaml
├── Dockerfile
├── requirements.txt
└── README.md
```

This structure reflects the project layout described in the project documentation. Some directories may contain additional files in the actual repository.

---

## Technology Stack

| Category                     | Technologies           |
| ---------------------------- | ---------------------- |
| Programming Language         | Python 3.10            |
| Data Processing              | Pandas, NumPy          |
| NLP                          | NLTK                   |
| Machine Learning             | Scikit-learn           |
| Experiment Tracking          | MLflow, DagsHub        |
| Data and Pipeline Versioning | DVC                    |
| Artifact Storage             | Amazon S3              |
| Application                  | Flask, Gunicorn        |
| Containerization             | Docker                 |
| CI/CD                        | GitHub Actions         |
| Container Registry           | Amazon ECR             |
| Orchestration                | Kubernetes, Amazon EKS |
| Cloud Infrastructure         | AWS EC2, S3, ECR, EKS  |
| Monitoring                   | Prometheus             |
| Visualization                | Grafana                |
| Testing                      | Python `unittest`      |

---

## Getting Started

### Prerequisites

Install the following tools as needed:

* Git
* Python 3.10
* Conda
* DVC
* Docker
* AWS CLI
* kubectl
* eksctl

For remote experiment tracking, configure DagsHub authentication. For DVC remote storage and AWS deployment, configure the required AWS credentials and permissions.

### 1. Clone the Repository

```bash
git clone https://github.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.git

cd Capstone-Project-MLOps-Text-Analysis
```

### 2. Create the Conda Environment

```bash
conda create -n atlas python=3.10 -y
conda activate atlas
```

If the `atlas` environment already exists, activate it instead.

### 3. Install Dependencies

```bash
python -m pip install --upgrade pip

pip install -r requirements.txt

pip install -e .
```

The editable installation makes the project package available in the active Python environment.

### 4. Configure DagsHub Authentication

Create a DagsHub access token and store it securely.

#### Linux / macOS

```bash
export CAPSTONE_TEST="<your-dagshub-token>"
```

#### Windows PowerShell

```powershell
$env:CAPSTONE_TEST = "<your-dagshub-token>"
```

Replace the placeholder with your token. Never commit the token to Git or include it in screenshots, logs, or public documentation.

### 5. Run the DVC Pipeline

From the repository root:

```bash
dvc repro
```

Check the pipeline state:

```bash
dvc status
```

If the required data is stored remotely, configure the DVC remote and retrieve the data before running the pipeline.

---

## Running the Application

The Flask application is located in the `flask_app/` directory.

### Start the Flask App

```bash
cd flask_app

python app.py
```

Ensure that the required model and vectorizer artifacts are available and that the required environment variables are configured.

The application is configured to run locally on port `5000`.

Open:

http://localhost:5000

### Prometheus Metrics Endpoint

The Flask application exposes a metrics endpoint:

http://localhost:5000/metrics

The documented custom metrics include request count, request latency, and prediction class counts.

---

## Docker

Docker packages the Flask application and its dependencies into a container image.

The documented image runs Gunicorn on port `5000` inside the container.

### Build the Docker Image

Run this command from the repository root:

```bash
docker build -t capstone-app:latest .
```

### Run the Container

```bash
docker run -p 8888:5000 \
  -e CAPSTONE_TEST="<your-dagshub-token>" \
  capstone-app:latest
```

For Windows PowerShell, use a single line or PowerShell line continuations as appropriate.

The application is accessible at:

http://localhost:8888

The metrics endpoint is:

http://localhost:8888/metrics

---

## CI/CD with GitHub Actions

The project uses GitHub Actions to automate the model and application delivery workflow.

**Workflow file:** `.github/workflows/ci.yaml`

The documented workflow runs on every push.

### CI/CD Workflow

1. Install project dependencies.
2. Execute the DVC pipeline.
3. Run model tests.
4. Promote the model from Staging to Production.
5. Run Flask application tests.
6. Build the Docker image.
7. Push the image to Amazon ECR.
8. Create or update the Kubernetes secret.
9. Apply the Kubernetes deployment configuration to Amazon EKS.

This workflow connects model development, testing, container publishing, and deployment.

### Required GitHub Secrets

Configure the following repository secrets in GitHub:

| Secret                  | Purpose                                 |
| ----------------------- | --------------------------------------- |
| `CAPSTONE_TEST`         | DagsHub token for MLflow authentication |
| `AWS_ACCESS_KEY_ID`     | AWS access key                          |
| `AWS_SECRET_ACCESS_KEY` | AWS secret access key                   |
| `AWS_REGION`            | AWS deployment region                   |
| `AWS_ACCOUNT_ID`        | AWS account ID                          |
| `ECR_REPOSITORY`        | Amazon ECR repository name              |

The documented example uses `us-east-1` as the AWS region and `flask-app` as an example ECR repository name.

### AWS Permissions

The AWS identity used by the workflow needs permissions for the relevant services, including:

* Amazon ECR
* Amazon EKS
* Amazon S3, if accessing the DVC remote
* Related deployment resources

Use least-privilege IAM permissions wherever possible rather than granting broad access unnecessarily.

---

## Deployment on AWS EKS

The project documentation describes deploying the containerized Flask application to Amazon EKS.

### Prerequisites

Install the AWS CLI, kubectl, and eksctl, and ensure they are available on your system's `PATH`.

Verify the installations:

```bash
aws --version

kubectl version --client

eksctl version
```

On Windows, the documented setup recommends using the AWS CLI MSI installer to avoid potential PATH conflicts with a pip-installed AWS CLI.

### 1. Configure AWS Credentials

```bash
aws configure
```

Select the appropriate AWS region and configure credentials with the required permissions.

### 2. Create the EKS Cluster

The following is the cluster configuration documented for this project:

```bash
eksctl create cluster \
  --name flask-app-cluster \
  --region us-east-1 \
  --nodegroup-name flask-app-nodes \
  --node-type t3.small \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --managed
```

Cluster creation can take several minutes and may incur AWS charges.

### 3. Configure kubectl

```bash
aws eks --region us-east-1 update-kubeconfig \
  --name flask-app-cluster
```

Verify the cluster connection:

```bash
kubectl get nodes
```

### 4. Deploy the Application

The Kubernetes configuration is defined in `deployment.yaml`.

The documented configuration includes:

* A Deployment named `flask-app`
* Two application replicas
* An image stored in Amazon ECR
* Environment variables from the `capstone-secret` Kubernetes Secret
* A LoadBalancer Service named `flask-app-service`
* Service port `5000`

Apply the deployment:

```bash
kubectl apply -f deployment.yaml
```

Check the running pods:

```bash
kubectl get pods
```

Check the service:

```bash
kubectl get svc flask-app-service
```

When the LoadBalancer has been provisioned, obtain its external address from the service output.

The application endpoint follows this format:

```text
http://<EXTERNAL-IP>:5000
```

Replace `<EXTERNAL-IP>` with the actual address assigned to the service.

Ensure the relevant security groups and network rules permit only the access required for the application.

---

## Monitoring and Observability

The project includes application metrics collection using Prometheus and dashboard visualization using Grafana.

The documented monitoring setup uses separate EC2 instances for Prometheus and Grafana.

### Prometheus

Prometheus collects metrics from the Flask application's `/metrics` endpoint.

The documented setup uses:

* Ubuntu EC2 instance
* Instance type: `t3.medium`
* Approximately 20 GB storage
* Prometheus web interface on port `9090`
* Scrape interval of 15 seconds

Example Prometheus configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "flask-app"
    static_configs:
      - targets: ["<ELB-DNS-OR-IP>:5000"]
```

Replace `<ELB-DNS-OR-IP>` with the actual LoadBalancer DNS name or address reachable from the Prometheus instance.

Start Prometheus with the configured file and access it at:

```text
http://<PROMETHEUS-EC2-IP>:9090
```

### Application Metrics

The documented Flask metrics include:

* Request count
* Request latency
* Prediction class counts

These metrics provide visibility into application traffic, response times, and the distribution of predictions.

### Grafana

Grafana provides dashboards for visualizing the collected metrics.

The documented setup uses an Ubuntu EC2 instance and the Grafana web interface on port `3000`.

Access Grafana at:

```text
http://<GRAFANA-EC2-IP>:3000
```

### Configure the Prometheus Data Source

1. Open the Grafana interface.

2. Add a Prometheus data source.

3. Set the Prometheus server URL:

   ```text
   http://<PROMETHEUS-EC2-IP>:9090
   ```

4. Save and verify the data source.

5. Build dashboards using the available application metrics.

Restrict access to monitoring services and configure authentication before exposing them beyond a trusted network.

---

## Testing

The project includes unit tests for the model and Flask application.

### Run Model Tests

From the repository root:

```bash
python -m unittest tests/test_model.py
```

### Run Flask Application Tests

```bash
python -m unittest tests/test_flask_app.py
```

The documented model tests cover:

* Loading a model from the MLflow Registry
* Checking the prediction signature
* Evaluating holdout performance against configured thresholds

The Flask tests validate application behavior.

Ensure that the required environment variables, model artifacts, and MLflow access are configured before running the tests.

---

## Model Evaluation

The documented pipeline logs the following evaluation metrics:

| Metric    | Description                                                               |
| --------- | ------------------------------------------------------------------------- |
| Accuracy  | Overall proportion of correct predictions                                 |
| Precision | Proportion of predicted positive cases that are correct                   |
| Recall    | Proportion of actual positive cases correctly identified                  |
| AUC       | Area under the ROC curve, measuring ranking performance across thresholds |

The evaluation results are saved in `reports/metrics.json` and logged to MLflow.

### Experiment Results

The actual numeric results should be taken from the final evaluation report or MLflow run.

| Metric    | Result                          |
| --------- | ------------------------------- |
| Accuracy  | Refer to `reports/metrics.json` |
| Precision | Refer to `reports/metrics.json` |
| Recall    | Refer to `reports/metrics.json` |
| AUC       | Refer to `reports/metrics.json` |

The repository documentation does not specify the final metric values, so they are intentionally not fabricated here.

---

## Security and Cost Management

### Secrets Management

* Never commit DagsHub tokens or AWS credentials.
* Use GitHub Actions secrets for CI/CD credentials.
* Use Kubernetes Secrets for application configuration where appropriate.
* Restrict IAM permissions to the services and operations required.
* Avoid exposing sensitive endpoints or credentials in logs.

### AWS Cost Management

AWS resources used by this project may incur charges, including:

* EKS clusters and worker nodes
* EC2 instances
* Load balancers
* ECR image storage
* S3 storage and requests
* Data transfer

Delete resources that are no longer required and verify that related resources have been removed.

### Delete the Kubernetes Resources

```bash
kubectl delete deployment flask-app

kubectl delete service flask-app-service

kubectl delete secret capstone-secret
```

### Delete the EKS Cluster

```bash
eksctl delete cluster \
  --name flask-app-cluster \
  --region us-east-1
```

Check whether the cluster still exists:

```bash
eksctl get cluster --region us-east-1
```

Also review and remove unused ECR images, S3 artifacts, Prometheus and Grafana EC2 instances, and related CloudFormation stacks where appropriate.

---

## Future Improvements

Potential areas for further development include:

* Expand the feature-engineering experiments with different BoW and TF-IDF configurations.
* Evaluate additional classification algorithms.
* Improve model evaluation and experiment comparison.
* Add stronger data validation and input checks.
* Expand automated tests for data processing and inference.
* Improve model promotion by ensuring all required validation passes before Production deployment.
* Add deployment health checks and rollback procedures.
* Introduce stricter IAM policies and network access controls.
* Add alerting for application availability and latency.
* Monitor prediction distributions and model quality over time.
* Document reproducible experiment results and model versions.

---

## Author

**Atif Mazhar**

Computer Engineering | Machine Learning | Data Science | MLOps

* GitHub: [AtifMazhar-01](https://github.com/AtifMazhar-01)
* Project Repository: [Capstone Project - MLOps Text Analysis](https://github.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis)
* MLflow Experiments: [DagsHub Tracking UI](https://dagshub.com/AtifMazhar-01/Capstone-Project-MLOps-Text-Analysis.mlflow)

---

## License

The project documentation specifies the MIT License.

See the [LICENSE](LICENSE) file for the license terms.
