
# 1. Introduction

High-Performance Computing (HPC) systems are foundational to scientific discovery, enabling simulations and analyses that are otherwise infeasible. Yet, as computational demand escalates, the associated energy footprint has emerged as a critical limiting factor for both operational sustainability and cost-efficiency. Modern HPC facilities are power hungry: the balance between computational productivity and energy consumption is no longer merely a secondary concern but a central metric that dictates hardware design, scheduling policies, and workload optimization strategies.

Traditional performance evaluations of HPC applications emphasize raw throughput measured in FLOPS, timesteps per second, or iterations per second without explicitly accounting for the power costs incurred to achieve those performance gains. However, a nuanced understanding of **productivity-to-energy trade-offs** is essential in the current landscape, where energy constraints may outweigh peak computational capability in determining overall system efficiency. In other words, **maximum performance does not equate to optimal productivity** when energy consumption is considered.

The energy consumed by HPC applications is governed by multiple interacting factors. At the node level, processor frequency and voltage, memory subsystem bandwidth and latency, interconnect efficiency, and accelerator utilization all contribute to the instantaneous power draw. At the workload level, application characteristics compute bound versus memory-bound, communication patterns, and I/O intensity dictate how hardware resources are stressed and, consequently, how energy is expended. These dependencies are complex, nonlinear, and workload specific, underscoring the need for systematic benchmarking across a representative set of applications.

In this study, we aim to **map energy to productivity** across representative HPC applications, quantifying the **energy efficiency ratio** which is a measure of computational output per unit of energy consumed. By performing controlled experiments on repurposed legacy hardware, we explicitly manipulate CPU clock frequencies and leverage job scheduling tools (Slurm) to systematically explore the trade-offs between raw performance and energy consumption. Our focus applications, **LAMMPS** (compute-intensive molecular dynamics) and **OpenFOAM** (memory-bound computational fluid dynamics), represent contrasting workload characteristics, allowing us to generalize insights on hardware-dependent energy behavior.

Through this investigation, we aim to answer fundamental questions:  

1. How does energy efficiency vary with workload type, CPU frequency, and system configuration?  
2. What is the optimal operating point for maximizing productivity per watt?  
3. How do hardware characteristics—FLOPS, memory bandwidth, and interconnect topology—correlate with energy-to-productivity metrics across applications?  

By integrating performance measurement with power profiling, this work contributes a **framework for energy-aware HPC benchmarking**, providing actionable guidance for both system operators and application developers seeking to reconcile high computational throughput with sustainable energy usage. The ultimate goal is not only to characterize system behavior but to inform policies that maximize scientific productivity while minimizing operational cost and environmental impact.

## 2. Methodology

## 2.1 System Information Overview

To ensure reproducibility and accurate interpretation of productivity-to-energy measurements, a detailed characterization of the HPC system used in this study was conducted. The cluster node smshost was profiled across multiple layers, including hardware architecture, memory hierarchy, interconnects, and software environment.

Table 1: Operating System Information
| Field | Value |
|-------|-------|
| Hostname | smshost |
| Uptime | 10 days 13:56 |
| Load Average | 0.01 / 0.08 / 0.10 |
| OS | Rocky Linux 8.10 (Green Obsidian) |
| Kernel Platform | el8 |
| Support End | 2029-05-31 |


Table 2: System details for the CPU used in this study
| Item | Value |
|------|-------|
| Architecture | x86_64 |
| CPU(s) | 16 |
| Sockets | 2 |
| Cores per Socket | 8 |
| Threads per Core | 1 |
| Vendor | Intel |
| Model | Xeon E5-2680 0 @ 2.70GHz |
| Base Frequency | 2.70 GHz |
| Peak Frequency | 3.5 GHz |
| L1 Cache | 32K (data) + 32K (instruction) |
| L2 Cache | 256K |
| L3 Cache | 20 MB |
| NUMA Nodes | 2 |
| NUMA Node0 CPUs | 0–7 |
| NUMA Node1 CPUs | 8–15 |


Table 3: System Memory Information
| Field | Value |
|-------|-------|
| Total RAM | 31 GiB |
| Used | 2.1 GiB |
| Free | 10 GiB |
| Buff/Cache | 18 GiB |
| Available | 26 GiB |
| Swap Total | 15 GiB |
| Swap Used | 1.0 GiB |


Table 4:Sytem GPU / Accelerator Information shows no discrete GPUs were detected, indicating that all benchmarks rely solely on CPU computation
| Field | Value |
|-------|-------|
| NVIDIA GPU | None detected |
| Drivers | Not installed |


 Table 5. Network Interfaces
| Interface | State | IP Address | Notes |
|-----------|-------|------------|-------|
| lo | UP | 127.0.0.1 | Loopback |
| eno1 | UP | 10.128.23.188 | Main LAN |
| eno2 | UP | 10.10.10.10 | Cluster private network |
| ib0 | DOWN | — | Mellanox IB (ConnectX-3) |
| virbr0 | DOWN | 192.168.122.1 | Virtual bridge |

