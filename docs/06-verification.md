# Testing & Verification

This section includes authorizing new users and verifying the deployed model with a custom test script.

## 1. Authorizing Users (Administrator Step)

> [!NOTE]
> **Execution Context:** Google Cloud Shell or Local Terminal with Admin Access

Assign the Vertex AI User role to accounts that need access.

**Add a New Developer:**

```bash
gcloud projects add-iam-policy-binding llm-development-XXXX \
    --member="user:[DEVELOPER_EMAIL_ADDRESS]" \
    --role="roles/aiplatform.user"
```

**Add a New Service Account:**

```bash
gcloud projects add-iam-policy-binding llm-development-XXXX \
    --member="serviceAccount:[SERVICE_ACCOUNT_EMAIL]" \
    --role="roles/aiplatform.user"
```

## 2. Testing the Model (Developer Step)

> [!NOTE]
> **Execution Context:** Local Machine (Developer)

This step is performed on your local machine.

### Prerequisites
*   Python 3 installed.
*   Install the Vertex AI SDK: `pip install google-cloud-aiplatform`
*   Authenticating with `gcloud auth application-default login`.
*   Test images (`cat.jpg`, `dog.jpg`, `car.jpg`) in the same directory.

### Create the Test Script (final_test.py)

Update `PROJECT_ID`, `REGION`, and `ENDPOINT_ID` in the script below:

```python
# final_test.py (FINAL VERSION – 3 Test Scenarios)
import base64
import os
from google.cloud import aiplatform

# --- SETTINGS ---
PROJECT_ID = "llm-development-XXXX"
REGION = "europe-west4"
ENDPOINT_ID = "[YOUR_ENDPOINT_ID]"  # Endpoint ID from Step 5

# --- Choose which test scenario to run (1, 2, or 3) ---
TEST_SCENARIO_TO_RUN = 3

# --- Test Scenarios ---
PROMPT_1 = "What is in this image? Describe it in detail."
IMAGE_FILES_1 = ["cat.jpg"]
PARAMETERS_1 = {"temperature": 0.3, "maxOutputTokens": 1024}

PROMPT_2 = "Compare the two animals in these images. What are the main differences and similarities?"
IMAGE_FILES_2 = ["cat.jpg", "dog.jpg"]
PARAMETERS_2 = {"temperature": 0.4, "maxOutputTokens": 1024}

PROMPT_3 = "Look at all these images. Which one of them contains a car?"
IMAGE_FILES_3 = ["cat.jpg", "dog.jpg", "car.jpg"]
PARAMETERS_3 = {"temperature": 0.1, "maxOutputTokens": 512}

def run_final_test(prompt: str, image_paths: list, parameters: dict):
    if not image_paths:
        print("❌ ERROR: No images were provided.")
        return

    print(f"--- Preparing request for {len(image_paths)} image(s) ---")
    aiplatform.init(project=PROJECT_ID, location=REGION)
    endpoint = aiplatform.Endpoint(endpoint_name=ENDPOINT_ID)

    encoded_images = []
    for path in image_paths:
        if not os.path.exists(path):
            print(f"❌ ERROR: Image file not found at '{path}'.")
            return
        with open(path, "rb") as image_file:
            encoded_images.append(base64.b64encode(image_file.read()).decode("utf-8"))

    instance = {"prompt": prompt, "parameters": parameters}
    if len(encoded_images) == 1:
        instance["image"] = encoded_images[0]
    else:
        instance["images"] = encoded_images

    try:
        response = endpoint.predict(instances=[instance])
        full_response_text = response.predictions[0].get('generated_text', '')
        cleaned_text = full_response_text.removeprefix(prompt).strip()
        print("\\n🎉 MODEL RESPONSE (Cleaned):")
        print(cleaned_text)
    except Exception as e:
        print(f"\\n❌ ERROR! Request failed: {e}")

if __name__ == "__main__":
    if TEST_SCENARIO_TO_RUN == 1:
        run_final_test(PROMPT_1, IMAGE_FILES_1, PARAMETERS_1)
    elif TEST_SCENARIO_TO_RUN == 2:
        run_final_test(PROMPT_2, IMAGE_FILES_2, PARAMETERS_2)
    elif TEST_SCENARIO_TO_RUN == 3:
        run_final_test(PROMPT_3, IMAGE_FILES_3, PARAMETERS_3)
    else:
        print("❌ ERROR: Invalid scenario selected.")
```

### Run the Test

```bash
python final_test.py
```

## Next Step

Proceed to [Monitoring](07-monitoring.md).
