#Introduction

High-Performance Computing (HPC) is a cornerstone of modern scientific research, enabling simulations of complex physical systems that would be otherwise impossible to model in real time. However, the operation of HPC systems comes with substantial energy costs, which can become a limiting factor, particularly when working with repurposed legacy hardware. While such hardware carries minimal acquisition costs, the energy required to run simulations often dominates operational expenses. Consequently, understanding how to maximize computational productivity while minimizing energy consumption is critical for sustainable and cost-effective HPC practices.

This research aims to map the productivity-to-energy cost ratio for different scientific applications running on repurposed legacy HPC hardware. In this context, productivity is defined as the rate at which useful computational work is completed, measured through application-specific metrics such as iterations per second or timesteps per second. Energy cost refers to the total electrical energy consumed during computation, measured in joules or watt-hours. Both productivity and energy consumption are influenced by several factors, including CPU frequency, parallelization strategy, memory bandwidth, solver configurations, and thread/process placement.

To explore this relationship, the study focuses on two widely used HPC applications that represent contrasting computational workloads:

OpenFOAM, a computational fluid dynamics (CFD) framework that is primarily memory-bound, stressing the memory hierarchy, cache system, and interconnect.

LAMMPS, a molecular dynamics (MD) simulator that is primarily compute-bound, stressing floating-point execution units and CPU frequency scaling.

By selecting these applications, the study spans the spectrum of HPC workloads, providing insight into how memory-bound and compute-bound algorithms respond to system-level tuning, including CPU frequency scaling, parallelization strategies, and thread/process placement.

The experiments are conducted on the Lengau Cluster, a high-performance computing platform designed to support a variety of scientific workloads. Using this infrastructure allows us to measure real-world performance and energy characteristics of legacy hardware under controlled conditions, providing practical insights into achieving energy-efficient computation without sacrificing productivity. The results of this study will guide HPC practitioners in optimizing repurposed hardware for sustainable, high-productivity operation, highlighting the trade-offs between computational throughput and energy consumption.
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

## 2. Aim of Running These Test Cases
The primary objective of running both the OpenFOAM simpleFoam solver and the LAMMPS 3D Lennard-Jones (LJ) melt benchmark is to evaluate how distinct HPC workload types respond to system-level tuning, and to quantify how these responses influence the productivity-to-energy cost ratio. By selecting a memory-bound application (OpenFOAM) and a compute-bound application (LAMMPS), the study provides a comprehensive view across the spectrum of HPC workloads.

Specifically, the aims of these experiments are as follows:

 - Measure the Trade-Off Between Performance and Energy Consumption

  By systematically adjusting key system parameters—including CPU frequency, parallelization strategy, and process/thread placement—we aim to understand their impact on:

 -Execution Time: How long the workload takes to complete under different configurations.

-Power Draw: The instantaneous and average electrical power consumed during execution.

 -Total Energy Consumed: The cumulative energy used, integrating power over time.

 -Work Done per Joule: A measure of efficiency, indicating how much computational work is accomplished per unit of energy consumed.

Through these measurements, the study identifies configurations that provide an optimal balance between performance and energy efficiency. This approach allows HPC practitioners to make informed decisions about tuning legacy hardware for sustainable, cost-effective scientific computation.

## 3. Measuring Computational Efficiency and Power Consumption

o accurately quantify the productivity-to-energy ratio, it is crucial to measure both computational performance and power consumption in a precise and repeatable manner. In this study, we focus on CPU-level power monitoring using the Running Average Power Limit (RAPL) interface, which provides detailed energy readings for modern Intel processors.

1. Computational Efficiency

Computational efficiency is measured as the amount of work completed per unit of energy consumed. For our test cases:

OpenFOAM (simpleFoam): The number of timesteps completed per second, combined with total energy consumption, yields the efficiency in timesteps per joule.

LAMMPS (LJ melt): The number of simulation timesteps per second is recorded, which, when normalized by energy usage, provides efficiency in iterations per joule.

The general formula used is:

Efficiency (work/joule) = Total Work Completed /Total Energy Consumed (J)

his metric allows a direct comparison of different system configurations, CPU frequencies, and parallelization strategies, enabling the identification of the optimal productivity-to-energy point.

2. Power Measurement using RAPL

RAPL (Running Average Power Limit) is an energy monitoring interface built into modern Intel CPUs. It provides highly accurate estimates of the energy consumed at the CPU package level, including cores, caches, and DRAM domains. The key reasons for choosing RAPL in this study are:

High Resolution and Accuracy: RAPL can report energy consumption in microjoules at intervals as short as milliseconds, allowing fine-grained power monitoring throughout the simulation.