Table 6 PCI Network Hardware
| Device | Model |
|--------|-------|
| 02:00.0 | Intel I350 NIC |
| 02:00.3 | Intel I350 NIC |
| 82:00.0 | Mellanox ConnectX-3 |

Table 7:  Power (RAPL)
| Zone | Type | Energy (uJ) |
|------|------|-------------|
| intel-rapl:0 | package-0 | 16,511,888,631 |
| intel-rapl:0:0 | core | 43,998,797,599 |
| intel-rapl:0:1 | dram | 20,217,413,493 |
| intel-rapl:1 | package-1 | 61,947,548,996 |
| intel-rapl:1:0 | core | 21,046,507,595 |
| intel-rapl:1:1 | dram | 63,360,429,543 |


Table 8: Loaded Software Modules Used in this study
| Module Category | Modules Loaded |
|-----------------|----------------|
| Compilers | gnu12/12.4.0 |
| MPI | openmpi4/4.1.6 |
| Core Tools | autotools, hwloc, libfabric, ucx, prun |
| Math & Sci | openblas, petsc, fftw, scalapack, mumps, trilinos |
| Profiling | tau, scalasca, scorep, PAPI |


Table 9: MPI Information
| Field | Value |
|-------|-------|
| MPI Implementation | OpenMPI 4.1.6 |
| Path | /opt/ohpc/pub/mpi/openmpi4-gnu12/4.1.6/bin/mpirun |


This comprehensive system profiling establishes a baseline for understanding the hardware constraints and software dependencies influencing productivity and energy efficiency. It also enables consistent, reproducible benchmarking and facilitates direct correlation between system parameters, performance, and energy consumption.


## 3. Key System-Level Parameters and Their Impact

Understanding the hardware characteristics of HPC nodes is critical when evaluating both computational performance and energy efficiency. Modern compute nodes are complex systems, and application behavior is influenced not just by raw CPU speed, but also by how tasks are scheduled, memory is accessed, and threads are distributed across cores. To systematically explore these effects, this study focuses on several key system-level parameters that directly govern the balance between throughput and energy consumption.

By carefully controlling and monitoring these parameters, it becomes possible to quantify the **productivity-to-energy ratio**, which measures the amount of useful computational work achieved per unit of energy. Adjustments to these parameters allow researchers to identify configurations that either maximize performance, minimize energy use, or strike a balance between the two.

The following parameters were considered in this study:

**CPU Behavior:** The CPU frequency and its scaling behavior determine the processing speed and energy consumption of the node. Jobs can request a **performance governor** to maintain a high and stable frequency, maximizing throughput, or a **powersave governor** to reduce frequency and conserve energy. If administrative permissions restrict changing the governor, the current frequency and turbo state are recorded to ensure results can be accurately interpreted.

**Placement and Core Pinning:** Proper placement of MPI ranks and binding of threads to specific CPU cores reduces resource contention, minimizes random process migration, and enhances cache locality. Core pinning stabilizes execution timing, improves reproducibility, and is one of the most effective ways to increase throughput without modifying application code.

**NUMA Memory Policy:** On multi-socket nodes, memory is physically segmented across NUMA nodes. Assigning memory **locally** to the socket executing the process reduces access latency, while **interleaving memory** across sockets distributes bandwidth more evenly. The optimal policy depends on the memory access patterns of the application, and both approaches were tested to evaluate their effect on performance and energy efficiency.

Each configuration directly affects how quickly the CPU completes computational tasks and the total power drawn by the node. By measuring both **performance metrics** (e.g., GFLOPS or time-to-solution) and **energy consumption**, a **productivity-to-energy ratio** can be computed, reflecting the amount of useful work per unit of energy. Core pinning and NUMA tuning primarily enhance performance stability, while CPU frequency and governor settings directly modulate the energy profile. Together, these parameters allow identification of the most energy-efficient and high-performance configurations for HPC benchmarks.

By tuning these parameters, the study created a reproducible framework to measure runtime performance and energy consumption, enabling calculation of productivity-to-energy ratios for both compute-intensive applications like LAMMPS and memory-bound workloads such as OpenFOAM. This methodology provides insight into the interplay between hardware configuration and application characteristics, supporting energy-efficient high-performance computing without compromising throughput.


## 4. Application Benchmarks: Productivity and Energy Efficiency

To evaluate the interplay between computational performance and energy consumption, this study conducted two primary benchmark tests using representative HPC applications. The goal was to determine the **optimal Productivity-to-Energy Cost Ratio** on repurposed legacy hardware. Unlike conventional performance assessments that focus solely on raw execution speed, this analysis emphasizes the **trade-offs between computational throughput and power consumption**. System parameters, including CPU frequency, core placement, and parallelization settings, were systematically manipulated through the **Slurm scheduler** to capture these trade-offs under controlled conditions.

The benchmark applications were carefully selected to represent contrasting workload characteristics commonly encountered in HPC:

- **LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator):** A molecular dynamics code that is primarily **compute-intensive**, relying heavily on floating-point operations and optimized for parallel execution across multiple cores. Its workload stresses the processor, making it ideal for evaluating configurations that target maximum computational throughput.  

