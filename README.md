# FETUS 2026 Docker Submission Template
[![FETUS 2026 Challenge Site](https://img.shields.io/badge/Official-FETUS%202026%20Challenge-red?style=for-the-badge)]((http://119.29.231.17:90/index.html))
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker)](https://www.docker.com/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.4+-orange?style=for-the-badge&logo=pytorch)](https://pytorch.org/)
[![CUDA](https://img.shields.io/badge/CUDA-11.8+-green?style=for-the-badge&logo=nvidia)](https://developer.nvidia.com/cuda-toolkit)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](./LICENSE)

## 📖 Introduction

This repository provides a **minimal, stable Docker submission template** for the **FETUS 2026 Challenge: Fetal HearT UltraSound Segmentation and Diagnosis Challenge**. Its goal is to standardize the inference pipeline so that participants only need to **replace the files**, without modifying the evaluation I/O logic.

## 🚀 Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/your_username/fetus2026_docker_submission_template.git
cd fetus2026_docker_submission_template

# 2. (Optional) Customize as needed
#    - Update base image: Edit Dockerfile line 1
#    - Update dependencies: Edit requirements.txt
#    - Replace model: Edit models/Model.py

# 3. Build Docker image
docker build -t {your_dockerhub_username}/{fetus2026_yourteamname_submission}:latest .

# 4. Local testing
docker run --rm --gpus all \
  -u $(id -u):$(id -g) \
  -v /path/to/sample_data:/data \
  -v /path/to/output:/output \
  {your_dockerhub_username}/{fetus2026_yourteamname_submission}:latest

# 5. Push to Docker Hub
docker push {your_dockerhub_username}/{fetus2026_yourteamname_submission}:latest
```

## 📁 Directory Structure

```
fetus2026_docker_submission_template/
├── checkpoints             # Default location for model weights
│	└──best.pth             # weight file
├── dataset                 # dataloader function 
├── model                   # model file
│	└──unet.py              # Baseline network
│	└──Echocare.py          
├── util                    # utility functions
├── inference.py            # Fixed inference entry script 
├── Dockerfile              # Docker image build recipe
├── requirements.txt        # Python dependencies
└── README.md               # This file
```

> ### 🪧 Notice
>
> 1).  Except for the Dockerfile, all other contents may be modified.  
>
> 2).  Within the Dockerfile, participants are only permitted to modify `the default command`; no other content may be altered. 
>
> ```
> Dockerfile
> 
> ...
> ...
> 
> # the defualt command  "inference.py" denotes the Python file that is actually executed during runtime.
> CMD ["python", "inference.py"]
> 
> If you choose to use your own inference code, please ensure that the input and output formats remain unchanged; otherwise, the execution may fail.
> ```
>
> 3).  When modifying  `the default command`, the specified Python file must be the main program used for final inference.
>
> 4).  There is no need to include the data directory in the Docker image; only the source code required for inference should be provided.

---

## 🛠️ Environment Setup

### 1. System Requirements

- **Docker**: 20.10+
- **NVIDIA Docker**: GPU inference support
- **CUDA**: 11.8+ (based on base image version)

> **📚 Installation Guide**: For detailed Docker installation instructions, please refer to the official documentation:
> - [Install Docker Engine (Linux)](https://docs.docker.com/engine/install/)
> - [Install Docker Desktop (Windows)](https://docs.docker.com/desktop/setup/install/windows-install/)

### 2. Verify Docker Environment

```bash
# Verify Docker installation
docker --version

# Verify NVIDIA Docker support
docker run --rm --gpus all nvidia/cuda:12.1-base-ubuntu22.04 nvidia-smi
```

---

## 🔧 Customization Options

### Option 1: Update Base Image

Edit line 1 of `Dockerfile` to choose an appropriate base image:

```dockerfile
# PyTorch 2.5 + CUDA 12.1 (default)
FROM pytorch/pytorch:2.5.1-cuda12.1-cudnn9-runtime

# Other available images:
# FROM pytorch/pytorch:2.4.0-cuda11.8-cudnn9-runtime
# FROM nvidia/pytorch:24.02-py3
```

> **Tip**: Choose a base image version that matches your model training environment.

### Option 2: Update Dependencies

Edit `requirements.txt`:

```txt
monai==1.5.1
scipy==1.15.3
scikit-learn==1.7.2
h5py==3.15.1
tensorboard==2.20.0
einops==0.8.1
numpy==1.26.4
# Add your custom dependencies...
```

### Option 3: Replace Model .etc

#### Step 1: Replace fiels with your own project.

#### Step 2: Place Model Weights

Copy your trained weights to the `checkpoints/` directory:

```bash
# Method 1: Use default path
cp your_best_model.pth checkpoints/best.pth

# Method 2: Mount external weights at runtime
docker run ... -e MODEL_PATH=/path/to/weights.pth ...
```

---

## 🐳 Build Docker Image

### Standard Build

```bash
docker build -t your_dockerhub_username/fetus2026_yourteamname_submission:latest .
```
---

## ▶️ Local Testing (Required Before Submission)

### Run Inference Test

```bash
docker run --rm --gpus all \
  -u $(id -u):$(id -g) \
  -v /path/to/sample_data:/data \
  -v /path/to/output:/output \
  your_dockerhub_username/fetus2026_yourteamname_submission:latest
```

> **✅ Verify Output**: After running, check that prediction files (`*.h5`) are generated in `/path/to/output`. Each `.h5` file should contain:
>
> - `mask`        np.ndarray,   # (512, 512) uint8 segmentation mask (0-14 classes)
>
> - `label`      np.ndarray   # (7,) uint8 binary classification labels
>
>   ⚠️ **Key names, data types, and file naming must not be changed**, otherwise evaluation will fail.

### Common Issues

| Issue | Solution |
|-------|----------|
| Permission errors | Ensure `-u $(id -u):$(id -g)` is used |
| GPU not available | Verify NVIDIA Docker is installed, run `nvidia-smi` |
| Weights file not found | Check `MODEL_PATH` environment variable and mount path |
| Out of memory | Reduce `BATCH_SIZE` or use CPU inference |

---

## 📦 Push to Image Registry

### 1. Tag Image

```bash
# Method 1: latest tag
docker tag fetus2026_yourteamname_submission:latest your_dockerhub_username/fetus2026_yourteamname_submission:latest

# Method 2: Specific version tag
docker tag fetus2026_yourteamname_submission:latest your_dockerhub_username/fetus2026_yourteamname_submission:v1.0
```

### 2. Push Image

```bash
# Push to Docker Hub
docker push your_dockerhub_username/fetus2026_yourteamname_submission:latest

# Push specific version
docker push your_dockerhub_username/fetus2026_yourteamname_submission:v1.0
```

### 3. Submit Image URL

After successfully pushing your Docker image, submit the image URL to the **FETUS Challenge Platform**:

- **Platform Link**: [FETUS 2026 Submission](http://119.29.231.17:9000/login)
- **Image URL Format**: `your_dockerhub_username/fetus2026_yourteamname_submission:latest` or `your_dockerhub_username/fetus2026_yourteamname_submission:v1.0`

---

## 📋 Complete Submission Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                  Complete Submission Workflow                   │
├─────────────────────────────────────────────────────────────────┤
│  1. Clone template                                              │
│     git clone <repo_url>                                        │
│     cd fetus2026_docker_submission_template                     │
│                                                                 │
│  2. Prepare model                                               │
│     • Replace files with your own project                       │
│                                                                 │
│  3. Configure environment (optional)                            │
│     • Update base image in Dockerfile                           │
│     • Update dependencies in requirements.txt                   │
│                                                                 │
│  4. Build image                                                 │
│     docker build -t <username>/fetus2026_submission:latest .    │
│                                                                 │
│  5. Local testing                                               │
│     docker run --rm --gpus all -v ... <username>/fetus2026...   │
│     • Verify output format is correct                           │
│                                                                 │
│  6. Push image                                                  │
│     docker login                                                │
│     docker push <username>/fetus2026_submission:latest          │
│                                                                 │
│  7. Submit link                                                 │
│     • Copy image URL to challenge submission website            │
└─────────────────────────────────────────────────────────────────┘
```
---

## 🖥️ Test Environment

The following environment was used for testing and development:

| Component | Specification |
|-----------|---------------|
| **CPU** | Intel(R) Xeon(R) Gold 6248R CPU @ 3.00GHz |
| **RAM** | 376G |
| **GPU** | NVIDIA A30 (24G) |
| **CUDA Version** | 13.0 |
---

## 💡 Best Practices

1. **Version Management**
   - Use meaningful image tags (e.g., `v1.0`, `final`)
   - Keep tested image versions

2. **Image Optimization**
   - Use Docker layer caching to reduce build time
   - Use `.dockerignore` to exclude unnecessary files

3. **Testing and Verification**
   - Thoroughly test with data provided by organizers before submission
   - Check that output files contain all required keys

4. **Documentation**
   - Record model architecture and training parameters
   - Save inference performance benchmarks

---

## 📄 License

This template is intended for **research and challenge submission purposes**. Participants are free to modify the model implementation while keeping the I/O interface intact.

---

## ❓ FAQ

### Q: Is Docker required?
A: Yes, the FETUS2026 Challenge requires all submissions to use Docker containers.

### Q: Can I modify inference.py?
A: **Not recommended**. Inference logic is frozen to ensure fair and consistent evaluation. Contact organizers for special needs.

### Q: How to handle large input sizes?
A: The model automatically resizes inputs to `RESIZE_TARGET` for inference, then upsamples back to original size.

### Q: Can I push to a private registry?
A: Yes, but ensure organizers have access permissions. Using a public registry or properly configured access is recommended.

---

## 📞 Contact

- **Challenge Website**: [FETUS 2026 Challenge](http://119.29.231.17:90/index.html)
- **Issue Reporting**: https://github.com/your_username/fetus2026_docker_submission_template/issues

---

If you use this template for your submission, please ensure that your final Docker image strictly follows the I/O specification described above.

