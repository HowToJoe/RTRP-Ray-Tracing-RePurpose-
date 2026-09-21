# RTRP-Ray-Tracing-RePurpose-
RTRP (Ray Tracing RePurpose): OptiX BVH + CUDA hybrid for spatial search on RTX 3070 Ti. Validated OptiX vs brute-force with separated kernel/e2e timing. Empirical, no RT-core overclaims.


What it is: deterministic AABB/ray harness with CPU-validated correctness (`hit/id/t ±1e-3`), separated `as,h2d,launch,d2h,e2e` timing, and brute-force CUDA baselines. At 2M/2000: OptiX launch `573 us` vs CUDA kernel `28361 us` (~49x kernel, ~1.3x e2e) — empirical BVH pruning, not `O(log N)` proof, not RT-core arithmetic.

Build: `cmake -S . -B build -A x64 -DOptiX_ROOT="C:/ProgramData/NVIDIA Corporation/OptiX SDK 9.1.0"` then `cmake --build build --config Release`. Run `build/Release/rtrp_app.exe --validate`, `--benchmark --both`.