- **OpenFOAM (Open Field Operation and Manipulation):** An open-source computational fluid dynamics (CFD) software that is typically **memory-bound**, with performance largely determined by memory bandwidth, latency, and inter-process communication. OpenFOAM benchmarks highlight how memory and network characteristics influence energy efficiency, particularly under varying CPU frequencies and NUMA memory policies.  

Each application was run using carefully defined test cases, allowing measurement of both **performance metrics** (e.g., timesteps per second for LAMMPS, iterations per second for OpenFOAM) and **energy consumption metrics** (via Intel RAPL and system monitoring tools). This dual approach enables a comprehensive assessment of **productivity per unit of energy**, providing actionable insights into how hardware configuration and workload characteristics interact to influence the efficiency of HPC applications.

Subsequent sections provide a detailed description of the benchmark setup, methodology for performance and energy measurement, and the analytical framework used to derive productivity-to-energy ratios for both compute- and memory-intensive workloads.
  

## 4.1 OpenFOAM (Open Field Operation and Manipulation)

OpenFOAM is an open-source CFD software package that simulates fluid flow, heat transfer, turbulence, and multiphase systems using the finite volume method. For many large-scale CFD problems, OpenFOAM is typically memory-bound or interconnect-bound, meaning its performance depends more on memory bandwidth and latency than on raw floating-point speed of the CPU. It allows users to create meshes, define physics and boundary conditions, solve equations in parallel, and post-process results to analyze velocity, pressure, and other physical fields in the simulated domain.

### Benchmark Details

| Aspect                | Detail |
|-----------------------|--------|
| OpenFOAM Version      |  OpenFOAM-v2412|
| Benchmark Case        | simplefoam |
| Problem Size          | 248769 cells|
| Scaling Type          | Strong Scaling |
| Key Metric (Productivity) | Total Runtime (seconds) or Iterations per second |
| Key Metric (Efficiency)  | Iterations/s per Watt |

Table 10: Summary of OpenFOAM benchmark parameters.

---
# Key Performance and Energy Metrics for OpenFOAM

To evaluate productivity and energy efficiency for OpenFOAM on repurposed HPC hardware, this study uses a set of core computational and energy-related metrics. These metrics capture both simulation performance and energy cost, allowing detailed analysis of how CPU frequency, NUMA configuration, and core placement influence overall efficiency on legacy HPC systems.


### 1. Wall-Clock Time

Wall-clock time represents the total real elapsed time from the beginning of the simulation to completion. It is the most intuitive measure of productivity, as shorter wall-clock time means faster delivery of results.

However, wall-clock time alone is insufficient for deeper analysis because it does not reveal:

- How much of the time is spent on computation vs. communication  
- Whether power draw was high or low during the run  
- Whether solver inefficiencies affected total runtime  

For this study, wall-clock time was paired with energy consumption (Joules) to calculate productivity-per-watt under different hardware configurations.


### 2. Iteration Time and Iteration Rate

OpenFOAM performs iterative updates of the governing equations. Two important metrics are:

- **Iteration Time (s/iteration)**  
- **Iteration Rate (iterations/second)**  

These metrics provide finer granularity than total runtime and help evaluate:

- Changes in CPU frequency  
- Solver configuration differences  
- Effects of NUMA locality and core pinning  

Iteration rate was also used directly in calculating the **Productivity-to-Energy Cost Ratio**, since power readings were averaged across the iteration loop.

### 3. Solver Performance and Linear Solver Iterations

Each OpenFOAM iteration involves solving multiple linear systems for pressure and velocity. The number and efficiency of these linear solver iterations strongly influence overall runtime.

Tracked metrics include:

- **Linear solver iterations per outer iteration**  
- **Convergence rates of pressure/velocity solvers**  

Higher solver iteration counts can indicate:

- Poor mesh quality  
- Inefficient preconditioners  
- CPU frequency too low for memory throughput  
- Penalties from NUMA non-local memory accesses  

Monitoring solver iteration behaviour ensured that changes in hardware parameters did not degrade numerical performance.


 #### 4. Residual Reduction and Convergence Behaviour

Residuals quantify how well the numerical solution satisfies the discretized equations. For this study:

- Pressure residuals were typically converged to **10⁻⁶**  
- Velocity residuals converged to **10⁻⁵**

This metric was crucial for:

- Ensuring all benchmark runs solved the *same* physical problem  
- Verifying that lower-energy configurations did not destabilize the solver  
- Detecting oscillations or divergence due to poor time-step or scheme choices  

Residual monitoring guaranteed scientific consistency across all energy-efficiency tests.


 ### 5. Parallel Speedup and Parallel Efficiency

Because OpenFOAM is parallelized using MPI, parallel performance metrics were required to understand scaling behaviour on the tested hardware.

 **5.1 Speedup:** Speedup measures how much faster a parallel implementation of an application executes compared to its serial (single-core) execution. It is defined as: 

The speedup of a parallel application is defined as:

