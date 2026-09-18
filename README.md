# MCUFormer-ESP32

## A Memory-Aware Vision Transformer for ESP32-S3

MCUFormer-ESP32 is an ongoing B.Tech major project focused on optimizing
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

------------------------------------------------------------------------

## Current Results

### Baseline ViT

The initial baseline model was trained on CIFAR-10 for 120 epochs.

  Metric                            Baseline
  -------------------------- ---------------
  Parameters                          6.27 M
  FP32 weight memory                23.91 MB
  INT8 weight memory                 5.98 MB
  Total FLOPs                  865.17 MFLOPs
  Peak activation memory           883.59 KB
  Input resolution                   32 × 32
  Patch size                           4 × 4
  Tokens                                  65
  Embedding dimension                    384
  Attention heads                         12
  Transformer depth                        7
  Best validation accuracy            89.24%

The baseline model is substantially larger than the intended MCU memory
budget, motivating the architecture search.

### Memory-Aware Architecture Search

A constrained architecture search was performed over different ViT
configurations. The search considered patch configuration, hidden
dimension, attention heads, Transformer depth, and MLP dimension.

A peak activation-memory budget of **256 KB** was used as the
feasibility constraint.

From the valid search space, six representative candidates were selected
for 15-epoch proxy training.

  ------------------------------------------------------------------------------------------------------------
  Candidate     Patches   Tokens   Hidden   Heads   Depth      MLP   Parameters   MFLOPs         Peak    Proxy
                                                            Hidden                         Activation     Val.
                                                                                                          Acc.
  ----------- --------- -------- -------- ------- ------- -------- ------------ -------- ------------ --------
  C201               64       65       96       4       2       96      124,714    19.19    253.91 KB   60.34%

  C225               64       65      128       2       2      128      215,434    31.84    228.52 KB   60.01%

  C161               16       17      384      12       2      384    1,862,794    64.30    154.59 KB   58.27%

  C193               64       65       96       2       2       96      124,714    19.11    187.89 KB   57.32%

  C121               16       17      256       4       2      256      848,650    29.49     94.03 KB   56.74%

  C073               16       17      128       8       2      128      227,722     8.08     60.56 KB   56.28%
  ------------------------------------------------------------------------------------------------------------

### Selected Architecture: C201

C201 retains the basic ViT structure while significantly reducing its
dimensions.

  Component                Original ViT      C201
  ---------------------- -------------- ---------
  Image size                    32 × 32   32 × 32
  Patches per side                    8         8
  Patch size                      4 × 4     4 × 4
  Image patches                      64        64
  Tokens                             65        65
  Hidden dimension                  384        96
  Attention heads                    12         4
  Transformer depth                   7         2
  MLP hidden dimension              384        96
  Classes                            10        10

C201 contains **124,714 parameters**, requires **19.19 MFLOPs**, and has
an estimated peak activation memory of **253.91 KB**, fitting within the
256 KB search constraint.

The architecture was fully trained after selection and achieved **76.93%
test accuracy**.

> Note: The 60.34% value is the 15-epoch proxy-training accuracy. The
> final fully trained C201 accuracy is 76.93%.

------------------------------------------------------------------------

## INT8 Quantization

The trained C201 FP32 model is quantized before embedded deployment.

The current quantization pipeline consists of:

1.  Activation calibration using 512 representative CIFAR-10 training
    images.
2.  Collection of ranges for 23 intermediate activations.
3.  Per-tensor INT8 weight quantization.
4.  Storage of the corresponding weight scaling factors.
5.  Evaluation against the original FP32 model.

The weight quantization scheme is:

``` text
s_x = max(|W|) / 127

W_INT8 = round(W_FP32 / s_x)
```

### Quantization Results

  Metric              FP32 C201     INT8 C201
  --------------- ------------- -------------
  Parameters            124,714       124,714
  Test samples           10,000        10,000
  Test accuracy          76.93%        76.76%
  Test loss            0.725030      0.725127
  Weight memory     0.475746 MB   0.118937 MB

INT8 representation provides approximately **4× weight-storage
compression** and a **75% reduction in raw weight memory**, while the
measured test accuracy decreases by only **0.17 percentage points**.

The reported maximum absolute quantization error is **0.010325**, with a
mean absolute error of **0.001162**.

------------------------------------------------------------------------

## Embedded Inference Design

The embedded implementation is being developed as a C/C++ inference
engine for the ESP32-S3.

The intended inference flow is:

``` text
Input Image
    │
    ▼
Preprocessing
    │
    ▼
4 × 4 Patch Extraction
    │
    ▼
Patch Embedding
    │
    ▼
CLS Token + Positional Embedding
    │
    ▼
Transformer Block × 2
    │
    ├── LayerNorm
    ├── Q / K / V
    ├── Attention
    ├── Projection
    ├── Residual
    ├── LayerNorm
    ├── MLP + GELU
    └── Residual
    │
    ▼
Final LayerNorm
    │
    ▼
Classifier (96 → 10)
    │
    ▼
Argmax
    │
    ▼
Predicted CIFAR-10 Class
```

------------------------------------------------------------------------

## Memory Management

Because activation memory is a major constraint on microcontrollers, the
embedded implementation is designed around static and reusable buffers.

The current memory-management strategy focuses on:

-   Static activation buffers
-   Reusable Q/K/V buffers
-   Reusable attention buffers
-   Buffer reuse between Transformer operations
-   Reduction of peak RAM consumption
-   Avoidance of unnecessary dynamic allocations

The final implementation will measure actual RAM consumption on the
ESP32-S3 and compare it with the estimated memory requirements used
during architecture selection.

------------------------------------------------------------------------

## Embedded Export

The trained INT8 model is exported from the PyTorch environment into
C/C++ compatible data structures.

The planned exported files include:

``` text
weights.h
config.h
```

`weights.h` contains INT8 model weights and their associated scaling
factors.

`config.h` contains model dimensions and quantization-related
configuration required by the embedded inference engine.

------------------------------------------------------------------------

## ESP-IDF Integration

The C/C++ inference application will be integrated into an ESP-IDF
project and built into firmware for the ESP32-S3.

The deployment stage will evaluate:

-   Successful model execution
-   Prediction correctness
-   Flash usage
-   RAM usage
-   Inference latency

------------------------------------------------------------------------

## Wokwi Validation

Before physical deployment, the ESP32-S3 implementation will be
validated in Wokwi.

The planned validation flow is:

``` text
ESP32-S3 Simulation
       │
       ▼
Load Model Weights
       │
       ▼
Run Inference
       │
       ▼
Serial Prediction
       │
       ▼
Functional Validation
```

Wokwi is intended as an intermediate validation stage before running the
inference engine on physical ESP32-S3 hardware.

------------------------------------------------------------------------

## Cross-Validation

To ensure that the embedded implementation reproduces the PyTorch model
correctly, the same input will be evaluated across multiple
implementations:

``` text
Same Input
   │
   ├── PyTorch FP32
   │
   ├── PyTorch INT8
   │
   └── C/C++ / Wokwi
```

Validation will include:

-   Final prediction matching
-   Intermediate tensor matching where applicable
-   Quantization consistency
-   Numerical differences introduced by the embedded implementation

------------------------------------------------------------------------

## Benchmarking

The final stage will benchmark the model on the ESP32-S3.

The primary measurements are:

  Metric              Target Measurement
  ------------------- -----------------------------------
  Flash usage         ESP32-S3 firmware/model footprint
  RAM usage           Runtime memory consumption
  Inference latency   Time required for one inference
  Model size          INT8 weight storage
  Accuracy            Comparison with PyTorch reference

The final benchmark results will be added to this README as the hardware
implementation progresses.

------------------------------------------------------------------------

## Repository Structure

The repository is expected to evolve toward a structure similar to:

``` text
MCUFormer-ESP32/
│
├── README.md
│
├── baseline/
│   ├── model/
│   ├── training/
│   └── evaluation/
│
├── architecture_search/
│   ├── search.py
│   ├── candidates/
│   └── results/
│
├── c201/
│   ├── model/
│   ├── checkpoints/
│   └── evaluation/
│
├── quantization/
│   ├── calibration/
│   ├── quantize.py
│   └── evaluation/
│
├── export/
│   ├── weights.h
│   └── config.h
│
├── esp32_vit/
│   ├── main/
│   ├── CMakeLists.txt
│   └── sdkconfig
│
├── wokwi/
│   └── simulation/
│
└── benchmarks/
    └── results/
```

The exact repository structure may change as the embedded implementation
develops.

------------------------------------------------------------------------

## Development Roadmap

### Completed

-   [x] Literature survey
-   [x] Baseline ViT implementation
-   [x] Baseline CIFAR-10 training
-   [x] Parameter, FLOP, and activation-memory analysis
-   [x] Memory-aware architecture search
-   [x] Candidate proxy training
-   [x] C201 architecture selection
-   [x] Full C201 FP32 training
-   [x] INT8 weight quantization
-   [x] Quantized model evaluation
-   [x] Initial embedded deployment design

### In Progress

-   [ ] C/C++ inference engine
-   [ ] INT8 embedded kernels
-   [ ] Static/reusable memory-buffer implementation
-   [ ] PyTorch-to-C/C++ tensor export
-   [ ] ESP-IDF integration
-   [ ] Wokwi inference validation

### Planned

-   [ ] Cross-validation between PyTorch and C/C++
-   [ ] Intermediate tensor verification
-   [ ] Physical ESP32-S3 deployment
-   [ ] Flash usage measurement
-   [ ] Runtime RAM measurement
-   [ ] Inference latency benchmarking
-   [ ] Final accuracy and embedded-performance analysis

------------------------------------------------------------------------

## References

1.  **MCUFormer: Deploying Vision Transformers on Microcontrollers with
    Limited Memory**\
    arXiv:2310.16898v3

2.  **Optimizing the Deployment of Tiny Transformers on Low-Power
    MCUs**\
    arXiv:2404.02945v1

------------------------------------------------------------------------

## Project Scope

This project is intended as a practical study of the complete path from
a conventional Vision Transformer to an MCU-oriented implementation:

``` text
Model Design
     ↓
Memory-Aware Optimization
     ↓
Quantization
     ↓
Embedded Model Export
     ↓
C/C++ Inference
     ↓
ESP-IDF
     ↓
ESP32-S3
     ↓
Hardware Benchmarking
```

The final outcome will be evaluated not only by model accuracy, but also
by the resource and runtime characteristics required for actual
microcontroller deployment.
