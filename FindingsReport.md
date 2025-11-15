# Introduction

High-Performance Computing (HPC) plays a pivotal role in advancing scientific research, enabling the simulation of complex physical systems that would otherwise be impossible to model in real time. However, running these large-scale simulations comes with a significant—and often prohibitive—energy cost. This challenge becomes even more critical when working with repurposed legacy HPC hardware, where acquisition costs are negligible, but power consumption becomes the dominant operational expense. Therefore, understanding how to maximize computational productivity while minimizing energy use is essential for sustainable, cost-effective HPC operations.
To investigate this balance between performance and energy efficiency, this study focuses on two widely used scientific applications that represent contrasting computational workloads:

- **OpenFOAM,** a computational fluid dynamics (CFD) framework that is primarily memory-bound, stressing the memory hierarchy, cache system, and interconnect.
 -**LAMMPS**, a molecular dynamics (MD) simulator that is mostly compute-bound, stressing floating-point execution units and CPU frequency scaling.
By selecting these two applications, the study covers both ends of the HPC workload spectrum, enabling a deeper understanding of how different types of algorithms respond to changes in system-level parameters such as CPU frequency, parallelization strategy, and thread/process binding.

## Test Cases Used in the Study

#### 1. OpenFOAM: simpleFoam Steady-State Test Case

For OpenFOAM, the chosen benchmark was the simpleFoam solver—a steady-state, incompressible, turbulent flow case commonly used for CFD performance evaluation. This test case is ideal because:
It is sensitive to memory bandwidth and cache locality, making it an excellent representative of memory-bound workloads.
It uses iterative linear solvers whose convergence behavior can vary dramatically with CPU frequency, decomposition strategy, and solver settings.
It exhibits predictable scaling behavior, which is crucial for isolating the impact of system parameter tuning.

Using simpleFoam allowed us to observe how memory-bound CFD computations respond to changes such as:

  1. CPU clock scaling
  2. MPI process decomposition
  3. NUMA memory placement
  4. Solver relaxation factors and residual controls

#### 2. LAMMPS: 3D Lennard-Jones Melt Test Case

For LAMMPS, we selected the well-established 3D Lennard-Jones (LJ) melt benchmark. This test case simulates an atomic fluid interacting under the Lennard-Jones potential—a standard MD model used extensively in HPC performance studies.
  
This case was chosen because:
  1. It is primarily compute-intensive, relying on heavy floating-point operations for force calculations.
  2. It allows clear measurement of timesteps per second, a reliable proxy for raw computational throughput.
  3.It is sensitive to CPU frequency, vectorization, thread affinity, and hybrid MPI/OpenMP parallelization.
  4. It is widely used in HPC benchmarking literature, enabling comparison with known scaling patterns.

Using the LJ melt benchmark enabled us to quantify how compute-bound workloads behave under:
-High vs. reduced CPU frequencies
-Fine-grained vs. coarse-grained parallelization
-Process/thread affinity under NUMA conditions
-Hybrid MPI + OpenMP execution models

## Aim of Running These Test Cases

The objective of running both simpleFoam and the LJ melt benchmark was to evaluate how different HPC workload types respond to system-level tuning, and how these responses translate into productivity-to-energy cost ratios. Specifically, the aims were:

### 1.Measure the Trade-off Between Performance and Energy Consumption

We manipulated CPU frequency, parallelization strategy, and execution layout to observe how each parameter affects:
-Execution time
-Power draw
-Total energy consumed
-Work done per joule

## Parameters and Metrics Tuned in the OpenFOAM Performance Experiments

During the performance and energy-efficiency evaluation of OpenFOAM, several hardware-level, system-level, and application-level parameters were deliberately tuned. These adjustments allowed us to study their direct impact on runtime, power consumption, and overall efficiency. Below is a breakdown of what was changed, why, and what behaviour it influences.




We first looked into perfomance 


### Table 1: Tuned System and Application Parameters for Perfomance OpenFOAM Benchmarking

