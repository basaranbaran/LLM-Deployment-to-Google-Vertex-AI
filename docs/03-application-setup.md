# Creating Application Files

> [!NOTE]
> **Execution Context:** Virtual Machine (VM) - Terminal

We need to create the core application logic and define dependencies.

## 1. Create the Prediction Server (predictor.py)

This Python file serves as the “brain” of the application. It handles incoming requests, loads the model from Hugging Face, and processes predictions.

Run the following command in your VM terminal:

```bash
cat > predictor.py << 'PRED_EOF'
# predictor.py (FINAL VERSION – Supports Parameters & Multiple Images)
import os
import base64
import io
from flask import Flask, request, jsonify
from transformers import LlavaForConditionalGeneration, AutoProcessor
from PIL import Image
import torch

app = Flask(__name__)

# --- Model Loading ---
MODEL_ID = "mistral-community/pixtral-12b"
print(f"🚀 Starting server... Model: {MODEL_ID}")
processor = AutoProcessor.from_pretrained(MODEL_ID)
model = LlavaForConditionalGeneration.from_pretrained(
    MODEL_ID, torch_dtype=torch.bfloat16, device_map="auto"
)
print("✅ Server is ready!")

@app.route('/health', methods=['GET'])
def health():
    return '', 200

@app.route('/predict', methods=['POST'])
def predict():
    try:
        data = request.get_json()
        instances = data.get("instances", [])
        predictions = []

        for inst in instances:
            prompt = inst.get("prompt")

            # Retrieve generation parameters or use default values
            params = inst.get("parameters", {})
            temperature = params.get("temperature", 0.2)  # Default: 0.2
            max_new_tokens = params.get("maxOutputTokens", 512)  # Default: 512

            # Support for both single (“image”) and multiple (“images”) image inputs
            image_b64_list = []
            if 'images' in inst and isinstance(inst['images'], list):
                image_b64_list = inst['images']
            elif 'image' in inst:
                image_b64_list.append(inst['image'])

            if not prompt or not image_b64_list:
                predictions.append({"error": "prompt and at least one image are required"})
                continue

            # Convert all received images to PIL format
            pil_images = [Image.open(io.BytesIO(base64.b64decode(img_data))).convert('RGB') for img_data in image_b64_list]

            # Dynamically construct the “content” list expected by the model
            content = [{"type": "image"} for _ in pil_images]
            content.append({"type": "text", "text": prompt})
            messages = [{"role": "user", "content": content}]

            prompt_text = processor.apply_chat_template(messages, add_generation_prompt=True)
            inputs = processor(text=prompt_text, images=pil_images, return_tensors="pt").to(model.device, torch.bfloat16)

            # Perform generation using user-provided parameters
            with torch.no_grad():
                outputs = model.generate(**inputs, max_new_tokens=max_new_tokens, temperature=temperature)

            # Process the response and extract the model’s output
            result = processor.decode(outputs[0], skip_special_tokens=True)
            assistant_response = result.split("ASSISTANT:")[-1].strip()
            predictions.append({"generated_text": assistant_response})

        return jsonify({"predictions": predictions})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    port = int(os.environ.get('AIP_HTTP_PORT', 8080))
    app.run(host='0.0.0.0', port=port)
PRED_EOF
echo "✅ predictor.py file created successfully."
```

## 2. Define Python Dependencies (requirements.txt)

Create the `requirements.txt` file to specify necessary libraries:

```bash
cat > requirements.txt << 'REQ_EOF'
flask
transformers
accelerate
Pillow
sentencepiece
protobuf
torch
REQ_EOF
echo "✅ requirements.txt file created successfully."
```

## 3. Create Dockerfile

Create the `Dockerfile` to build the container image:

```bash
cat > Dockerfile << 'EOF'
FROM pytorch/pytorch:2.1.2-cuda11.8-cudnn8-runtime

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY predictor.py .

EXPOSE 8080

CMD ["python3", "predictor.py"]
EOF
echo "✅ Dockerfile created successfully."
```

## Next Step

Proceed to [Building the Container](04-container-build.md).