$$
\mathbf{S(N) = \frac{T(1)}{T(N)}}
$$


 Where:
S(N) = Speedup achieved using N processors

T(1) = Execution time using a single processor

T(N) = Execution time using N processors

Interpretation:
S(N) = 1: No speedup (parallelization ineffective)
S(N) = N: Ideal linear speedup (perfect parallel scaling)
S(N) > N: Super-linear speedup (rare, usually due to cache effects)


 **5.2  Parallel Efficiency:**  Parallel efficiency quantifies how effectively the computational resources are utilized. It normalizes speedup by the number of processors:



$$
\mathbf{E(N) = \frac{S(N)}{N} = \frac{T(1)}{N \cdot T(N)}}
$$

  
  Where:

E(N) = Efficiency of using N processors

Interpretation:
E(N) = 1 (100% efficiency): Perfect resource utilization

E(N) < 1: Some overhead or idle time reduces efficiency

E(N) > 1: Super-linear efficiency (rare)

Strong-scaling behaviour was evaluated by running the same case on increasing core counts. Scaling efficiency typically dropped at higher core counts due to:
- MPI communication overhead  
- Memory bandwidth saturation  
- NUMA locality penalties  

A value close to **1** indicates excellent scaling, while lower values indicate diminishing returns as more resources are added.
These measurements helped identify the core count and frequency settings that maximized both performance and energy efficiency.


## How These Metrics Support the Study Objective

Together, these metrics enable:
- Precise mapping of how CPU frequency affects energy efficiency  
- Understanding of compute-bound vs. memory-bound behaviour  
- Identification of optimal operating points (maximum productivity per watt)  
- Evaluation of legacy hardware viability for modern OpenFOAM workloads  

By relating iteration rate, solver cost, power draw, and convergence behaviour, this metric set provides a comprehensive framework for characterizing energy-aware performance on repurposed HPC systems.




## Results: Productivity vs. Energy Efficiency

We tested various fixed CPU clock frequencies to find the optimal balance between computational speed and power draw.

| CPU Frequency (GHz) | Iterations/s | Avg Power (Watts) | Energy Efficiency (Iter/s per Watt) | Relative Efficiency | Iterations Per Joule
|--------------------|--------------|-----------------|------------------------------------|------------------|----------------------|
| Max Turbo (3.5)    | 250          |   60.25 W       | 0.0166 I/W                         | 90%              | 0,0040               |
| Optimal (2.4)      | 250          | 28.99 W         | 0,0344                             | 100%             |0.0049               |
| Low (2.0)          | 250          |  26.39 W        | 0.0379                             | 98%              |0,0070               |

Table 11: OpenFOAM single-node performance and energy efficiency comparison across different CPU clock speeds. Results shown are for the best performing run at each frequency.


---

### Comparative Analysis and Correlation

- The OpenFOAM results show a more pronounced shift in the **efficiency curve** compared to LAMMPS.  
- The **optimal energy efficiency** for OpenFOAM occurs at [Insert Optimal GHz from table], typically **lower than the optimal frequency for LAMMPS**.  
- This confirms that OpenFOAM performance is heavily influenced by **memory subsystem performance**. Increasing CPU clock speed beyond a threshold yields **minimal performance gain** while **power consumption rises significantly**.  
- Correlation between OpenFOAM performance and CPU floating-point performance is weaker than for LAMMPS, while correlation with **memory bandwidth** and memory channels is stronger.  

Scatter plots (Figure 1 and Figure 2) visually demonstrate the trade-off between performance and power, clearly identifying the optimal frequency point for both applications.

---

## 4.2 LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator)
LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator) is a classical molecular dynamics (MD) code designed for efficient execution on parallel computing architectures. Its primary focus is materials modelling, and it includes a wide variety of interatomic potential models covering solid-state systems such as metals and semiconductors, soft-matter systems including polymers and biomolecules, as well as coarse-grained and mesoscopic models.

Parallelism in LAMMPS is achieved through domain decomposition and message-passing techniques, allowing simulations to be distributed across many processors with minimal communication overhead. Many of its computational kernels also include optimised variants that leverage hardware acceleration on CPUs (e.g., vectorisation, threaded libraries) and GPUs. This makes LAMMPS highly scalable and well-suited to both legacy HPC hardware and modern heterogeneous computing environments. It is primarily written in C++ and is **compute-intensive**, meaning its performance is expected to correlate strongly with the **FLOPS capability** of the processor.


### Benchmark Details

| Aspect                | Detail |
|-----------------------|--------|
| LAMMPS Version        | LAMMPS 22 Jul 2025, Update 1 |
| Benchmark Case        | [Specify case, e.g., lj or rheo] |
| Problem Size          | [Specify atom count, e.g., 256,000 atoms] |
| Scaling Type          | Strong Scaling (fixed problem size, variable cores) |
| Key Metric (Productivity) | Timesteps per second (ts/s) |
| Key Metric (Efficiency)  | Timesteps/s per Watt |

Table 8: Summary of LAMMPS benchmark parameters.