Direct CPU-Level Measurement: Unlike system-level power meters or IPMI-based sensors that measure total node power—including fans, storage, and network interfaces—RAPL focuses on the CPU and DRAM, which are the most significant contributors to HPC workload energy consumption. This ensures that the measurement reflects the true energy cost of computation rather than auxiliary subsystems.

Repeatability and Consistency: Being an on-chip interface, RAPL readings are unaffected by external measurement noise or sensor placement, allowing consistent comparisons across runs and configurations.

In practice, the script reads the energy_uj files under /sys/class/powercap/intel-rapl at fixed intervals (e.g., every second), calculates the total energy consumed during the simulation, and combines it with elapsed execution time to compute average power:

Average Power (W) = Total Energy (J) / Elapsed Time (s)

By emphasizing CPU-level power measurement with RAPL, we ensure that the study captures the core energy-performance trade-offs of HPC workloads, providing a robust basis for evaluating productivity-to-energy ratios on legacy HPC hardware such as the Lengau Cluster.  
  
  ​
## 4. Parameters and Metrics Tuned in the OpenFOAM Performance Experiments

During the performance and energy-efficiency evaluation of OpenFOAM, several hardware-level, system-level, and application-level parameters were deliberately tuned. These adjustments allowed us to study their direct impact on runtime, power consumption, and overall efficiency. Below is a breakdown of what was changed, why, and what behaviour it influences.


Test1: We first looked into perfomance 


### Table 1: Tuned System and Application Parameters for Perfomance OpenFOAM Benchmarking

| Parameter / Setting                | Applied Value                     |
|----------------------------------|----------------------------------|
| OMP_PROC_BIND                     | close                             |
| OMP_PLACES                        | cores                             |
| KMP_AFFINITY                       | compact,1,0,granularity=fine     |
| MPI Bind                           | --bind-to core                    |
| MPI Map                            | --map-by socket:PE=$OMP_NUM_THREADS |
| Number of MPI Tasks (SLURM_NTASKS)|   4                      |
| Number of OpenMP Threads (OMP_NUM_THREADS) | 1 (default)                |
| Decomposition Method               | scotch                            |
| RAPL Sampling Interval             | 0.1 s                             |
| UCX Network Devices                | all                               |


To ensure predictable performance and minimize unnecessary overhead, we carefully tuned several thread-affinity and process-mapping parameters that control how OpenMP threads and MPI ranks are placed on CPU cores. First, we enabled OMP_PROC_BIND=close, which prevents OpenMP threads from migrating between cores during execution. This reduces cache invalidations, minimizes thread movement penalties, and maintains consistent locality. In conjunction with this, OMP_PLACES=cores was used to explicitly assign each OpenMP thread to a unique physical CPU core, ensuring optimal use of the L1 and L2 cache hierarchy and preventing threads from competing for the same compute resources. For more granular control, we set KMP_AFFINITY=compact,1,0,granularity=fine, instructing the OpenMP runtime to place threads as closely together as possible within each socket (compact mode), thereby lowering memory-access latency while binding threads to distinct hardware execution units (fine granularity). Finally, MPI process placement was controlled with --bind-to core and --map-by socket:PE=$OMP_NUM_THREADS, which fixes each MPI rank to a specific core while distributing ranks evenly across CPU sockets, with each rank allocated a defined number of processing elements matching its thread count. Together, these affinity settings ensured stable core locality, reduced NUMA effects, improved cache utilization, and produced both more consistent performance and more accurate energy-efficiency measurements across all test runs.


