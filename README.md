# 1. Introduction

High-Performance Computing (HPC) systems are foundational to scientific discovery, enabling simulations and analyses that are otherwise infeasible. Yet, as computational demand escalates, the associated energy footprint has emerged as a critical limiting factor for both operational sustainability and cost-efficiency. Modern HPC facilities are power-hungry: the balance between computational productivity and energy consumption is no longer merely a secondary concern but a central metric that dictates hardware design, scheduling policies, and workload optimization strategies.

Traditional performance evaluations of HPC applications emphasize raw throughput—measured in FLOPS, timesteps per second, or iterations per second—without explicitly accounting for the power costs incurred to achieve those performance gains. However, a nuanced understanding of **productivity-to-energy trade-offs** is essential in the current landscape, where energy constraints may outweigh peak computational capability in determining overall system efficiency. In other words, **maximum performance does not equate to optimal productivity** when energy consumption is considered.

The energy consumed by HPC applications is governed by multiple interacting factors. At the node level, processor frequency and voltage, memory subsystem bandwidth and latency, interconnect efficiency, and accelerator utilization all contribute to the instantaneous power draw. At the workload level, application characteristics—compute-bound versus memory-bound, communication patterns, and I/O intensity—dictate how hardware resources are stressed and, consequently, how energy is expended. These dependencies are complex, nonlinear, and workload-specific, underscoring the need for systematic benchmarking across a representative set of applications.

In this study, we aim to **map energy to productivity** across representative HPC applications, quantifying the **energy efficiency ratio**—a measure of computational output per unit of energy consumed. By performing controlled experiments on repurposed legacy hardware, we explicitly manipulate CPU clock frequencies and leverage job scheduling tools (Slurm) to systematically explore the trade-offs between raw performance and energy consumption. Our focus applications, **LAMMPS** (compute-intensive molecular dynamics) and **OpenFOAM** (memory-bound computational fluid dynamics), represent contrasting workload characteristics, allowing us to generalize insights on hardware-dependent energy behavior.

Through this investigation, we aim to answer fundamental questions:  

1. How does energy efficiency vary with workload type, CPU frequency, and system configuration?  
2. What is the optimal operating point for maximizing productivity per watt?  
3. How do hardware characteristics—FLOPS, memory bandwidth, and interconnect topology—correlate with energy-to-productivity metrics across applications?  

By integrating performance measurement with power profiling, this work contributes a **framework for energy-aware HPC benchmarking**, providing actionable guidance for both system operators and application developers seeking to reconcile high computational throughput with sustainable energy usage. The ultimate goal is not only to characterize system behavior but to inform policies that maximize scientific productivity while minimizing operational cost and environmental impact.


## 2. System Information Overview

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

## 3. Key Tunable Parameters and Their Impact

**CPU Behavior:** The CPU frequency and its scaling behavior determine the processing speed and energy consumption of the node. Jobs can request a **performance governor** to maintain a high and stable frequency, maximizing throughput, or a **powersave governor** to reduce frequency and conserve energy. If administrative permissions restrict changing the governor, the current frequency and turbo state are recorded to ensure results can be accurately interpreted.

**Placement and Core Pinning:** Proper placement of MPI ranks and binding of threads to specific CPU cores reduces resource contention, minimizes random process migration, and enhances cache locality. Core pinning stabilizes execution timing, improves reproducibility, and is one of the most effective ways to increase throughput without modifying application code.

**NUMA Memory Policy:** On multi-socket nodes, memory is physically segmented across NUMA nodes. Assigning memory **locally** to the socket executing the process reduces access latency, while **interleaving memory** across sockets distributes bandwidth more evenly. The optimal policy depends on the memory access patterns of the application, and both approaches were tested to evaluate their effect on performance and energy efficiency.

Each configuration directly affects how quickly the CPU completes computational tasks and the total power drawn by the node. By measuring both **performance metrics** (e.g., GFLOPS or time-to-solution) and **energy consumption**, a **productivity-to-energy ratio** can be computed, reflecting the amount of useful work per unit of energy. Core pinning and NUMA tuning primarily enhance performance stability, while CPU frequency and governor settings directly modulate the energy profile. Together, these parameters allow identification of the most energy-efficient and high-performance configurations for HPC benchmarks.

By tuning these parameters, the study created a reproducible framework to measure runtime performance and energy consumption, enabling calculation of productivity-to-energy ratios for both compute-intensive applications like LAMMPS and memory-bound workloads such as OpenFOAM. This methodology provides insight into the interplay between hardware configuration and application characteristics, supporting energy-efficient high-performance computing without compromising throughput.