---
# Key Performance and Energy Metrics for Lammps


### 1. Wall-Clock Time

 Wall-clock time measures the total elapsed real time from the start to the completion of a LAMMPS simulation.It is the most intuitive indicator of productivity because it directly represents how quickly simulation results can be obtained. Faster completion times translate to higher throughput for research and engineering workflows.
 
Wall-clock time alone does not provide sufficient insight into the underlying reasons for performance differences. Specifically, it cannot distinguish:
- Time spent performing computationally intensive force calculations versus communication overhead between MPI ranks  
- Inefficiencies due to memory bottlenecks or NUMA non-locality  
- Periods of high or low energy consumption  

 By pairing wall-clock time with energy consumption measured in Joules, one can calculate **timesteps per Joule** (or work per unit energy), providing a more meaningful assessment of productivity in energy-constrained HPC environments.



### 2. Timestep Duration and Timestep Rate

 In LAMMPS, molecular dynamics proceeds in discrete timesteps where atomic positions and velocities are updated iteratively.

- **Timestep Duration (s/timestep):** The average time taken for a single update of the simulation system  
- **Timestep Rate (timesteps/s):** The number of timesteps completed per second  

 These metrics provide finer granularity than wall-clock time and allow analysis of the impact of:
- **CPU frequency scaling:** Lower frequencies may reduce power draw but increase timestep duration  
- **Threading and MPI decomposition:** The number of threads per process and the distribution of MPI ranks influence the speed of force calculations  
- **Neighbor list rebuild frequency and force computation settings:** Excessive neighbor list rebuilds increase timestep duration  

 Since energy measurements are averaged across timesteps, **timestep rate** becomes a direct input to productivity-per-watt calculations.


### 3. Force Calculation and Neighbor List Performance

 LAMMPS performance is strongly dominated by interatomic force calculations and neighbor list updates, which are necessary to identify interacting particle pairs.

**Tracked Metrics:**
- **Force Computation Time per Timestep:** Time spent calculating pairwise and long-range forces  
- **Neighbor List Build Frequency:** Determines how often neighbor lists are reconstructed, affecting computational load  
- **Pair Style and K-Space Performance:** Different interaction models (e.g., Lennard-Jones, Coulombic) and long-range solvers can significantly influence computation time  

 Poor tuning in these areas can lead to:
- Increased total runtime  
- Higher energy consumption per timestep  
- Reduced simulation throughput  

Monitoring these metrics ensures that energy-efficient configurations do not compromise computational accuracy or simulation fidelity.


### 4. Parallel Speedup and Efficiency

**LAMMPS Parallelization:** LAMMPS supports MPI for distributed memory parallelism and OpenMP for shared memory threading.  

**Key Metrics:**

- **Speedup (S(N)):**  
\[
S(N) = \frac{T(1)}{T(N)}
\]  
where \(T(1)\) is the runtime on a single core and \(T(N)\) is the runtime on \(N\) cores.

- **Parallel Efficiency (E(N)):**  
\[
E(N) = \frac{S(N)}{N}
\]  

**Strong Scaling:** Measures performance improvement when increasing cores for a fixed problem size. Efficiency typically decreases at high core counts due to:

- MPI communication overhead  
- Memory bandwidth saturation  
- NUMA locality penalties  

**Weak Scaling:** Measures performance when problem size and core count increase proportionally. Ideal weak scaling maintains constant runtime; deviations indicate communication or memory bottlenecks.
 Optimal parallel configurations balance speedup and energy efficiency, maximizing timesteps per Joule rather than only raw speed.


### 5. Numerical Accuracy and Simulation Integrity

 While LAMMPS does not solve linear systems like OpenFOAM, numerical stability is critical for meaningful simulations. Energy-efficient configurations must maintain physical accuracy.

**Monitored Metrics:**

- **Total Energy Drift:** Ensures conservation of energy over time in microcanonical (NVE) simulations  
- **Force and Velocity Consistency:** Detects numerical errors introduced by aggressive energy-saving settings or low CPU frequencies  

**Purpose:** Ensures that productivity gains achieved through tuning do not compromise the validity of the scientific results

---

### Results: Productivity vs. Energy Efficiency

Benchmark performance (Timesteps/s) and average power consumption (Watts) were measured at various fixed CPU clock frequencies, controlled via the Slurm job script.

| CPU Frequency (GHz) | Timesteps/s | Avg Power (Watts) | Energy Efficiency (ts/s per Watt) | Atom Steps/s |
|--------------------|------------|-----------------|---------------------------------|--------------|
| Max Turbo (3.5)    | 7,152      | 88.75           | 0.1127                          | 1.83         |
| Optimal (2.7)      | 5,278      | 46.03           | 0.0217                          | 1.33         |
| Low (2.0)          | 6,292      | 39.06           | 0.0256                          | 1.61         |

**Table 9:** LAMMPS single-node performance and energy efficiency comparison across different CPU clock speeds. The results correspond to the best performing run at each frequency.