![WhatsApp Image 2025-11-15 at 12 27 34_7c8ac487](https://github.com/user-attachments/assets/e2d39156-166d-4bb1-9bbc-8e2a0655a2c2)

**Figure 1:** The foam.out of the run.

Solver Performance Summary (OpenFOAM Timesteps 41–45)

The solver output between timesteps 61 and 64 indicates stable and well-converged behaviour across all equations. The momentum equations (Ux, Uy, Uz) consistently show initial residuals on the order of 10⁻²–10⁻³, dropping to 10⁻³–10⁻⁴ within only two solver iterations. This reflects efficient convergence of the velocity field.
The pressure equation, solved with GAMG, exhibits initial residuals decreasing from approximately 0.10 to 0.07, with final residuals consistently around 5×10⁻³ to 9×10⁻³. These values fall within the expected range for a transient simulation using multigrid and indicate that the pressure correction step remains stable.
Continuity errors remain small throughout this interval. The global continuity error is on the order of 10⁻⁶, and although the cumulative error drifts slightly (around 8×10⁻⁴ in magnitude), it does not show signs of divergence and remains acceptable for a transient run.
The turbulence equations (k and ε) demonstrate good convergence, with residuals typically reduced from ~10⁻² to ~10⁻⁴–10⁻³, again within only two iterations.
Execution time increases steadily and linearly with timestep, averaging approximately 4 seconds per timestep, indicating consistent computational cost and no solver slowdown.
Overall, the solver output shows stable behaviour, good convergence, low continuity errors, and consistent computational performance, with no signs of numerical instability or divergence.


![WhatsApp Image 2025-11-15 at 07 42 21_6759b0d3](https://github.com/user-attachments/assets/d687f604-0314-4a4b-95bc-565062f027ca)
Thread and Core Binding

OMP_PROC_BIND=close and OMP_PLACES=cores ensure that each OpenMP thread is bound to a specific core.

KMP_AFFINITY=compact,1,0,granularity=fine further enforces that threads are packed close together on cores, minimizing cross-core memory access.

MPI --bind-to core and --map-by socket:PE=$OMP_NUM_THREADS ensure each MPI rank is bound to cores in a controlled fashion, mapping ranks to sockets efficiently.


This enabled us to identify the performance sweet spot and the energy-optimal point for each application.


### Test 2: Balanced Performance Mode Configuration (2.4 GHz DVFS Setting)

To achieve a balanced operating mode—one that provides good performance at significantly reduced power draw—the script includes a CPU frequency control section that forces the processors to run at a fixed mid-range frequency of 2.4 GHz. This frequency was selected because it typically represents the “knee” of the DVFS curve:

Lower frequencies reduce power but often degrade time-to-solution sharply.
Higher frequencies improve performance but increase power disproportionately.
A mid-range value like 2.4 GHz provides an excellent compromise.

For each CPU core, the script:
   -Reads available hardware frequencies
   -Finds the closest value to 2.4 GHz
   -Writes that value to the CPU’s scaling_setspeed file
   
We also set the OMP_PROC_BIND=close this ensures that  OpenMP threads stay on their assigned cores.
OMP_PLACES=cores Each thread is placed on a physical core to avoid SMT interference.

KMP_AFFINITY=compact,granularity=fine this Helps threads share cache efficiently and reduce memory latency.
MPI mapping
OpenFOAM Solver-Level Tunings for Energy Stability
At fixed frequency, some solver parameters can reduce unnecessary iterations.
We Increase under-relaxation slightly to 0.3 → 0.5 for U to ensure a faster convergence and less iteration time.We configured the MpI rank to reduce the communication overhead 


2. Compare Memory-Bound vs Compute-Bound Behavior

By using one memory-bound and one compute-bound test case, we aimed to determine:

Which parameters influence each workload most strongly
Whether optimal energy-efficient settings differ between CFD and MD simulations
How parallelization strategies and CPU frequency scaling shift between the two application types


## Results: Productivity vs. Energy Efficiency

To identify the optimal balance between computational speed and power consumption, we tested OpenFOAM under several fixed CPU clock frequencies. Table 1 summarizes the results, showing the effect of frequency scaling on iterations per second, average power draw, and energy efficiency. At the maximum turbo frequency of 3.5 GHz, the solver achieved 250 iterations per second, but at a high average power of 60.25 W, resulting in relatively low energy efficiency (0.0166 iterations per Watt, or 90% relative efficiency). Reducing the CPU frequency to 2.4 GHz maintained the same iteration rate while significantly reducing power consumption to 28.99 W. This setting provided the best overall energy efficiency (0.0344 iterations per Watt), which we considered 100% relative efficiency and the “balanced-performance” point. At a further reduced frequency of 2.0 GHz, the solver’s iteration rate remained unchanged, power draw dropped slightly to 26.39 W, and energy efficiency increased marginally (0.0379 iterations per Watt). However, the reduced frequency did not yield substantial performance gains relative to 2.4 GHz and could increase runtime for larger, more complex cases. These results confirm that 2.4 GHz represents the optimal compromise between maintaining computational throughput and minimizing power consumption.


| CPU Frequency (GHz) | Iterations/s | Avg Power (Watts) | Energy Efficiency (Iter/s per Watt) | Relative Efficiency | Iterations Per Joule
|--------------------|--------------|-----------------|------------------------------------|------------------|----------------------|
| Max Turbo (3.5)    | 250          |   60.25 W       | 0.0166 I/W                         | 90%              | 0,0040               |
| Optimal (2.4)      | 250          | 28.99 W         | 0,0344                             | 100%             |0.0049               |
| Low (2.0)          | 250          |  26.39 W        | 0.0379                             | 98%              |0,0070               |
 
 Table 2 : Showing The avarage power and Iteration per Joules in 3 different CPU frequencies 

## Test 3 : Power Save 

![WhatsApp Image 2025-11-15 at 07 42 20_25006310](https://github.com/user-attachments/assets/1e5d7664-de89-41c1-8643-00cda77bd8e8)


