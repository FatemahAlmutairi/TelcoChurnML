# Telco Churn – End-to-End ML Project

Predict which telecom customers are likely to churn — so teams can act before they leave.

## Quick Start

Run the container:

```bash
docker pull fatemahalmutairi/telco-fastapi:latest
docker run -p 8000:8000 fatemahalmutairi/telco-fastapi:latest

Then access:

Gradio UI: http://localhost:8000/ui
API endpoint: POST to /predict with customer data
Health check: GET /health


## Project Structure

├── src/
│   ├── app.py              # FastAPI app with endpoints
│   ├── inference.py        # Model loading and prediction logic
│   └── model.pkl           # Serialized XGBoost model
├── train/
│   ├── train.py            # Feature engineering + XGBoost training
│   └── mlflow_tracking.py  # MLflow experiment logging
├── Dockerfile              # Container setup with uvicorn
├── requirements.txt        # Python dependencies
├── .github/workflows/
│   └── ci-cd.yml          # CI/CD pipeline to Docker Hub
└── terraform/
    ├── main.tf            # AWS ECS Fargate + ALB configuration
    └── variables.tf       # Infrastructure variables

## How It Works

Model: XGBoost classifier trained on telecom customer data, logged with MLflow
API: FastAPI serving predictions via /predict POST endpoint
UI: Gradio interface mounted at /ui for manual testing
Deployment: GitHub Actions builds image → pushes to Docker Hub → updates ECS Fargate
Networking: Application Load Balancer on port 80 forwards to ECS task on port 8000
Logging: CloudWatch captures stdout/stderr and service events
API Usage

## Prediction Endpoint
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": "12345",
    "tenure": 12,
    "monthly_charges": 75.50,
    "total_charges": 906.00,
    "contract_type": "Month-to-month",
    "internet_service": "Fiber optic",
    "payment_method": "Electronic check",
    "has_phone_service": 1,
    "has_online_security": 0
  }'
Response

json
{
  "customer_id": "12345",
  "churn_probability": 0.72,
  "churn_prediction": 1,
  "prediction_timestamp": "2026-08-17T12:34:56.789Z"
}

## Roadblocks & Fixes

Issue	Solution
Unhealthy ALB targets	Added GET / health endpoint; fixed listener/target port mismatches
Module import errors in container	Set PYTHONPATH=/app/src in Dockerfile
ALB DNS timing out	Fixed security groups: ALB allows 80 from 0.0.0.0/0; task allows 8000 from ALB SG
ECS not picking up new image	Force new deployment via CLI or console
Gradio "No runs found"	Standardized MLflow experiment name; inference loads logged model consistently
Local vs. prod paths	Local loads from ./mlruns/; container loads packaged model at build time

## Requirements

Python 3.9+
Docker
AWS account (for deployment)
MLflow
FastAPI + Gradio
Deployment

## Local Development


pip install -r requirements.txt
uvicorn src.app:app --host 0.0.0.0 --port 8000 --reload

##Docker Build


docker build -t telco-fastapi .
docker run -p 8000:8000 telco-fastapi

##AWS Deployment (Terraform)


cd terraform
terraform init
terraform plan
terraform apply
The infrastructure provisions:

ECS Fargate cluster with the container
Application Load Balancer with public endpoint
Security groups with proper ingress/egress rules
CloudWatch log group for container logs

## CI/CD Pipeline

GitHub Actions workflow:

Lints Python code
Runs unit tests
Builds Docker image
Pushes to Docker Hub (fatemahalmutairi/telco-fastapi)
Forces new ECS deployment with the updated image

## Monitoring

Application logs: CloudWatch Logs (/ecs/telco-fastapi)
Container health: ECS service events in AWS Console
Model performance: MLflow dashboard for tracking experiments
API metrics: FastAPI built-in OpenTelemetry support

## Contributing

Fork the repository
Create a feature branch
Commit your changes
Push to the branch
Open a Pull Request

## License

MIT License - see LICENSE file for details

## Acknowledgments

Built as an end-to-end ML shipping exercise. Works fine.