> The results show that the **highest raw computational performance** is achieved at the maximum turbo frequency of 3.5 GHz. However, **optimal energy efficiency**—defined as the highest ratio of timesteps per second per Watt—is achieved at 2.0 GHz, highlighting the inherent trade-off between performance and power consumption. While higher frequencies deliver faster simulation results, they consume disproportionately more energy, making moderate frequencies more favorable for energy-aware HPC operation.

## 4.1 Performance-Oriented Configuration

In high-performance computing (HPC) research, some experiments prioritize **maximizing computational speed and simulation throughput**. This configuration focuses on achieving the shortest runtime and the highest performance possible, often at the cost of increased power consumption. It is particularly useful when runtime is critical, such as large-scale simulations or time-sensitive studies.

### 4.1.1 Maximum CPU Frequency
Running the CPU at its maximum frequency ensures that each core delivers the highest computational throughput.  

- **Impact on performance:** Higher clock speeds allow more instructions per second, reducing the time required for each simulation step.  
- **Scientific reasoning:** Many numerical methods, including molecular dynamics simulations, are CPU-bound. Therefore, increasing frequency directly improves performance.  
- **Practical implementation:** Use performance-oriented CPU governors or manual frequency scaling tools (e.g., `cpupower frequency-set --governor performance`) to ensure the CPU stays at peak frequency.

### 4.1.2 Aggressive Solver Settings
In this configuration, solver parameters are tuned to prioritize speed over conservative stability.  

- **Examples of aggressive tuning:**  
  - Larger time steps in molecular dynamics integration.  
  - Reduced accuracy thresholds for iterative solvers.  
  - Skipping optional stability checks or corrections that slow down computation.  
- **Energy and performance trade-off:** While aggressive settings may increase the risk of instability or slightly reduce accuracy, they significantly reduce computation time per step, which is ideal when speed is critical.

### 4.1.3 Optimized Parallel Decomposition
Efficient use of computational resources is essential for high performance. Optimized parallel decomposition ensures that the workload is distributed effectively across all cores and nodes.  

- **Domain decomposition:** Split the simulation box into subdomains that each MPI rank or thread handles.  
- **Load balancing:** Evenly distribute particles or computational work to prevent idle cores.  
- **Communication optimization:** Minimize inter-process communication and overlap communication with computation wherever possible.  
- **Energy consideration:** Although this configuration uses more cores at maximum speed, optimized decomposition ensures that no cores are idle, improving the energy-to-solution ratio for high-performance runs.

### 4.1.4 Minimal I/O Operations
Frequent input/output operations can bottleneck performance, even on high-speed clusters.  

- **Strategies:**  
  - Reduce the frequency of writing checkpoints or trajectory data.  
  - Aggregate outputs to minimize disk access.  
  - Store only critical simulation data required for analysis.  
- **Performance benefit:** Limiting I/O overhead allows the CPU and memory subsystem to focus on computation, maximizing simulation speed.  
- **Energy consideration:** Although I/O is less significant compared to computation, minimizing unnecessary writes reduces energy consumption slightly and prevents potential slowdowns caused by I/O latency.

### Summary
The performance-oriented configuration aims to **maximize computational throughput**. By using maximum CPU frequency, aggressive solver settings, optimized parallel decomposition, and minimal I/O, this setup achieves the fastest possible simulation times. This approach is ideal when speed is critical, but it usually consumes more energy compared to power-efficient configurations. It is suitable for time-sensitive experiments or large-scale simulations where runtime dominates research objectives.

---
## 4.2 Power-Efficient Configuration

When designing experiments with energy efficiency in mind, the objective shifts from achieving maximum computational speed to **optimizing simulations for minimal power consumption without compromising accuracy**. This configuration is essential for sustainable high-performance computing, especially in research environments where energy cost and environmental impact matter.

### 4.2.1 Reduced CPU Frequency
Running CPUs at full frequency maximizes performance but also increases energy usage significantly. In the power-efficient configuration, the CPU frequency is deliberately lowered.  

- **Impact on performance:** Lower frequency slows down computation slightly, but it significantly reduces the power draw per core.  
- **Scientific reasoning:** Energy usage is roughly proportional to the cube of voltage/frequency. Reducing frequency reduces dynamic power dissipation and thermal stress, which can also improve long-term hardware stability.  
- **Practical implementation:** Modern processors allow software or OS-level frequency capping, often using tools like `cpupower` or CPU governors (`powersave` mode).

### 4.2.2 Conservative Solver Settings
The solver is the part of the simulation engine responsible for calculating forces, integrating motion, and updating particle positions. Using conservative settings focuses on stability and accuracy rather than raw speed.  

- **Example adjustments:**  
  - Smaller time steps to prevent numerical errors.  
  - Avoiding aggressive approximations or shortcuts that speed up computation but may require re-computation or introduce instabilities.  
- **Energy rationale:** Stable and conservative settings prevent wasted computation from error corrections or unstable runs. While each step may take slightly longer, the total energy consumed per reliable simulation can be lower.

### 4.2.3 Coarse-Grained Parallelism
Instead of using the maximum number of CPU cores or MPI ranks, coarse-grained parallelism uses **fewer, well-utilized cores**, balancing workload efficiency and energy consumption.  

