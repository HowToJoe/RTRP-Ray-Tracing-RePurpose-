# RTRP — Ray Tracing RePurpose

**RTRP (Ray Tracing RePurpose)** is an experimental research project investigating when NVIDIA RTX ray-tracing acceleration can be repurposed for non-rendering spatial computation.

The project uses **NVIDIA OptiX BVH traversal** alongside conventional **CUDA spatial and brute-force baselines**, with controlled workloads, separated timing, correctness validation, and reproducible benchmark data.

> **Research question:** When does BVH-based RTX ray-tracing acceleration outperform conventional CUDA approaches for spatial queries?

## Overview

RTRP evaluates ray/AABB spatial queries using three main approaches:

* **OptiX / BVH** — RTX ray-tracing pipeline with an acceleration structure
* **CUDA 3D grid** — GPU spatial partitioning using a uniform 3D grid
* **Brute-force CUDA** — direct ray/primitive testing without spatial acceleration

The project measures construction, host-to-device transfer, query execution, device-to-host transfer, and end-to-end execution separately where applicable.

Correctness is checked against CPU reference implementations using deterministic workloads and `hit / id / t` comparisons with a tolerance of `±1e-3`.

## Key Results

Results below are empirical measurements from an **NVIDIA RTX 3070 Ti**. They describe the tested implementations and workloads; they are not claims about RTX hardware in general.

### 1M rays / 2,000 primitives

For the original brute-force comparison:

| Implementation   |  Query / Kernel |
| ---------------- | --------------: |
| OptiX / BVH      |    **302.9 µs** |
| Brute-force CUDA | **15,913.3 µs** |

This corresponds to approximately **52.5× lower kernel time** for the OptiX implementation under the stated benchmark conditions.

However, end-to-end execution is much closer because data transfers and setup costs dominate at this scale.

### Scattered workload

For a tested scattered workload, the OptiX/BVH implementation outperformed the 3D CUDA grid:

| Implementation | Query / Kernel |
| -------------- | -------------: |
| OptiX / BVH    |    **23.5 µs** |
| CUDA 3D grid   |    **84.4 µs** |

### Coherent workload

The result changes for highly structured/coherent workloads:

| Implementation | Query / Kernel |
| -------------- | -------------: |
| OptiX / BVH    |      **58 µs** |
| CUDA 3D grid   |      **47 µs** |

This demonstrates that the OptiX/BVH approach is **not universally faster**. Workload structure and spatial distribution affect which acceleration strategy performs better.

## Important Interpretation

RTRP does **not** demonstrate that "RT cores are 52× faster than CUDA."

The measurements compare different computational strategies:

> **BVH-accelerated OptiX traversal vs. brute-force CUDA**

The observed advantage comes from spatial acceleration and pruning of candidate intersections. Programmable OptiX shader stages still execute on the GPU's programmable compute hardware.

RTRP also does **not** establish `O(log N)` complexity. The project reports observed scaling behavior over tested workloads rather than making an asymptotic complexity claim.

Finally, query-time improvements do not necessarily translate directly into end-to-end improvements. PCIe transfers can dominate total runtime, particularly when data is repeatedly moved between the CPU and GPU.

## Timing Methodology

Where supported, timing is separated into:

```text
AS build
H2D transfer
kernel / OptiX launch
D2H transfer
end-to-end
```

This separation is important because comparing an accelerated query with a brute-force kernel alone can hide the cost of building structures and moving data.

Repeated-query experiments also examine **amortization**, where acceleration-structure construction can be reused across multiple queries.

## Correctness

Correctness is validated against CPU reference implementations.

For large workloads, full CPU validation can become prohibitively expensive. In those cases RTRP uses explicitly labeled **sampled validation** rather than claiming full exhaustive validation.

Example:

> 1M rays × 2,000 primitives = 2 billion potential ray/primitive tests for a brute-force CPU reference.

The benchmark documentation identifies which results use full validation and which use sampled validation.

## Dataset & Reproducibility

The complete benchmark dataset is included in the repository:

**[`experiments/full/FULL_BENCHMARKS.csv`](experiments/full/FULL_BENCHMARKS.csv)**

The accompanying methodology and reproduction instructions are here:

**[`experiments/full/METHOD_FULL.md`](experiments/full/METHOD_FULL.md)**

Raw per-executable benchmark CSVs are also included under `experiments/full/`.

Earlier benchmark datasets are preserved separately and are not overwritten by the full benchmark collection.

## Hardware / Software

Current benchmark environment:

* **GPU:** NVIDIA GeForce RTX 3070 Ti
* **GPU architecture:** Ampere / SM 8.6
* **CUDA:** 13.4
* **OptiX:** 9.1.0
* **Compiler:** Microsoft Visual C++
* **CMake:** 4.3+
* **OS:** Windows

Results are hardware- and implementation-dependent and should not be interpreted as universal performance claims.

## Build

Configure with CMake:

```powershell
cmake -S . -B build -A x64 -DOptiX_ROOT="C:/ProgramData/NVIDIA Corporation/OptiX SDK 9.1.0"
```

Build:

```powershell
cmake --build build --config Release
```

## Basic Usage

Validation:

```powershell
build/Release/rtrp_app.exe --validate
```

Benchmark both implementations:

```powershell
build/Release/rtrp_app.exe --benchmark --both
```

See the benchmark methodology for the complete set of workloads and reproduction commands.

## Project Structure

```text
RTRP/
├── src/                    # CUDA / OptiX implementations
├── include/                # Project headers
├── experiments/
│   ├── results/            # Earlier benchmark datasets
│   ├── three_way/          # CUDA grid / OptiX comparisons
│   └── full/               # Consolidated benchmark dataset + methodology
├── docs/                   # Additional documentation
└── README.md
```

## Limitations

RTRP is an experimental project rather than a production spatial-query library.

Important limitations include:

* Results depend on the specific RTX GPU, CUDA/OptiX versions, workload, and implementation.
* The CUDA grid is a research baseline, not a claim to represent every possible CUDA spatial data structure.
* Large workloads may use sampled correctness validation because exhaustive CPU validation becomes impractical.
* PCIe transfer overhead can substantially reduce end-to-end gains.
* The experiments do not establish asymptotic complexity.
* The project does not claim that RT cores provide general-purpose arithmetic comparable to CUDA cores.

## Prior Work

RTRP builds on existing work investigating the use of NVIDIA RTX ray-tracing hardware for non-rendering computation and spatial workloads.

Relevant prior work includes projects and research involving OptiX, BVH traversal, ray-based spatial queries, and the use of RT hardware for general computation.

RTRP's contribution is primarily **experimental and comparative**: implementing and evaluating several spatial-query strategies under controlled workloads and documenting where each approach succeeds or falls short.

## Status

**RTRP v1.1**

Current focus:

* OptiX/BVH spatial traversal
* CUDA brute-force baselines
* CUDA 3D grid acceleration
* workload-dependent performance
* transfer and end-to-end costs
* repeated-query amortization
* reproducible benchmark data
