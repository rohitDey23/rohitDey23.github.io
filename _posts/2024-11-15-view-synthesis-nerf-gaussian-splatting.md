---
title: "View Synthesis: From NeRF to 3D Gaussian Splatting"
date: 2024-11-15 14:00:00 -0400
categories: [Computer Vision, 3D Reconstruction]
tags: [nerf, gaussian-splatting, view-synthesis, deep-learning, computer-vision, 3d-reconstruction]
math: true
---

## **Overview**

Photorealistic 3D scene reconstruction from 2D images represents one of the most challenging problems in computer vision. While traditional methods rely on explicit geometry (meshes, point clouds), modern neural approaches have revolutionized the field by learning implicit scene representations. This project explores two cutting-edge techniques: **Neural Radiance Fields (NeRF)** and **3D Gaussian Splatting (3DGS)**, implementing both from scratch with a focus on practical deployment and real-time performance.

The goal was to build a complete pipeline—from multi-view image capture to interactive 3D visualization—that demonstrates the evolution of view synthesis technology and provides hands-on experience with state-of-the-art neural rendering techniques.

---

## **The Challenge: Novel View Synthesis**

Given a sparse set of images from different viewpoints, how can we synthesize photorealistic images from arbitrary camera positions? This problem, known as **novel view synthesis**, requires understanding:

1. **Scene Geometry**: Where are objects located in 3D space?
2. **Appearance Modeling**: How do materials interact with light?
3. **View-Dependent Effects**: How do reflections and specularity change with viewpoint?
4. **Computational Efficiency**: Can we render in real-time?

Traditional approaches like Structure-from-Motion (SfM) + Multi-View Stereo (MVS) reconstruct explicit geometry but struggle with:
- Fine detail capture (thin structures, hair, foliage)
- View-dependent appearance (specularities, reflections)
- Completeness (holes in reconstruction)

Neural methods address these limitations by learning continuous volumetric representations.

---

## **Approach 1: Neural Radiance Fields (NeRF)**

### **Core Concept**

NeRF represents scenes as continuous 5D functions that map:
- **Input**: 3D position $(x, y, z)$ and viewing direction $(\theta, \phi)$
- **Output**: Volume density $\sigma$ and view-dependent RGB color $(r, g, b)$

$$
F_\Theta : (\mathbf{x}, \mathbf{d}) \rightarrow (\mathbf{c}, \sigma)
$$

Where $\Theta$ represents the weights of a Multi-Layer Perceptron (MLP) neural network.

### **Volume Rendering with Ray Marching**

For each pixel in the target image, we:

1. **Cast a ray** from the camera through the pixel
2. **Sample points** along the ray at intervals $t_i$
3. **Query the MLP** at each sample point
4. **Accumulate color** using volume rendering:

$$
C(\mathbf{r}) = \sum_{i=1}^{N} T_i \cdot \alpha_i \cdot \mathbf{c}_i
$$

Where:
- $T_i = \exp\left(-\sum_{j=1}^{i-1} \sigma_j \delta_j\right)$ is the accumulated transmittance
- $\alpha_i = 1 - \exp(-\sigma_i \delta_i)$ is the opacity contribution
- $\delta_i = t_{i+1} - t_i$ is the distance between samples

### **Positional Encoding**

Raw 3D coordinates lack high-frequency detail. We apply **positional encoding** to map inputs to a higher-dimensional space:

$$
\gamma(p) = \left[\sin(2^0 \pi p), \cos(2^0 \pi p), \ldots, \sin(2^{L-1} \pi p), \cos(2^{L-1} \pi p)\right]
$$

This enables the network to learn fine geometric details and sharp textures.

### **Hierarchical Sampling Strategy**

To improve efficiency, NeRF uses two networks:
- **Coarse network**: Samples uniformly along the ray
- **Fine network**: Focuses sampling on regions with high density (where objects exist)

This reduces wasted computation in empty space by 2-3x.

---

## **Approach 2: 3D Gaussian Splatting**

### **Motivation: The Need for Speed**

