# An Edge AI Approach to Memory-Aware Vision Transformer Optimization, Quantization, and Embedded Deployment

## A Memory-Aware Vision Transformer for ESP32-S3

This is an ongoing B.Tech major project focused on optimizing
and deploying a Vision Transformer (ViT) for resource-constrained
microcontrollers, with the ESP32-S3 as the target platform.

The project investigates a practical optimization-to-deployment pipeline
covering memory-aware architecture reduction, INT8 weight quantization,
C/C++ inference, memory management, ESP-IDF integration, simulation in
Wokwi, and final benchmarking on physical ESP32-S3 hardware.

> **Project Status:** Ongoing\
> **Supervisor:** Dr. Sudhakar Modem\
> **Team:** Deepak Parmar and Gaurav Tiwari

------------------------------------------------------------------------

## Motivation

Vision Transformers provide strong performance for computer vision
tasks, but their memory and computational requirements can make direct
deployment on microcontrollers difficult.

This project studies the trade-off between:

-   Model accuracy
-   Number of parameters
-   FLOPs
-   Weight memory
-   Activation memory
-   Quantization error
-   Embedded inference latency
-   RAM and flash consumption

The main objective is to develop a compact ViT that can be reproduced
and deployed on an ESP32-S3 while retaining useful classification
performance.

------------------------------------------------------------------------

## Research Gap

Existing work has demonstrated hardware-aware ViT optimization and
efficient Tiny Transformer deployment on resource-constrained MCUs.
However, these approaches primarily focus on specialized optimization
frameworks and specific MCU platforms.

This project explores a simplified and reproducible ESP32-S3-oriented
optimization-to-deployment pipeline, with particular emphasis on the
relationship between architecture reduction, INT8 quantization,
accuracy, memory usage, and computational cost.

------------------------------------------------------------------------

## Project Pipeline

``` text
CIFAR-10
   │
   ▼
Baseline ViT
   │
   ├── Accuracy
   ├── Parameters
   ├── FLOPs
   ├── Weight Memory
   └── Activation Memory
   │
   ▼
Memory-Aware Architecture Search
   │
   ├── Patch configuration
   ├── Hidden dimension
   ├── Attention heads
   ├── Transformer depth
   └── MLP dimension
   │
   ▼
Candidate Filtering
   │
   ▼
Proxy Training
   │
   ▼
C201 Compact ViT
   │
   ▼
Full FP32 Training
   │
   ▼
INT8 Weight Quantization
   │
   ▼
C/C++ Tensor Export
   │
   ▼
C/C++ Inference Engine
   │
   ▼
ESP-IDF
   │
   ▼
Wokwi Validation
   │
   ▼
Physical ESP32-S3
   │
   ▼
Embedded Benchmarking
```


## Current Progress

- Baseline ViT implemented and trained on CIFAR-10.
- Memory-aware architecture search completed.
- C201 selected as the compact architecture.
- C201: **124,714 parameters**, **19.19 MFLOPs**, **76.93% FP32 accuracy**.
- INT8 quantization completed: **0.118937 MB** weight memory vs **0.475746 MB** in FP32.
- INT8 accuracy: **76.76%** with a **0.17 percentage-point drop**.
- ESP32-S3 C/C++ deployment is currently in progress.

## Next Steps

- [ ] C/C++ INT8 inference engine
- [ ] Memory-efficient buffer management
- [ ] ESP-IDF integration
- [ ] Wokwi validation
- [ ] Physical ESP32-S3 deployment
- [ ] Latency, RAM and flash benchmarking

## References

- MCUFormer: Deploying Vision Transformers on Microcontrollers with Limited Memory — arXiv:2310.16898v3
- Optimizing the Deployment of Tiny Transformers on Low-Power MCUs — arXiv:2404.02945v1