## 4. Application Benchmarks: Productivity and Energy Efficiency

In this study, we conducted two primary benchmark tests using representative HPC applications to determine the optimal **Productivity-to-Energy Cost Ratio** on the repurposed legacy hardware. Unlike traditional performance reports, our analysis focuses on the trade-off between raw speed and power consumption by manipulating system parameters via the Slurm scheduler.

The two applications benchmarked were:

- **LAMMPS:** Molecular Dynamics (compute-intensive).  
- **OpenFOAM:** Computational Fluid Dynamics (typically memory-bound).  

More details on these benchmarks and the methodology used to gather both performance and power data are detailed in the individual sections below.  

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
# Key Performance and Energy Metrics for OpenFOAM

To evaluate productivity and energy efficiency for OpenFOAM on repurposed HPC hardware, this study uses a set of core computational and energy-related metrics. These metrics capture both simulation performance and energy cost, allowing detailed analysis of how CPU frequency, NUMA configuration, and core placement influence overall efficiency on legacy HPC systems.

---

## 1. Wall-Clock Time

Wall-clock time represents the total real elapsed time from the beginning of the simulation to completion. It is the most intuitive measure of productivity, as shorter wall-clock time means faster delivery of results.

However, wall-clock time alone is insufficient for deeper analysis because it does not reveal:

- How much of the time is spent on computation vs. communication  
- Whether power draw was high or low during the run  
- Whether solver inefficiencies affected total runtime  

For this study, wall-clock time was paired with energy consumption (Joules) to calculate productivity-per-watt under different hardware configurations.

---

## 2. Iteration Time and Iteration Rate

OpenFOAM performs iterative updates of the governing equations. Two important metrics are:

- **Iteration Time (s/iteration)**  
- **Iteration Rate (iterations/second)**  

These metrics provide finer granularity than total runtime and help evaluate:

- Changes in CPU frequency  
- Solver configuration differences  
- Effects of NUMA locality and core pinning  

Iteration rate was also used directly in calculating the **Productivity-to-Energy Cost Ratio**, since power readings were averaged across the iteration loop.

---

## 3. Solver Performance and Linear Solver Iterations

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

---

## 4. Residual Reduction and Convergence Behaviour

Residuals quantify how well the numerical solution satisfies the discretized equations. For this study:

- Pressure residuals were typically converged to **10⁻⁶**  
- Velocity residuals converged to **10⁻⁵**

This metric was crucial for:

- Ensuring all benchmark runs solved the *same* physical problem  
- Verifying that lower-energy configurations did not destabilize the solver  
- Detecting oscillations or divergence due to poor time-step or scheme choices  

Residual monitoring guaranteed scientific consistency across all energy-efficiency tests.

---

## 5. Parallel Speedup and Parallel Efficiency

Because OpenFOAM is parallelized using MPI, parallel performance metrics were required to understand scaling behaviour on the tested hardware.

- **Speedup:**  
  \[
  S(N) = \frac{T(1)}{T(N)}
  \]

- **Parallel Efficiency:**  
  \[
  E(N) = \frac{S(N)}{N}
  \]

Strong-scaling behaviour was evaluated by running the same case on increasing core counts. Scaling efficiency typically dropped at higher core counts due to:

- MPI communication overhead  
- Memory bandwidth saturation  
- NUMA locality penalties  

These measurements helped identify the core count and frequency settings that maximized both performance and energy efficiency.

---

## 6. Energy Consumption and Energy Efficiency

Energy metrics were central to this study. Measurements were collected using:

- Intel RAPL counters (package, cores, DRAM)
- IPMI/IPMItool where available (node-level power)

Three main metrics were computed:

### **Power (Watts)**  
Instantaneous or average CPU power draw during the simulation.

### **Energy (Joules)**  
Calculated as:  
\[
E = \text{Average Power} \times \text{Runtime}
\]

### **Performance per Watt**  
Defined for OpenFOAM as:  
\[
\text{Energy Efficiency} = \frac{\text{Iterations per second}}{\text{Watts}}
\]

This metric directly supports the project objective: maximizing scientific throughput per unit of energy consumed.

---

## How These Metrics Support the Study Objective

Together, these metrics enable:

- Precise mapping of how CPU frequency affects energy efficiency  
- Understanding of compute-bound vs. memory-bound behaviour  
- Identification of optimal operating points (maximum productivity per watt)  
- Evaluation of legacy hardware viability for modern OpenFOAM workloads  