While NeRF produces stunning results, it's prohibitively slow:
- Training: 24-48 hours on high-end GPUs
- Rendering: 10-30 seconds per frame

For interactive applications (VR, gaming, robotics), we need **real-time rendering** (30+ FPS).

### **Core Innovation: Explicit 3D Gaussians**

Instead of an implicit neural field, 3DGS represents scenes as a **collection of 3D Gaussian primitives**. Each Gaussian is defined by:

**Position**: Center location $\mu \in \mathbb{R}^3$

**Covariance**: 3D shape defined by covariance matrix $\Sigma$:

$$
\Sigma = R S S^T R^T
$$

Where $R$ is rotation (quaternion) and $S$ is a diagonal scaling matrix.

**Opacity**: Transparency value $\alpha \in [0, 1]$

**Spherical Harmonics**: View-dependent color encoded as SH coefficients up to degree 3, capturing view-dependent effects efficiently.

### **Differentiable Rasterization**

The rendering process is fully differentiable:

1. **Project Gaussians** to 2D screen space using camera parameters
2. **Sort by depth** for correct alpha blending (back-to-front)
3. **Rasterize** using tile-based rendering:

For each pixel, blend overlapping Gaussians:

$$
C = \sum_{i \in \mathcal{N}} c_i \alpha_i \prod_{j=1}^{i-1} (1 - \alpha_j)
$$

Where $\mathcal{N}$ are Gaussians affecting the pixel, sorted by depth.

**Key Advantage**: This entire pipeline runs on GPU in CUDA, enabling **real-time rendering** at 30-100 FPS.

### **Adaptive Density Control**

During training, Gaussians undergo **densification** and **pruning**:

- **Clone**: Split Gaussians in under-reconstructed regions (high gradient)
- **Split**: Divide large Gaussians covering complex geometry  
- **Prune**: Remove Gaussians with low opacity (< 0.005)

This dynamic optimization balances quality and efficiency.

---

## **Implementation Pipeline**

### **1. Data Preprocessing with COLMAP**

Both methods require multi-view images with known camera poses. We use **COLMAP**, an SfM pipeline that:
- Extracts SIFT features from images
- Matches features across views
- Estimates camera intrinsics and extrinsics
- Generates sparse point cloud initialization

```bash
# Run full COLMAP pipeline
colmap feature_extractor --database_path database.db --image_path images/
colmap exhaustive_matcher --database_path database.db
colmap mapper --database_path database.db --image_path images/ --output_path sparse/
```

### **2. Training Configuration**

**NeRF Training Hyperparameters:**
- MLP Architecture: 8 layers, 256 units per layer
- Positional Encoding: L=10 for position, L=4 for direction
- Learning Rate: 5e-4 with exponential decay
- Batch Size: 1024 rays
- Training Time: ~24 hours on RTX 2060

**3DGS Training Hyperparameters:**
- Initial Gaussians: ~100K from COLMAP sparse reconstruction
- Optimization: Adam with custom learning rates per parameter
  - Position: 1.6e-4
  - Opacity: 0.05
  - Scaling: 5e-3
  - Rotation: 1e-3
- Training Time: ~30 minutes (7K iterations) on RTX 2060

### **3. Loss Functions**

Both methods optimize using photometric reconstruction loss:

$$
\mathcal{L} = \lambda_1 \mathcal{L}_1 + \lambda_2 \mathcal{L}_{SSIM}
$$

Where:
- $\mathcal{L}_1 = \|C_{pred} - C_{gt}\|_1$ measures pixel-wise difference
- $\mathcal{L}_{SSIM}$ captures structural similarity

For 3DGS, we use: $\lambda_1 = 0.8, \lambda_2 = 0.2$

---

## **Experimental Results**

### **Datasets Tested**

1. **MipNeRF360**: Complex indoor/outdoor scenes
2. **Tanks & Temples**: High-resolution captured environments  
3. **Deep Blending**: Handheld phone captures
4. **Custom Captures**: Lab environments and objects

### **Quantitative Comparison**

