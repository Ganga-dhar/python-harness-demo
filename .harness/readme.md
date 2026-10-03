# 🚀 Secure CI/CD Pipeline with Harness, Docker, AWS OIDC & Amazon ECR

A real-time enterprise CI/CD implementation for a containerized Python application using **Harness, Docker, AWS IAM, AWS OIDC, and Amazon ECR**.

The objective is to establish a secure and automated delivery process that validates application changes before building and publishing container images, while avoiding long-lived AWS credentials.

---

## 🏗️ Architecture / CI/CD Flow

```text
                    ┌─────────────────┐
                    │     GitHub      │
                    │ Source Code     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Harness CI   │
                    │ Pipeline        │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
       Python Environment             Dependency Install
              │
              ▼
       ┌───────────────┐
       │    Flake8     │
       │ Code Quality  │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │     Pytest    │
       │ Automated Test│
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │ Docker Build  │
       │ Container Img │
       └───────┬───────┘
               │
               ▼
       ┌────────────────┐
       │   AWS OIDC     │
       │ Authentication │
       └───────┬────────┘
               │
               ▼
       ┌────────────────┐
       │    AWS IAM     │
       │ Role Assumption│
       └───────┬────────┘
               │
               ▼
       ┌────────────────┐
       │ Amazon ECR     │
       │ Image Registry │
       └───────┬────────┘
               │
               ▼
       ┌────────────────┐
       │ Docker Image   │
       │     Push       │
       └────────────────┘
🔄 CI/CD Pipeline Flow
GitHub
   ↓
Harness CI
   ↓
Source Code Checkout
   ↓
Python Environment Setup
   ↓
Dependency Installation
   ↓
Flake8 Code Quality Validation
   ↓
Pytest Automated Testing
   ↓
Docker Image Build
   ↓
AWS OIDC Authentication
   ↓
IAM Role Assumption
   ↓
Amazon ECR Authentication
   ↓
Docker Image Push
📌 Project Objectives

The main objectives of this implementation are:

Automate the application CI/CD workflow
Integrate GitHub with Harness
Validate Python code automatically
Execute automated unit tests
Build a Docker container image
Authenticate Harness with AWS using OIDC
Avoid storing long-lived AWS credentials
Push validated Docker images to Amazon ECR
Implement least-privilege IAM access
Create a repeatable and auditable delivery process

📁 Project Structure
python-harness-demo/
│
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
└── .gitignore
File	Description
app.py	Flask application
test_app.py	Pytest automated tests
requirements.txt	Python dependencies
Dockerfile	Docker image definition
.gitignore	Git ignored files
🧪 Harness CI Pipeline

The Harness CI pipeline performs multiple validation stages before publishing the container image.

1. Source Code Checkout

Harness connects to GitHub and checks out the required source code.

GitHub Repository
       ↓
Harness CI
       ↓
Source Checkout
2. Python Environment Setup

A Python virtual environment is created inside the Harness build environment.

python3 -m venv .venv

The environment is activated:

. .venv/bin/activate
3. Dependency Installation

Dependencies are installed from requirements.txt.

python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
🔍 Code Quality Validation

Flake8 is used for Python static code analysis.

flake8 . --exclude=venv,.venv

Flake8 helps identify:

Syntax errors
Undefined variables
Unused imports
Code quality violations
Formatting issues

If the linting stage fails, the pipeline stops and the Docker image is not published.

🧪 Automated Testing

Pytest is used to execute application tests.

pytest -v

The tests validate:

HTTP response status
Application endpoint behavior
Health endpoint response
Expected application output

Only successfully validated code should proceed to the Docker build and publishing stages.

🐳 Docker Containerization

After successful code validation and testing, the application is packaged into a Docker image.

Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
Docker Build Flow
Python Application
       ↓
Dockerfile
       ↓
Docker Build
       ↓
Docker Image
       ↓
Image Tag
       ↓
Amazon ECR

Example:

python-harness-demo:latest
🔐 AWS OIDC Integration

A key security component of this project is integrating Harness with AWS using OpenID Connect (OIDC).

Instead of storing long-lived AWS credentials inside Harness:

❌ AWS Access Key
❌ AWS Secret Access Key

the pipeline uses OIDC federation to obtain temporary AWS credentials.

Authentication Flow
                 Harness CI
                     │
                     │ OIDC Token
                     ▼
             AWS IAM OIDC Provider
                     │
                     │ AssumeRole
                     ▼
                AWS IAM Role
                     │
                     │ Temporary Credentials
                     ▼
                AWS Services

🚦 Pipeline Quality Gates

The pipeline follows a fail-fast approach.

Source Checkout
       │
       ▼
Dependency Installation
       │
       ▼
Flake8
       │
       ├── ❌ Failure → Stop Pipeline
       │
       ▼
Pytest
       │
       ├── ❌ Failure → Stop Pipeline
       │
       ▼
Docker Build
       │
       ├── ❌ Failure → Stop Pipeline
       │
       ▼
AWS OIDC
       │
       ├── ❌ Failure → Stop Pipeline
       │
       ▼
ECR Authentication
       │
       ├── ❌ Failure → Stop Pipeline
       │
       ▼
Docker Image Push

This ensures that only successfully validated application changes proceed toward container publishing.