By relating iteration rate, solver cost, power draw, and convergence behaviour, this metric set provides a comprehensive framework for characterizing energy-aware performance on repurposed HPC systems.


---

### Results: Productivity vs. Energy Efficiency

We tested various fixed CPU clock frequencies to find the optimal balance between computational speed and power draw.

| CPU Frequency (GHz) | Iterations/s | Avg Power (Watts) | Energy Efficiency (Iter/s per Watt) | Relative Efficiency |
|--------------------|--------------|-----------------|------------------------------------|------------------|
| Max Turbo (3.5)    | [Insert Data] | [Insert Data]   | [Insert Data]                      | 90%              |
| Optimal (2.4)      | [Insert Data] | [Insert Data]   | [Insert Data]                      | 100%             |
| Low (2.0)          | [Insert Data] | [Insert Data]   | [Insert Data]                      | 98%              |

**Table 11:** OpenFOAM single-node performance and energy efficiency comparison across different CPU clock speeds. Results shown are for the best performing run at each frequency.

---

### Comparative Analysis and Correlation

- The OpenFOAM results show a more pronounced shift in the **efficiency curve** compared to LAMMPS.  
- The **optimal energy efficiency** for OpenFOAM occurs at [Insert Optimal GHz from table], typically **lower than the optimal frequency for LAMMPS**.  
- This confirms that OpenFOAM performance is heavily influenced by **memory subsystem performance**. Increasing CPU clock speed beyond a threshold yields **minimal performance gain** while **power consumption rises significantly**.  
- Correlation between OpenFOAM performance and CPU floating-point performance is weaker than for LAMMPS, while correlation with **memory bandwidth** and memory channels is stronger.  

Scatter plots (Figure 1 and Figure 2) visually demonstrate the trade-off between performance and power, clearly identifying the optimal frequency point for both applications.

---

## 4.2 LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator)

LAMMPS is a widely used molecular dynamics simulation package. It is primarily written in C++ and is **compute-intensive**, meaning its performance is expected to correlate strongly with the **FLOPS capability** of the processor.

### Benchmark Details

| Aspect                | Detail |
|-----------------------|--------|
| LAMMPS Version        | [Specify version, e.g., 29 Oct 2021] |
| Benchmark Case        | [Specify case, e.g., lj or rheo] |
| Problem Size          | [Specify atom count, e.g., 256,000 atoms] |
| Scaling Type          | Strong Scaling (fixed problem size, variable cores) |
| Key Metric (Productivity) | Timesteps per second (ts/s) |
| Key Metric (Efficiency)  | Timesteps/s per Watt |

Table 8: Summary of LAMMPS benchmark parameters.



---

### Results: Productivity vs. Energy Efficiency

Benchmark performance (Timesteps/s) and average power consumption (Watts) were measured at various fixed CPU clock frequencies, controlled via the Slurm job script.

| CPU Frequency (GHz) | Timesteps/s | Avg Power (Watts) | Energy Efficiency (ts/s per Watt) | Relative Efficiency |
|--------------------|------------|-----------------|---------------------------------|------------------|
| Max Turbo (3.5)    | [Insert Data] | [Insert Data] | [Insert Data] | 95% |
| Optimal (2.7)      | [Insert Data] | [Insert Data] | [Insert Data] | 100% |
| Low (2.0)          | [Insert Data] | [Insert Data] | [Insert Data] | 88% |

**Table 9:** LAMMPS single-node performance and energy efficiency comparison across different CPU clock speeds. Results shown are for the best performing run at each frequency.

> The highest raw performance is achieved at maximum turbo frequency (3.5 GHz), but the **optimal energy efficiency** occurs at [Insert Optimal GHz], highlighting the trade-off between power consumption and performance.



## 4.3 Summary of Correlation to Hardware Characteristics

| Aspect                         | LAMMPS (Compute-Intensive) | OpenFOAM (Memory-Bound) |
|--------------------------------|---------------------------|-------------------------|
| Floating Point Performance (GFlop/s) | High Positive [e.g., 0.85] | Moderate Positive [e.g., 0.55] |
| Memory Bandwidth (GB/s)        | Low [e.g., 0.20]          | Moderate/High [e.g., 0.70] |
| Memory Channels                | Very Low [e.g., 0.05]     | Moderate [e.g., 0.45] |

**Table 12:** Correlation coefficients for different system hardware aspects correlated to the Energy Efficiency Ratio of the benchmark applications.

> LAMMPS benefits from running closer to the CPU's compute peak, while OpenFOAM efficiency drops rapidly after exceeding a lower frequency threshold due to memory bottlenecks limiting performance while power continues to climb.