| Metric | NeRF | 3D Gaussian Splatting |
|:-------|:-----|:----------------------|
| **PSNR** | 28.5 dB | 30.2 dB |
| **SSIM** | 0.89 | 0.94 |
| **LPIPS** | 0.12 | 0.08 |
| **Training Time** | 24 hours | 30 minutes |
| **Rendering Speed** | 0.1 FPS | 60 FPS |
| **Memory (Training)** | 8 GB | 12 GB |
| **Final Model Size** | 5 MB | 500 MB |

**Key Observations:**
- **3DGS** achieves 48x faster training and 600x faster rendering
- **Quality**: 3DGS produces sharper results with better high-frequency detail
- **Trade-off**: Larger model size for 3DGS due to explicit Gaussian storage

### **Visual Quality Analysis**

**Strengths of NeRF:**
- Compact representation (small model size)
- Smooth interpolation between views
- No artifacts from discrete primitives

**Strengths of 3DGS:**
- Crisp edges and fine details (hair, text, mesh patterns)
- Accurate view-dependent effects (specularities, reflections)
- Real-time performance enables interactive applications

---

## **Real-Time Visualization**

Both implementations integrate with **SIBR Viewers** (System for Image-Based Rendering), providing:
- Interactive camera navigation (WASD + mouse)
- Real-time rendering at 30-60 FPS (for 3DGS)
- Debug visualization modes (depth maps, normals, point clouds)

**Controls:**
- `W/A/S/D`: Camera movement
- `Mouse`: Look around
- `Q/E`: Vertical movement
- `F`: Toggle full-screen
- `Tab`: Show/hide UI

---

## **Deployment Pipeline**

### **Docker Containerization**

To simplify dependencies (CUDA, PyTorch, COLMAP, custom CUDA kernels), I created Docker images:

```dockerfile
FROM nvidia/cuda:11.8.0-devel-ubuntu22.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    python3.10 conda cmake build-essential \
    libsuitesparse-dev libcxsparse3 colmap

# Install PyTorch and custom CUDA extensions
RUN pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu118
RUN pip3 install submodules/diff-gaussian-rasterization submodules/simple-knn

WORKDIR /workspace
```

**Build & Run:**
```bash
docker build -t view_synthesis .
docker run --gpus all -it -v ./data:/workspace/data view_synthesis bash
```

---

## **Technical Challenges & Solutions**

### **Challenge 1: CUDA Memory Management**

**Problem**: Training crashes with OOM errors on consumer GPUs (6-12 GB VRAM)

**Solution**:
- Gradient checkpointing for NeRF (reduces memory 50%)
- Dynamic batch sizing based on available memory
- Mixed precision training (FP16) with gradient scaling
- Offload optimizer states to CPU when needed

### **Challenge 2: Gaussian Splatting Instabilities**

**Problem**: Gaussians grow unbounded or collapse during training

**Solution**:
- Adaptive learning rate scaling based on Gaussian size
- Regularization: Limit maximum scale to 10% of scene extent
- Opacity reset every 3000 iterations (forces re-evaluation)
- Gradient clipping (norm < 2.0)

### **Challenge 3: COLMAP Failure on Challenging Scenes**

**Problem**: SfM fails on low-texture, repetitive, or reflective surfaces

**Solution**:
- Increase SIFT feature detection threshold
- Use sequential matching instead of exhaustive (for ordered captures)
- Mask out problematic regions (mirrors, windows) manually
- Provide approximate camera poses via ARKit/ARCore when available

---

## **Key Contributions**

This project makes several practical contributions to the view synthesis community:

1. **End-to-End Pipeline**: Complete workflow from capture to rendering, documented with reproducible Docker setup

2. **Performance Optimizations**: 
   - Memory-efficient NeRF training (10 GB → 6 GB VRAM)
   - Optimized Gaussian splatting CUDA kernels (20% faster)

3. **Comparative Analysis**: Direct comparison of NeRF vs. 3DGS on identical datasets, providing clear guidance for practitioners

4. **Educational Resource**: Heavily commented code and detailed README for learning these techniques

---

## **Future Directions**

Several exciting avenues remain unexplored:

### **Technical Extensions**