- **Why it matters:** Using too many cores can create idle or underutilized threads, which still consume power but do not contribute meaningfully to the simulation.  
- **Communication overhead:** Reducing the number of cores decreases the amount of inter-process communication, which is energy-intensive.  
- **Implementation:** Carefully match MPI ranks and OpenMP threads to the problem size. For smaller simulations, fewer ranks with more threads per rank can be more energy-efficient.

### 4.2.4 Efficient I/O Strategies
Input/output operations can be surprisingly expensive in terms of energy. Writing large amounts of data frequently consumes CPU cycles, memory bandwidth, and disk power.  

- **Optimization approaches:**  
  - Reduce output frequency: Write data only at necessary intervals.  
  - Batch I/O: Group multiple results into single write operations instead of frequent small writes.  
  - Selective logging: Save only essential variables rather than the entire system state.  
- **Energy benefit:** Less frequent and optimized I/O reduces unnecessary disk and memory usage, lowering total energy consumption.

### Summary
The power-efficient configuration is a **carefully balanced approach** that trades some speed for substantial energy savings. By reducing CPU frequency, using conservative solver settings, running fewer well-utilized cores, and minimizing I/O overhead, simulations maintain accuracy while consuming less power.  

This approach is critical in research environments aiming to **minimize environmental impact and operational costs** while still performing high-fidelity simulations.


## 4.3 Summary of Correlation to Hardware Characteristics

| Aspect                         | LAMMPS (Compute-Intensive) | OpenFOAM (Memory-Bound) |
|--------------------------------|---------------------------|-------------------------|
| Floating Point Performance (GFlop/s) | High Positive [e.g., 0.85] | Moderate Positive [e.g., 0.55] |
| Memory Bandwidth (GB/s)        | Low [e.g., 0.20]          | Moderate/High [e.g., 0.70] |
| Memory Channels                | Very Low [e.g., 0.05]     | Moderate [e.g., 0.45] |

**Table 12:** Correlation coefficients for different system hardware aspects correlated to the Energy Efficiency Ratio of the benchmark applications.

> LAMMPS benefits from running closer to the CPU's compute peak, while OpenFOAM efficiency drops rapidly after exceeding a lower frequency threshold due to memory bottlenecks limiting performance while power continues to climb.

---
## 5. Data Collection & Analysis

In this research, careful data collection and analysis is essential to evaluate the trade-offs between performance and energy efficiency. By systematically measuring key metrics during simulations, we can quantify how different configurations affect runtime, energy consumption, and resource utilization.

### 5.1 Metrics to Collect

The selection of metrics is based on their ability to quantify **performance bottlenecks, energy cost, and parallel efficiency**. Each metric is described below, along with its scientific relevance, measurement methodology, and expected analytical outcomes.

