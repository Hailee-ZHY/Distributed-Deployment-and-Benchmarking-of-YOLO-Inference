# Distributed Deployment & Benchmarking of YOLO Inference

![System Architecture](./Solution%20Design.png)

This project implements a scalable, containerized computer vision service using a fine-tuned **YOLO model**. The primary objective was to engineer and benchmark two distinct cloud architectures—**Managed Kubernetes (GKE)** and **Serverless (Cloud Run)**—to analyze the trade-offs between performance consistency, resource efficiency, and elasticity under high-concurrency workloads.

## System Architecture & Design
The system is built on a unified Dockerized inference engine that exposes a RESTful API. The design explores two scaling philosophies:
- **Kubernetes (GKE):** Utilizes a Horizontal Pod Autoscaler (HPA) and a Load Balancer for proactive scaling and low-latency steady-state processing.
- **Serverless (Cloud Run):** Implements reactive scaling with a scale-to-zero configuration to optimize cost during idle periods.

## Tech Stack
* **Model:** YOLO (Object Detection) fine-tuned on COCO128.
* **Inference Engine:** Python 3.10, PyTorch, OpenCV.
* **API Layer:** Flask, Gunicorn (1 worker, 2 threads for optimized concurrency).
* **Infrastructure:** Google Cloud Platform (GKE, Cloud Run, Artifact Registry).
* **Observability:** Grafana, Google Cloud Monitoring (MQL).
* **Testing:** Apache JMeter, Asynchronous Python (`aiohttp`).

## Performance Benchmarking Results
We conducted stress tests simulating **10 requests/second** for 30-second intervals to evaluate how each architecture handled resource pressure.

| Metric | Cloud Run (Serverless) | GKE (Kubernetes) | Technical Insight |
| :--- | :---: | :---: | :--- |
| **Startup Latency** | **>700ms (Cold Start)** | **<50ms (Consistent)** | GKE’s warm-pod strategy eliminates the initialization delay inherent in serverless scaling. |
| **Avg. Memory Util.** | **0.56** | **0.48** | GKE demonstrated 14% higher memory efficiency during steady-state loads due to pre-defined resource requests. |
| **CPU Profile** | Reactive Spikes | Stable Baseline | Cloud Run spikes during instance spin-up; GKE maintains a stable utilization profile. |
| **Scaling Strategy** | Elastic/Reactive | Predictable/Static | GKE prioritizes performance; Cloud Run prioritizes cost-efficiency. |

## Monitoring & Observability
The project features a full-stack observability dashboard built in **Grafana**. 
* **Metrics Collection:** Leverages Google Cloud Monitoring via an IAM-authenticated service account.
* **Custom Queries:** Utilizes **Monitoring Query Language (MQL)** to aggregate real-time CPU and Memory utilization across distributed pods.
* **Health Tracking:** Monitors 2xx/4xx/5xx HTTP response codes to ensure service reliability during autoscaling events.

## 📂 Repository Structure
```text
├── src/                # Inference logic and Flask API implementation
├── k8s/                # Kubernetes Deployment, Service, and HPA manifests
├── benchmarks/         # JMeter test plans and aiohttp load testing scripts
├── Dockerfile          # Multi-stage build for the YOLO inference environment
└── Solution Design.png # High-level architectural diagram