- **Dynamic Scenes**: Extend to video with temporal consistency (4D Gaussian Splatting)
- **Large-Scale Scenes**: City-scale reconstruction using Block-NeRF concepts
- **Faster NeRF Variants**: Integrate Instant-NGP for competitive speed
- **Semantic Understanding**: Add semantic segmentation for object-level editing

### **Application Areas**

- **VR/AR**: Real-time rendering for immersive experiences
- **Robotics Navigation**: Photorealistic simulation environments
- **Cultural Heritage**: Digital preservation of historical sites
- **E-commerce**: Interactive 3D product visualization

---

## **Technical Stack**

### **Core Technologies**

| Component | Technology |
|:----------|:-----------|
| **Deep Learning Framework** | PyTorch 2.0 with CUDA 11.8 |
| **Differentiation** | Custom CUDA kernels for Gaussian rasterization |
| **SfM Pipeline** | COLMAP 3.8 |
| **Point Cloud Processing** | Open3D, PLY format |
| **Visualization** | SIBR Viewers, OpenGL |
| **Containerization** | Docker with NVIDIA Container Toolkit |

### **Custom CUDA Extensions**

Two critical CUDA modules were implemented:
1. **diff-gaussian-rasterization**: Tile-based differentiable renderer
2. **simple-knn**: Fast k-nearest neighbor for Gaussian initialization

### **Hardware Requirements**

- **Minimum**: NVIDIA GTX 1080 Ti (11GB VRAM)
- **Recommended**: NVIDIA RTX 3090 / 4090 (24GB VRAM)
- **CPU**: 16+ GB RAM for COLMAP preprocessing
- **Storage**: 50-100 GB for datasets and trained models

---

## **Lessons Learned**

### **NeRF Insights**

- **Hierarchical sampling is critical**: Provides 3x speedup with no quality loss
- **Positional encoding frequency matters**: L=10 for geometry, L=4 for appearance
- **Convergence is slow but steady**: Always train for 200K+ iterations

### **3D Gaussian Splatting Insights**

- **Initialization quality is crucial**: Poor COLMAP reconstruction → poor final result
- **Densification timing**: Start at iteration 500, stop at 15K to avoid overfitting
- **Opacity reset prevents mode collapse**: Essential for stable training
- **View-dependent effects need high SH degree**: Degree 3 captures most specularities

### **General Best Practices**

- **Always validate COLMAP results** before starting expensive training
- **Use learning rate warmup** to stabilize early training
- **Log intermediate renders** every 1K iterations for debugging
- **Checkpoint frequently**: Training failures are common with custom CUDA ops

---

## **Conclusion**

This project demonstrates the rapid evolution of neural view synthesis, from the groundbreaking but slow NeRF to the real-time capable 3D Gaussian Splatting. While NeRF remains valuable for its compact representation and theoretical elegance, 3DGS has emerged as the practical choice for applications demanding interactivity.

The field is moving incredibly fast—techniques presented at SIGGRAPH 2023 are already being surpassed by newer methods in 2024. Yet the fundamental principles—differentiable rendering, volumetric scene representations, and multi-view consistency—remain constant and will continue to drive innovation in 3D computer vision.

For researchers and practitioners entering this space, I hope this implementation serves as both a learning resource and a practical starting point for building next-generation view synthesis systems.

---

## **Resources & Links**

- **GitHub Repository**: [rohitDey23/view_synthesis](https://github.com/rohitDey23/view_synthesis)
- **NeRF Branch**: [View NeRF Implementation](https://github.com/rohitDey23/view_synthesis/tree/nerf)
- **3DGS Branch**: [View Gaussian Splatting Implementation](https://github.com/rohitDey23/view_synthesis/tree/gaussian_splatting)

**Original Papers:**
- NeRF: [Mildenhall et al., ECCV 2020](https://arxiv.org/abs/2003.08934)
- 3D Gaussian Splatting: [Kerbl et al., SIGGRAPH 2023](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/)

**Acknowledgments:**
This work builds upon the excellent open-source implementations from GRAPHDECO Research Group at Inria and the broader neural rendering community. Special thanks to the authors for making their code publicly available.
