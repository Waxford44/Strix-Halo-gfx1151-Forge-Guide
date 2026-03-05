# Strix-Halo-gfx1151-Forge-Guide
Optimized Forge Neo setup for AMD Strix Halo (gfx1151) on CachyOS. This guide is written as of February 2026. It is meant as a one time guide and was made in colaboration with Gemini 3.1. It is not a guide I will maintain.

🚀 Strix Halo: The Ultimate Forge Neo & ROCm 7.12 Optimization Guide

🌟 Introduction
The AMD Strix Halo (RDNA 3.5) is a revolutionary APU, but out-of-the-box performance on Windows or standard Linux distros is often limited to ~0.5 it/s. This guide shows you how to leverage CachyOS and ROCm 7.12 Nightlies to unlock 300% more performance, hitting 1.5+ it/s on SDXL models with a unified 32GB VRAM allocation. I am writing this one time guide because I found it difficult to find a linux guide on making stable diffusion work for this APU. Mostly because all available guides are written for Comfyui. But let's be honest, for enthusiasts, comfyui is anything other than comfy. Will I do understand the benefits and control it offers, those are features used by pro devs. If you are tired of pipelines, nodes, and just want to get some image generations done and have fun with your new APU,then you're in luck because Forge comes to the rescue. 

💻 System Specs & Environment

Device: GMKTEC EVO-X2

APU: AMD Strix Halo (gfx1151) - Radeon 8060S Graphics

VRAM: 32GB Allocated Unified RAM (64GB System Total)

OS: CachyOS (Linux-Zen Kernel)

Drivers: ROCm 7.12 Nightlies

Software: Forge Neo (2026 Branch)

"Neo" mainly serves as an continuation for the "latest" version of Forge, which was built on Gradio 4.40.0 before lllyasviel became too busy... Additionally, this fork is focused on optimization and usability, with the main goal of being able to run the latest popular models, on an easy-to-use GUI without any bloatwares. It is a fork of Forge with all the latest bells and whistles and most importantly it works up to python 3.13. The original Forge fork was made under 3.10 which will clash with the ROCm nightlies as they have the best stability on python 3.11.

https://github.com/Haoming02/sd-webui-forge-classic

So let's get this started.

🛠 Step 1: System & VRAM Prep (The "Unified" Secret)
Unlike traditional GPUs, Strix Halo shares its memory with the system. We need to tell Linux exactly how much RAM to "reserve" for the GPU.

Install CoreCtrl to manage APU power profiles. APU power management is broken without this, at least on the GMKtec. rocm-smi leaves the chip throttled.

<pre>

sudo pacman -S corectrl
  </pre>
  
Allocate VRAM: Reboot into your BIOS. Find the "UMA Framebuffer" or "VRAM" setting. Set this to 32GB. (If your BIOS is locked, the kernel will manage this via GTT, but BIOS-level reservation is more stable).

🐍 Step 2: Python 3.11 Environment
Forge Neo currently performs best on Python 3.11. Since CachyOS usually ships with 3.13, we create a specific "jail" for Forge.

Install Python 3.11:
<pre>

sudo pacman -S python311
  </pre>
  
Clone the Forge Neo Repository:

<pre>
git clone https://github.com/Haoming02/sd-webui-forge-classic --branch neo forge-neo
cd forge-neo
  </pre>
  
Create and Activate the virtual environment:

<pre>
python3.11 -m venv venv
source venv/bin/activate.fish # Use .bash if not using Fish shell
</pre>

🏗 Step 3: Install ROCm 7.12 Nightlies

Standard ROCm versions don't yet fully support the gfx1151 architecture of the Strix Halo. We use the nightly builds to get the latest instructions.TheROCk is AMD's official next-generation stack, not a community fork. ROCm is at 7.2 as of writing this guide. By using the Nightlies, the performance difference is dramatic.

<pre>
pip install --upgrade pip
pip uninstall torch torchvision torchaudio -y
pip install --index-url https://rocm.nightlies.amd.com/v2/gfx1151/ torch torchvision torchaudio --force-reinstall</pre>

Verify:

<pre>
python -c "import torch; print(torch.__version__)"
# It should say: 2.9.1+rocm7.12.0a20260303</pre>

⚡ Step 4: The High-Performance Startup Script

Create a file named start_neo.fish in your home directory. You can use the standard KWRITE app. This script contains the "magic" environment variables that prevent the GPU from crashing during heavy AI loads.

<pre>#!/usr/bin/fish

# 1. Environment Variables for Strix Halo Stability
set -x HSA_ENABLE_SDMA 0
set -x HSA_USE_SVM 0
set -x TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL 1
set -x PYTORCH_ALLOC_CONF "garbage_collection_threshold:0.8,max_split_size_mb:512"

# 2. Launch Forge with RDNA 3.5 Optimizations
cd /home/YOUR_USER/forge-neo
source venv/bin/activate.fish

python launch.py \
    --device-id 0 \
    --cuda-malloc \
    --pin-shared-memory \
    --bf16-unet \
    --bf16-vae \
    --bf16-text-enc \
    --skip-prepare-environment \
    --skip-version-check \
    --disable-smart-memory</pre>

Make it executable: <pre> chmod +x ~/start_neo.fish </pre>

Launch it and let it install all the packages it needs. It will automatically detect the Nightlies torch files we manually installed earlier and skip them. The UI should pop up in your browser. Congratulations, you just managed to install Forge!

📊 Benchmark Results

Resolution: 832x1216 (SDXL model)

Sampler: Euler a

Steps: 20

Speed: 1.4-1.53 it/s

🛑 Support Disclaimer
This guide is provided "as-is" for the community. I am not a developer and cannot provide technical support or troubleshooting. If it works for you, awesome! If not, please check the official Forge or ROCm Discord servers.

License
MIT - Use freely.

