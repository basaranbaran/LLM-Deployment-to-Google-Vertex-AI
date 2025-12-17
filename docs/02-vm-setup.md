# VM Setup & Folder Structure

All commands should be executed in a Linux development environment (VM) that has gcloud and Docker installed, and is authorized for the project.

## Virtual Machine (VM) Specifications

> [!NOTE]
> **Execution Context:** Google Cloud Console / Web Browser

Create a VM with the following specifications:

*   **Region**: Select a region that supports the A100 GPU (e.g., “europe-west4”).
*   **Machine Type**: Select a VM from the E2 series under General Purpose.
*   **Disk Size**: 200 GB.
*   **Image**: Debian GNU/Linux 12 (bookworm).

## Virtual Machine Commands

> [!NOTE]
> **Execution Context:** Virtual Machine (VM) - Terminal

SSH into your VM and run the following commands:

```bash
# System update
sudo apt-get update && sudo apt-get upgrade -y

# Install essential tools
sudo apt-get install -y git git-lfs curl wget gnupg apt-transport-https ca-certificates software-properties-common

# Enable Git LFS
git lfs install

# Install Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Allow Docker usage without sudo
sudo usermod -aG docker $USER

# Install Python and pip
sudo apt-get install -y python3 python3-pip python3-venv

# Configure Google Cloud SDK
gcloud auth login
gcloud config set project llm-development-XXXX # Replace with your “Project ID”
gcloud config set compute/region europe-west4 # Replace with your chosen region

# Authenticate Docker for Artifact Registry
gcloud auth configure-docker europe-west4-docker.pkg.dev # Replace with your selected region

# Create an Artifact Registry repository (first time only)
gcloud artifacts repositories create pixtral-models-repo \
  --repository-format=docker \
  --location=europe-west4 \
  --description="Pixtral model containers"

echo "✅ VM setup complete!"
echo "❗ Please type 'exit' to close SSH and reconnect (required for Docker group changes)."
```

After Docker is installed, close your SSH terminal and reconnect. Then, create your project directory:

```bash
mkdir -p pixtral-run
cd pixtral-run
```

## Next Step

Proceed to [Creating Application Files](03-application-setup.md).
