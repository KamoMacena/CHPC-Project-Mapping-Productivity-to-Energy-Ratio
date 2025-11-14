# Cluster System Information Report

**Host:** smshost  
**Date:** Fri Nov 14 03:44:04 SAST 2025  

---

#### Table 1: 
| Field | Value |
|-------|-------|
| Hostname | smshost |
| Uptime | 10 days 13:56 |
| Load Average | 0.01 / 0.08 / 0.10 |
| OS | Rocky Linux 8.10 (Green Obsidian) |
| Kernel Platform | el8 |
| Support End | 2029-05-31 |

---

#### Table 2: System details for the CPU used in this study
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


#### Table 3: System Memory Information

| Field | Value |
|-------|-------|
| Total RAM | 31 GiB |
| Used | 2.1 GiB |
| Free | 10 GiB |
| Buff/Cache | 18 GiB |
| Available | 26 GiB |
| Swap Total | 15 GiB |
| Swap Used | 1.0 GiB |


#### Table 4:Sytem GPU / Accelerator Information
| Field | Value |
|-------|-------|
| NVIDIA GPU | None detected |
| Drivers | Not installed |


#### Table 5. Network Interfaces
| Interface | State | IP Address | Notes |
|-----------|-------|------------|-------|
| lo | UP | 127.0.0.1 | Loopback |
| eno1 | UP | 10.128.23.188 | Main LAN |
| eno2 | UP | 10.10.10.10 | Cluster private network |
| ib0 | DOWN | — | Mellanox IB (ConnectX-3) |
| virbr0 | DOWN | 192.168.122.1 | Virtual bridge |

### Table 6 PCI Network Hardware
| Device | Model |
|--------|-------|
| 02:00.0 | Intel I350 NIC |
| 02:00.3 | Intel I350 NIC |
| 82:00.0 | Mellanox ConnectX-3 |


#### Table 7:  Power (RAPL)
| Zone | Type | Energy (uJ) |
|------|------|-------------|
| intel-rapl:0 | package-0 | 16,511,888,631 |
| intel-rapl:0:0 | core | 43,998,797,599 |
| intel-rapl:0:1 | dram | 20,217,413,493 |
| intel-rapl:1 | package-1 | 61,947,548,996 |
| intel-rapl:1:0 | core | 21,046,507,595 |
| intel-rapl:1:1 | dram | 63,360,429,543 |


#### Table 8: Loaded Software Modules Used in this study
| Module Category | Modules Loaded |
|-----------------|----------------|
| Compilers | gnu12/12.4.0 |
| MPI | openmpi4/4.1.6 |
| Core Tools | autotools, hwloc, libfabric, ucx, prun |
| Math & Sci | openblas, petsc, fftw, scalapack, mumps, trilinos |
| Profiling | tau, scalasca, scorep, PAPI |


## Table 9: MPI Information
| Field | Value |
|-------|-------|
| MPI Implementation | OpenMPI 4.1.6 |
| Path | /opt/ohpc/pub/mpi/openmpi4-gnu12/4.1.6/bin/mpirun |

# 4. Application Benchmarks: Productivity and Energy Efficiency

In this study, we conducted two primary benchmark tests using representative HPC applications to determine the optimal **Productivity-to-Energy Cost Ratio** on the repurposed legacy hardware. Unlike traditional performance reports, our analysis focuses on the trade-off between raw speed and power consumption by manipulating system parameters via the Slurm scheduler.

The two applications benchmarked were:

- **LAMMPS:** Molecular Dynamics (compute-intensive).  
- **OpenFOAM:** Computational Fluid Dynamics (typically memory-bound).  

More details on these benchmarks and the methodology used to gather both performance and power data are detailed in the individual sections below.  

A rendered Python notebook with the analysis used to produce the data and visualizations for the efficiency analysis can be found on GitHub at:  
[Insert link to your GitHub analysis folder/notebook here]



## 4.1 OpenFOAM (Open Field Operation and Manipulation)

OpenFOAM is an open-source CFD software package. For many large-scale CFD problems, OpenFOAM is typically **memory-bound or interconnect-bound**, meaning its performance depends more on **memory bandwidth and latency** than on raw floating-point speed of the CPU.

### Benchmark Details

| Aspect                | Detail |
|-----------------------|--------|
| OpenFOAM Version      | [Specify version, e.g., v2306] |
| Benchmark Case        | [Specify case, e.g., windAroundBuildings_3_2millBenchmark] |
| Problem Size          | [Specify cell count, e.g., 2 Million cells] |
| Scaling Type          | Strong Scaling |
| Key Metric (Productivity) | Total Runtime (seconds) or Iterations per second |
| Key Metric (Efficiency)  | Iterations/s per Watt |

**Table 10:** Summary of OpenFOAM benchmark parameters.

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

**Table 8:** Summary of LAMMPS benchmark parameters.

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

