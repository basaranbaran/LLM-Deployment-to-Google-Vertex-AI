# Publishing & Deploying the Model

> [!NOTE]
> **Execution Context:** Virtual Machine (VM) - Terminal

This section covers running the uploaded container on a GPU and making it accessible as an API.

## 1. Register the Model with Vertex AI

Register the uploaded container image as a Model resource in Vertex AI.

```bash
echo "🔄 Uploading model to Vertex AI..."
gcloud ai models upload \
  --region=$REGION \
  --display-name="pixtral-run-v1" \
  --container-image-uri=$IMAGE_URI \
  --container-health-route="/health" \
  --container-predict-route="/predict" \
  --container-ports="8080"
```

## 2. Create an Endpoint

Create a persistent URL (endpoint) through which the model can be accessed.

```bash
echo "🌐 Creating endpoint..."
gcloud ai endpoints create \
  --region=$REGION \
  --display-name="pixtral-run-endpoint"
```

## 3. Deploy the Model to the Endpoint

Instruct Vertex AI to provision a server equipped with an NVIDIA A100 GPU, run our container, and connect it to the endpoint.

> [!NOTE]
> This process may take approximately 10–25 minutes to complete.

```bash
# Get Model and Endpoint IDs
export MODEL_ID=$(gcloud ai models list --region=$REGION --filter="displayName=pixtral-run-v1" --format="value(name)" | head -1)
export ENDPOINT_ID=$(gcloud ai endpoints list --region=$REGION --filter="displayName=pixtral-run-endpoint" --format="value(name)" | head -1)

echo "🚀 Deploying model to endpoint... (Please wait 10–25 minutes)"
gcloud ai endpoints deploy-model $ENDPOINT_ID \
  --region=$REGION \
  --model=$MODEL_ID \
  --display-name="pixtral-run-deployment" \
  --machine-type="a2-highgpu-1g" \
  --accelerator="type=nvidia-tesla-a100,count=1" \
  --min-replica-count=1 \
  --max-replica-count=2 
```
*Note: We set `max-replica-count` to 2 for scaling, but you can set it to 1 if preferred.*

## Next Step

Proceed to [Testing & Verification](06-verification.md).