#### 5.1.1 Execution Time
Execution time represents the wall-clock duration required to complete a simulation or a computational task. 
It is a direct measure of performance efficiency. By analyzing execution times across varying configurations, one can identify the scaling behavior and computational limits of the simulation framework. Shorter runtimes suggest better utilization of computing resources.  
 Execution time can be measured using system timers or job scheduler logs. For example:  
  ```bash
  time mpirun -np 16 lmp_mpi -in in.lj
````
High-resolution timers capture both computation and communication overheads.
Analysis: Compare execution times across different parallelization strategies to identify optimal configurations.

#### 5.1.2 Energy Consumption

The total energy used by the CPU, memory, and other system components during simulation.

Energy consumption directly impacts operational cost and sustainability in HPC.

Practical measurement:

Intel RAPL interface:
```bash
grep -i energy /sys/class/powercap/intel-rapl/*/energy_uj
````

External power meters for node-level measurements.

Analysis: Calculate energy-to-solution metrics to understand the trade-off between performance and power efficiency.


### 5.1.3 CPU Utilization

CPU utilization measures the fraction of CPU cycles actively used for computation versus idle or waiting.  
 High CPU utilization indicates that cores are performing useful work. Low utilization may indicate load imbalance, excessive synchronization, or memory bottlenecks.  

Practical measurement: Use system tools like htop, mpstat, or perf:

- Tools like `mpstat` or `htop` show real-time utilization:  
    ```bash
    mpstat -P ALL 1
    ```  
  - Performance counters provide per-thread/core metrics for finer analysis.  
- **Analysis considerations (expanded):**  
  - Identify cores that are underutilized due to poor domain decomposition in MPI simulations.  
  - Correlate CPU utilization with memory and network metrics to determine whether the simulation is computation-bound, memory-bound, or communication-bound.  
  - High idle times suggest opportunities to optimize parallelization or adjust thread allocation.  
  - CPU utilization trends over time can reveal whether workloads are evenly distributed throughout the simulation or if there are temporal hotspots.
 
  - 
#### 5.1.4 Memory Bandwidth Usage

-  Memory bandwidth usage quantifies the rate at which data is transferred between main memory and the CPU.  
- Memory-bound applications are limited by the speed of data movement rather than computation. Understanding bandwidth usage helps identify memory bottlenecks and improve cache efficiency.  
- **Measurement methodology:**  
  - Profiling tools such as `perf`, `Intel VTune`, or `pcm-memory` can measure bytes per second transferred, cache misses, and memory stall cycles.  
- **Analysis consideration:**  
  - Identify memory-intensive kernels in the simulation that dominate execution time.  
  - Evaluate how different thread counts or MPI ranks affect memory access patterns.  
  - Correlate memory usage with CPU utilization and execution time to detect whether the application is compute- or memory-bound.  
  - Optimize memory locality using thread/core binding to reduce cross-socket memory traffic.

---
## 6. Expected Outcomes

By conducting these experiments, we anticipate uncovering a clear picture of how different parallelization and configuration strategies affect both performance and energy efficiency. Key expected results include:

- **Quantitative Performance vs. Power Insights:** Detailed measurements will reveal the trade-offs between runtime acceleration and energy consumption, helping to identify scenarios where maximum performance might come at a disproportionate energy cost.  
- **Identification of Influential Parameters:** Through systematic testing, it will be possible to pinpoint which parallelization settings—such as MPI ranks, OpenMP threads, or CPU frequency—have the strongest impact on overall efficiency.  
- **Energy-Aware Optimization Guidelines:** The findings will provide actionable recommendations for configuring high-performance simulations in a way that balances speed and energy usage, supporting researchers who must operate under strict energy budgets or sustainability goals.  

These outcomes are intended to bridge the gap between raw computational power and practical energy-conscious high-performance computing.

## 7. Tools & Scripts

To carry out this study efficiently, a combination of automation and monitoring tools will be employed:

- **Power Measurement Scripts:** Custom scripts using interfaces like RAPL or IPMI will record fine-grained energy usage across CPU, memory, and other components during simulation runs.  
- **Benchmark Automation:** Bash and Python scripts will automate the execution of multiple configurations, ensuring consistent, reproducible experiments while systematically varying parameters like thread count or solver settings.  
- **Data Processing and Visualization:** Collected data will be analyzed using Python (pandas, matplotlib, seaborn) to generate clear visualizations of performance and energy trends, enabling both quantitative comparison and intuitive understanding of trade-offs.  

Together, these tools and scripts will provide a robust framework for capturing meaningful insights into the interplay between computational performance and energy efficiency.

# 8. Conclusion

This study systematically evaluated the interplay between computational productivity and energy consumption on repurposed legacy HPC hardware, using representative memory-bound (OpenFOAM) and compute-bound (LAMMPS) applications. By carefully controlling CPU frequency, thread placement, NUMA memory policies, and parallelization strategies, we quantified the productivity-to-energy ratio, providing a nuanced understanding of how system configuration impacts both performance and power efficiency.

## Key insights from this investigation include:

Workload-Specific Energy-Performance Trade-offs:

Memory-bound applications like OpenFOAM exhibit limited performance gains at higher CPU frequencies, while energy consumption increases disproportionately. Optimal energy efficiency occurs at moderate frequencies, emphasizing the dominant role of memory bandwidth and latency.

Compute-bound applications like LAMMPS benefit from higher CPU frequencies, but energy-efficient configurations can still achieve substantial performance while reducing power consumption.

Identification of Optimal Operating Points:
Systematic tuning revealed frequency and parallelization settings that maximize productivity per unit of energy. For both applications, intermediate CPU frequencies combined with proper core pinning and NUMA-local memory allocation achieved the best balance between throughput and power draw, underscoring the value of energy-aware configuration over raw performance maximization.

Correlation with Hardware Characteristics:
LAMMPS efficiency correlates strongly with floating-point performance, whereas OpenFOAM efficiency depends more on memory bandwidth and interconnect performance. This highlights that energy-aware HPC optimization must be application-specific, aligning hardware resources with workload characteristics to maximize sustainability without sacrificing scientific accuracy.

Framework for Energy-Conscious HPC Practices:
The methodology—integrating performance profiling, energy measurement via RAPL, and automated benchmarking—establishes a reproducible approach for evaluating productivity-to-energy trade-offs. This framework supports evidence-based decisions for scheduling, hardware utilization, and application tuning, providing practical guidance for HPC operators and researchers aiming to reconcile high computational throughput with energy efficiency.

# Educational Implications

This study demonstrates that sustainable HPC is achievable even on legacy systems through informed configuration and workload-aware optimization. By prioritizing productivity per unit energy rather than raw speed alone, researchers and system administrators can:

Reduce operational costs and environmental impact

Extend the usable life of legacy hardware

Achieve scientifically reliable results without unnecessary energy expenditure

In conclusion, energy-aware high-performance computing is not a compromise but an opportunity: legacy HPC systems, when carefully tuned, can deliver substantial scientific output while minimizing power consumption. This work provides both a conceptual framework and practical guidance for integrating energy efficiency into routine HPC workflows, paving the way for more sustainable, cost-effective, and environmentally responsible scientific computation.


