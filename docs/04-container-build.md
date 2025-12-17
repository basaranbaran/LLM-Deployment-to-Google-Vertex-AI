# Building the Container

> [!NOTE]
> **Execution Context:** Virtual Machine (VM) - Terminal

This section describes how to package the prepared application and deploy it to Google Cloud Artifact Registry.

## 1. Set Environment Variables

Assign project information to variables to make the following commands more readable.

```bash
export PROJECT_ID="llm-development-XXXX"   # Replace with your project ID
export REGION="europe-west4"               # Replace with your region
export REPO_NAME="pixtral-models-repo"     # Artifact Registry repository name
export IMAGE_NAME="pixtral-run"            # Docker image name
export IMAGE_URI="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:v1"
```

## 2. Build and Push the Docker Image

Using the `Dockerfile` created in the previous step, build your container image and push it to Artifact Registry.

```bash
echo "🔨 Building Docker image ($IMAGE_NAME:v1)..."
docker build -t $IMAGE_URI .

echo "📤 Pushing image to Artifact Registry..."
docker push $IMAGE_URI
```

## Next Step

Proceed to [Publishing & Deploying the Model](05-deploy-model.md).