| Parameter / Setting                     | Applied Value                            | Purpose / Role                                         | Reason / Expected Effect                                                                 |
|----------------------------------------|------------------------------------------|--------------------------------------------------------|------------------------------------------------------------------------------------------|
| `SLURM_NTASKS`                          | 4 (default, adjustable per test)         | Number of MPI processes                                | Controls parallel decomposition; balances workload across CPU cores                       |
| `OMP_NUM_THREADS`                       | 1                                        | Threads per MPI process                                 | Limits OpenMP thread count to reduce oversubscription and simplify core mapping           |
| `OMP_PROC_BIND`                          | `close`                                  | Prevents OpenMP threads from moving between cores      | Reduces cache misses and improves data locality                                          |
| `OMP_PLACES`                             | `cores`                                  | Assigns threads to physical CPU cores                  | Ensures L1/L2 cache reuse and prevents thread contention                                 |
| `KMP_AFFINITY`                           | `compact,1,0,granularity=fine`          | Controls how Intel OpenMP threads are placed           | Compact placement reduces memory latency; fine granularity binds threads to individual cores |
| MPI `--bind-to core`                      | Enabled                                  | Fixes each MPI rank to a physical core                 | Avoids process migration and improves cache locality                                     |
| MPI `--map-by socket:PE=$OMP_NUM_THREADS`| Socket mapping per thread count          | Distributes MPI ranks across sockets                   | Reduces cross-socket memory access; aligns threads with NUMA-local memory                |
| Decomposition Method (`decomposeParDict`)| `scotch`                                 | Determines domain decomposition for parallel mesh      | Minimizes processor boundaries and balances load across subdomains                       |
| Memory Settings (`cacheCoherent`, `memory`)| `0.8`, `low`                             | Hardware-specific cache/memory optimization            | Reduces memory overhead and improves cache utilization for solver performance            |
| Power Monitoring Interval (`RAPL_INTERVAL`)| 0.1 s                                    | Frequency of energy measurement                        | Provides high-resolution energy consumption data for productivity-to-energy calculations |




Tkjjo ensure predictable performance and minimize unnecessary overhead, we carefully tuned several thread-affinity and process-mapping parameters that control how OpenMP threads and MPI ranks are placed on CPU cores. First, we enabled OMP_PROC_BIND=close, which prevents OpenMP threads from migrating between cores during execution. This reduces cache invalidations, minimizes thread movement penalties, and maintains consistent locality. In conjunction with this, OMP_PLACES=cores was used to explicitly assign each OpenMP thread to a unique physical CPU core, ensuring optimal use of the L1 and L2 cache hierarchy and preventing threads from competing for the same compute resources. For more granular control, we set KMP_AFFINITY=compact,1,0,granularity=fine, instructing the OpenMP runtime to place threads as closely together as possible within each socket (compact mode), thereby lowering memory-access latency while binding threads to distinct hardware execution units (fine granularity). Finally, MPI process placement was controlled with --bind-to core and --map-by socket:PE=$OMP_NUM_THREADS, which fixes each MPI rank to a specific core while distributing ranks evenly across CPU sockets, with each rank allocated a defined number of processing elements matching its thread count. Together, these affinity settings ensured stable core locality, reduced NUMA effects, improved cache utilization, and produced both more consistent performance and more accurate energy-efficiency measurements across all test runs.


![WhatsApp Image 2025-11-15 at 07 42 21_6759b0d3](https://github.com/user-attachments/assets/d687f604-0314-4a4b-95bc-565062f027ca)





This enabled us to identify the performance sweet spot and the energy-optimal point for each application.
2. Compare Memory-Bound vs Compute-Bound Behavior

By using one memory-bound and one compute-bound test case, we aimed to determine:

Which parameters influence each workload most strongly
Whether optimal energy-efficient settings differ between CFD and MD simulations
How parallelization strategies and CPU frequency scaling shift between the two application types

3. Develop a Framework for Energy-Aware HPC Benchmarking

The goal was not just to collect data but to establish a reproducible methodology that future users of repurposed hardware can apply, including:
   -How to tune system parameters using Slurm job scripts
   -How to measure power consumption in real time
   -How to generate reliable productivity-per-watt metrics
   -How to interpret scaling curves in an energy-aware context

4. Provide Practical Guidance for Running Scientific Codes on Legacy Hardware

Legacy HPC systems often have lower efficiency per watt than modern hardware. However, by tuning:

CPU frequency governors
MPI process placement
Thread affinity and NUMA policy
Solver tolerances
Domain decomposition

it is still possible to run them cost-effectively.

The aim of this research was to deliver actionable, data-driven recommendations for how to operate legacy clusters at minimum energy cost while maintaining acceptable scientific throughput.
