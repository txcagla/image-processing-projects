# Edge-Preserving Smoothing Filters with Gradient-Based Adaptive Parameter Tuning

This repository contains the official implementation, experimental pipeline, and evaluation suite for the Digital Image Processing (DIP) final term project. The study introduces a region-aware, spatially varying parameter adaptation layout for classical spatial denoising algorithms.

---

## 📌 Project Overview
Classical spatial smoothing filters (e.g., isotropic Gaussian) effectively suppress noise in homogeneous regions but induce severe blurring across high-frequency structural boundaries. This project implements and evaluates three foundational spatial filters—**Gaussian, Bilateral, and Guided Image Filters**—under multiple Additive White Gaussian Noise (AWGN) regimes ($\sigma \in \{10, 25, 50\}$). 

Furthermore, we propose a **Gradient-Based Adaptive Parameter Tuning Strategy** that dynamically modulates the local smoothing intensity $\sigma(x, y)$ based on local gradient magnitudes computed via Sobel operators. To eliminate computational bottlenecks in Python, a high-efficiency, non-overlapping **Tile-Based Processing Layout ($16 \times 16$ blocks)** is introduced.

---

## 🛠️ System Architecture & Methodology

The proposed pipeline operates through the following mathematical stages:
1. **Pre-Processing:** Input images are converted to grayscale using luminance weighting and linearly normalized to continuous float32 values in $[0.0, 1.0]$.
2. **Gradient Estimation:** Horizontal ($G_x$) and vertical ($G_y$) derivatives are calculated using $3 \times 3$ Sobel masks to obtain the isotropic gradient magnitude map $G(x,y) = \sqrt{G_x^2 + G_y^2}$.
3. **Adaptive Mapping:** The local parameter map is derived using an exponential decay distribution bound by boundary constraints:
   $$\sigma(x, y) = \sigma_{min} + (\sigma_{max} - \sigma_{min}) \cdot \exp(-\alpha \cdot G(x, y))$$
4. **Tile-Based Approximation:** The image domain is partitioned into $16 \times 16$ blocks. Each tile is processed atomically using the mathematical mean of its internal adaptive sigma values ($\sigma_{local}$).

---

## 📊 Experimental Results & Key Findings

All configurations were systematically bench-marked on the canonical **Kodak Lossless True Color Image Dataset (24 images)**. The first 4 images were reserved as a validation subset for grid-search optimization, yielding the optimal hyperparameter triple: **$\sigma_{min} = 1.0, \sigma_{max} = 5.0, \alpha = 5.0$**. The remaining 20 images served as the independent test partition.

### Quantitative Metrics Summary (Test Set Averages at $\sigma_\eta=25$)
* **Gaussian (Adaptive - Ours):** **24.80 dB** PSNR | **0.686** SSIM | **0.071s** Runtime
* **Bilateral (Adaptive - Ours):** **25.32 dB** PSNR | **0.663** SSIM | **0.950s** Runtime
* **Guided (Adaptive - Ours):** **22.26 dB** PSNR | **0.574** SSIM | **0.115s** Runtime

### Core Observations
* **Gaussian Transformation:** The proposed adaptive framework successfully injects edge-preserving characteristics into the standard Gaussian filter, yielding an average improvement of **$+2.94\text{ dB}$** and elevating the SSIM from $0.492$ to $0.686$ under moderate noise.
* **The Runtime-Performance Trade-off:** While Adaptive Bilateral offers precise edge demarcation, its block-wise matrix calculation demands high computational overhead ($0.950\text{ s}$ per frame), making Adaptive Gaussian ($0.071\text{ s}$) the ideal candidate for real-time applications.
* **Guided Filter Boundary Artifacts:** The adaptive Guided scheme suffers from step-wise parameter drops across non-overlapping tile borders, resulting in block boundaries that degrade quantitative quality matrices.

---

## 🚀 Getting Started

### Prerequisites
Run the following command to install the required dependencies in your environment (or Google Colab environment):
```bash
pip install scikit-image opencv-python-headless requests matplotlib numpy
