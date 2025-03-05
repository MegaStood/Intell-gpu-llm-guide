
README.md (Tutorial Content)

# Running LLMs on Intel GPUs: A Complete Guide

This tutorial demonstrates how to set up Intel GPU acceleration for Large Language Models (LLMs) on an Intel 12th Generation CPU with Iris Xe integrated graphics. We'll cover everything from initial driver setup to running models with a graphical interface.

**Table of Contents**  
1. [Prerequisites](#prerequisites)  
2. [Set Up the Intel GPU Environment](#set-up-the-intel-gpu-environment)  
   1. [Install Intel GPU Drivers](#install-intel-gpu-drivers)  
   2. [Install oneAPI Base Toolkit](#install-oneapi-base-toolkit)  
3. [Set Up Ollama with Intel GPU Support](#set-up-ollama-with-intel-gpu-support)  
   1. [Download and Set Up Portable Ollama](#download-and-set-up-portable-ollama)  
   2. [Create Convenient Launcher Scripts](#create-convenient-launcher-scripts)  
4. [Run Ollama with Intel GPU Acceleration](#run-ollama-with-intel-gpu-acceleration)  
5. [Set Up Open WebUI (Optional)](#set-up-open-webui-optional)  
   1. [Install Prerequisites](#install-prerequisites)  
   2. [Set Up Open WebUI](#set-up-open-webui)  
6. [Monitoring GPU Usage](#monitoring-gpu-usage)  
7. [Tips for Best Performance](#tips-for-best-performance)  
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

- **Intel 12th Gen (or newer) CPU with Iris Xe Graphics**  
- **Ubuntu 22.04 LTS (or newer)**
- **An internet connection**

---

## Set Up the Intel GPU Environment

### Install Intel GPU Drivers

First, add the Intel graphics repository and install driver packages:

```bash
# Add Intel graphics repository
wget -qO - https://repositories.intel.com/gpu/intel-graphics.key | \
  sudo gpg --yes --dearmor --output /usr/share/keyrings/intel-graphics.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] \
  https://repositories.intel.com/gpu/ubuntu jammy/lts/2350 unified" | \
  sudo tee /etc/apt/sources.list.d/intel-gpu-jammy.list

sudo apt update

# Install driver packages
sudo apt-get -y install gawk dkms linux-headers-$(uname -r) libc6-dev
sudo apt install -y intel-i915-dkms=1.24.2.17.240301.20+i29-1 intel-fw-gpu=2024.17.5-329~22.04

# Install compute runtime
sudo apt-get install -y udev \
    intel-opencl-icd intel-level-zero-gpu level-zero \
    intel-media-va-driver-non-free libmfx1 libmfxgen1 libvpl2

# Reboot to apply driver changes
sudo reboot
Install oneAPI Base Toolkit
The oneAPI toolkit provides libraries for GPU acceleration:


# Add Intel oneAPI repository
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
  | gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] \
  https://apt.repos.intel.com/oneapi all main" \
  | sudo tee /etc/apt/sources.list.d/oneAPI.list

sudo apt update

# Install oneAPI packages
sudo apt install -y intel-oneapi-common-vars=2024.0.0-49406 \
  intel-oneapi-common-oneapi-vars=2024.0.0-49406 \
  intel-oneapi-diagnostics-utility=2024.0.0-49093 \
  intel-oneapi-compiler-dpcpp-cpp=2024.0.2-49895 \
  intel-oneapi-dpcpp-ct=2024.0.0-49381 \
  intel-oneapi-mkl=2024.0.0-49656 \
  intel-oneapi-mkl-devel=2024.0.0-49656 \
  intel-oneapi-dnnl-devel=2024.0.0-49521 \
  intel-oneapi-dnnl=2024.0.0-49521
Set Up Ollama with Intel GPU Support
In this guide, we’ll use a special Intel-optimized version of Ollama that’s pre-configured for Intel GPUs.

Download and Set Up Portable Ollama

# Create directory for Ollama
mkdir -p ~/intel-ollama
cd ~/intel-ollama

# Download portable Ollama
wget https://github.com/intel/ipex-llm/releases/download/v2.2.0-nightly/ipex-ollama-linux-x64.tgz

# Extract the package
tar -xzf ipex-ollama-linux-x64.tgz

# Make scripts executable
chmod +x start-ollama.sh
chmod +x ollama
Create Convenient Launcher Scripts

# Create launcher directory
mkdir -p ~/.local/bin

# Create launcher for Ollama server
cat > ~/.local/bin/intel-ollama-server << 'EOF'
#!/bin/bash
cd "$HOME/intel-ollama"

# Clear any existing oneAPI environment variables
unset LD_LIBRARY_PATH
unset CPATH
unset LIBRARY_PATH
unset ONEAPI_ROOT
unset CMPLR_ROOT

# Set necessary environment variables
export OLLAMA_NUM_GPU=999
export no_proxy=localhost,127.0.0.1
export ZES_ENABLE_SYSMAN=1
export SYCL_CACHE_PERSISTENT=1
export SYCL_PI_LEVEL_ZERO_USE_IMMEDIATE_COMMANDLISTS=1
export IPEX_LLM_NUM_CTX=8192  # Increase context length

# Start the Ollama server
./start-ollama.sh
EOF

# Create launcher for Ollama client
cat > ~/.local/bin/intel-ollama << 'EOF'
#!/bin/bash
cd "$HOME/intel-ollama"
./ollama "$@"
EOF

# Make the launchers executable
chmod +x ~/.local/bin/intel-ollama-server
chmod +x ~/.local/bin/intel-ollama

# Add to PATH
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
source ~/.bashrc
Run Ollama with Intel GPU Acceleration
Now you can start the Ollama server with GPU acceleration:


# Start the server
intel-ollama-server
Then, in a new terminal, pull and run a model:


# Pull a model compatible with your GPU
intel-ollama pull gemma:7b

# Run the model
intel-ollama run gemma:7b
Set Up Open WebUI (Optional)
If you want a graphical interface to interact with Ollama, you can use Open WebUI.

Install Prerequisites

# Install Node.js LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Check versions
node -v  # Should be at least v20.x.x
npm -v   # Should be at least 10.x.x
Set Up Open WebUI

# Clone repository
git clone https://github.com/open-webui/open-webui.git
cd open-webui/

# Copy the environment config
cp -RPp .env.example .env

# Install dependencies and build frontend
npm i
npm run build

# Install backend dependencies
cd ./backend
pip install -r requirements.txt -U
cd ..

# Start Open WebUI
export no_proxy=localhost,127.0.0.1
bash start.sh
After the server starts, visit http://localhost:8080 in your browser to access Open WebUI.

Create an account
Log in
Go to Settings -> Connections and set the Ollama API URL to http://localhost:11434
Monitoring GPU Usage
Use Intel's GPU tools to monitor utilization:


# Install Intel GPU tools
sudo apt install intel-gpu-tools

# Monitor GPU usage
sudo intel_gpu_top
Look for activity in the "Render/3D" engine to confirm GPU acceleration.

Tips for Best Performance
Try smaller models first (e.g., 7B parameters) before attempting larger ones on integrated GPUs.
Check GPU utilization with intel_gpu_top. If you see high usage in "Render/3D," the GPU is working.
Modify environment variables in intel-ollama-server if needed.
SYCL/Level Zero environment variables can be toggled for performance:
SYCL_PI_LEVEL_ZERO_USE_IMMEDIATE_COMMANDLISTS=0 vs =1
If you have multiple GPUs, you can set ONEAPI_DEVICE_SELECTOR to pick specific devices.
Troubleshooting
Verify Ollama is using the GPU:
Look for runners=[ipex_llm] in the server logs, indicating Intel GPU acceleration.
DKMS build errors:
Often occur for non-active kernels; usually safe to ignore if the module builds for your current kernel.
GPU not detected:
Reinstall the i915 driver packages or check dmesg for error messages.
Performance issues:
Remember that integrated GPUs have limited memory compared to dedicated GPUs, so don’t expect the same performance as a high-end discrete GPU.
Adjust GPU layers:
Modify OLLAMA_NUM_GPU in your launcher script if you want more or fewer layers on the GPU.
Enjoy running LLMs on your Intel GPU!

