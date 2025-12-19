# Pixtral 12B Deployment on Vertex AI

This document provides a detailed, step-by-step explanation of how to deploy the **mistral-community/pixtral-12b** model from scratch under the deployment project, within the `europe-west4` region, using the `pixtral-run` naming convention. The project is implemented using Google Cloud Vertex AI.

This guide is designed to be followed sequentially, similar to "Kubernetes The Hard Way", ensuring you understand each step of the deployment process.

## Target Audience

The target audience for this tutorial is engineers and developers who want to understand the fundamentals of deploying Large Language Models (LLMs) on Vertex AI using custom containers and GPUs.

## Labs

This tutorial is broken down into the following sections:

1.  [Prerequisites](docs/01-prerequisites.md)
2.  [VM Setup & Folder Structure](docs/02-vm-setup.md)
3.  [Creating Application Files](docs/03-application-setup.md)
4.  [Building the Container](docs/04-container-build.md)
5.  [Publishing & Deploying the Model](docs/05-deploy-model.md)
6.  [Testing & Verification](docs/06-verification.md)
7.  [Monitoring](docs/07-monitoring.md)

## Deployment Overview:

We will be deploying the `mistral-community/pixtral-12b` model.
The deployment process involves:
*   Setting up a GPU-enabled VM for development.
*   Creating a Python Flask application (`predictor.py`) to serve the model.
*   Containerizing the application with Docker.
*   Pushing the container to Google Artifact Registry.
*   Deploying the container to a Vertex AI Endpoint with A100 GPU acceleration.
*   Verifying the deployment with a custom test script.
*   Monitoring the deployment via Google Cloud Monitoring.

---
*Generated for the Pixtral 12B Deployment Project.*
