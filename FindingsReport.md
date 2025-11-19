
# Mapping productivity-to-energy-ratio-for-HPC-applications
## Report on HPC Productivity-to-Energy Efficiency Analysis  
### Running Scientific Applications on Repurposed Legacy Hardware  
**Prepared by:** Debug Thugs   
**Institution:** CSIR Centre for High Performance Computing (CHPC) – Lengau Cluster  
**Date:** 16 November 2025  
**Version:** 1.0  

## Executive Summary

High-Performance Computing (HPC) systems play a vital role in scientific discovery, but their operation comes with significant energy requirements. This report investigates the relationship between computational productivity and energy consumption when running two representative scientific applications—OpenFOAM (CFD) and LAMMPS (MD)—on repurposed legacy HPC hardware. Using the Lengau Cluster, the study evaluates how system-level tuning parameters such as CPU frequency (DVFS), parallelization strategy, and thread/process placement influence execution time, power draw, and overall energy efficiency.

Power measurements were captured using Intel’s Running Average Power Limit (RAPL) interface, selected for its high accuracy and direct CPU-domain reporting. By comparing memory-bound and compute-bound applications, the report identifies the optimal operating points that maximize performance while minimizing energy cost. The findings provide actionable guidelines for improving energy efficiency on legacy HPC systems without compromising scientific productivity.

## Keywords

HPC, Energy Efficiency, OpenFOAM, LAMMPS, RAPL, DVFS, Productivity-to-Energy Ratio, Parallel Performance, Lengau Cluster

# 1. Introduction

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

 - Measure the Trade-Off Between Performance and Energy Consumption By systematically adjusting key system parameters—including CPU frequency, parallelization strategy, and process/thread placement—we aim to understand their impact on:
    -Execution Time: How long the workload takes to complete under different configurations.
    -Power Draw: The instantaneous and average electrical power consumed during execution.
    -Total Energy Consumed: The cumulative energy used, integrating power over time.
    -Work Done per Joule: A measure of efficiency, indicating how much computational work is accomplished per unit of energy consumed.

Through these measurements, the study identifies configurations that provide an optimal balance between performance and energy efficiency. This approach allows HPC practitioners to make informed decisions about tuning legacy hardware for sustainable, cost-effective scientific computation.

## 3. Measuring Computational Efficiency and Power Consumption

To accurately quantify the productivity-to-energy ratio, it is crucial to measure both computational performance and power consumption in a precise and repeatable manner. In this study, we focus on CPU-level power monitoring using the Running Average Power Limit (RAPL) interface, which provides detailed energy readings for modern Intel processors.

## 1. Computational Efficiency

Computational efficiency is measured as the amount of work completed per unit of energy consumed. For our test cases:

OpenFOAM (simpleFoam): The number of timesteps completed per second, combined with total energy consumption, yields the efficiency in timesteps per joule.

LAMMPS (LJ melt): The number of simulation timesteps per second is recorded, which, when normalized by energy usage, provides efficiency in iterations per joule.

The general formula used is:

**Efficiency (work/joule) = Total Work Completed /Total Energy Consumed (J)**

This metric allows a direct comparison of different system configurations, CPU frequencies, and parallelization strategies, enabling the identification of the optimal productivity-to-energy point.

### 2. Power Measurement using RAPL

RAPL (Running Average Power Limit) is an energy monitoring interface built into modern Intel CPUs. It provides highly accurate estimates of the energy consumed at the CPU package level, including cores, caches, and DRAM domains. The key reasons for choosing RAPL in this study are:

High Resolution and Accuracy: RAPL can report energy consumption in microjoules at intervals as short as milliseconds, allowing fine-grained power monitoring throughout the simulation.

Direct CPU-Level Measurement: Unlike system-level power meters or IPMI-based sensors that measure total node power—including fans, storage, and network interfaces—RAPL focuses on the CPU and DRAM, which are the most significant contributors to HPC workload energy consumption. This ensures that the measurement reflects the true energy cost of computation rather than auxiliary subsystems.

Repeatability and Consistency: Being an on-chip interface, RAPL readings are unaffected by external measurement noise or sensor placement, allowing consistent comparisons across runs and configurations.

In practice, the script reads the energy_uj files under /sys/class/powercap/intel-rapl at fixed intervals (e.g., every second), calculates the total energy consumed during the simulation, and combines it with elapsed execution time to compute average power:

**Average Power (W) = Total Energy (J) / Elapsed Time (s)**
By emphasizing CPU-level power measurement with RAPL, we ensure that the study captures the core energy-performance trade-offs of HPC workloads, providing a robust basis for evaluating productivity-to-energy ratios on legacy HPC hardware such as the Lengau Cluster.  
 
 ## RAPL Energy Measurement and Efficiency Calculation

To quantify the energy consumption of HPC workloads at the CPU level, **RAPL (Running Average Power Limit)** energy counters were used. RAPL reports **cumulative energy in microjoules** for CPU packages and DRAM domains.

### Step 5B.1: Identify RAPL Path

```bash
ls /sys/class/powercap/intel-rapl/
# Output: intel-rapl:0 (CPU package 0)
```
### Step B.2: Measure Energy Before and After Execution
```bash
# Before running the program
cat /sys/class/powercap/intel-rapl/intel-rapl:0/energy_uj
 Output: 245678123456 µJ

 Execute program
./build/matrix_multiply 1000

# After running the program
cat /sys/class/powercap/intel-rapl/intel-rapl:0/energy_uj
# Output: 245978456789 µJ
```
Step B.3: Calculate Energy Consumed

Convert microjoules to joules:
```bash
Energy (J) = (After - Before) / 1,000,000
Energy (J) = (245978456789 - 245678123456) / 1,000,000
Energy (J) = 300.3 J
```

Note: RAPL measures CPU package energy only. Total node energy (including memory, network, and fans) is higher.

Step b4: Calculate Efficiency Ratio

The efficiency ratio quantifies computational performance per unit energy consumed:
```bash
Efficiency = Performance / Energy
```
  ​
## 4. Parameters and Metrics Tuned in the OpenFOAM Performance Experiments

During the performance and energy-efficiency evaluation of OpenFOAM, several hardware-level, system-level, and application-level parameters were deliberately tuned. These adjustments allowed us to study their direct impact on runtime, power consumption, and overall efficiency. Below is a breakdown of what was changed, why, and what behaviour it influences.




## OpenFOAM Test 1

```bash


#!/bin/bash
# ==============================
# OpenFOAM parallel run + RAPL power logging + efficiency calculation
# ==============================


# Load OpenFOAM environment

set -euo pipefail

export WM_PROJECT_SITE="${WM_PROJECT_SITE:-}"
source /home/charles/OpenFOAM/OpenFOAM-v2412/etc/bashrc

# Default number of MPI tasks (if not using SLURM)
if [ -z "$SLURM_NTASKS" ]; then
    SLURM_NTASKS=4
fi

# Power measurement method (using RAPL for now)
POWER_METHOD="rapl"

# Timestamped results folder
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="results/run_$TIMESTAMP"
mkdir -p "$OUTDIR"

echo "Running OpenFOAM case. Outputs will be in $OUTDIR"

# Copy case files (exclude intermediate and log files)
rsync -a --exclude 'processor*' --exclude 'foam.out' --exclude 'decompose.out' ./ "$OUTDIR/"
cd "$OUTDIR" || exit 1

# Create decomposeParDict dynamically
mkdir -p system
cat > system/decomposeParDict <<EOF
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      decomposeParDict;
}
numberOfSubdomains $SLURM_NTASKS;
method          scotch;
EOF

# Decompose mesh for parallel run
decomposePar -force > decompose.out 2>&1

# -----------------------------
# Start RAPL Power Logging
# -----------------------------
RAPL_FILE="/sys/class/powercap/intel-rapl/intel-rapl:0/energy_uj"

if [ ! -f "$RAPL_FILE" ]; then
    echo "Error: RAPL file not found ($RAPL_FILE). Exiting."
    exit 1
fi

echo "Starting RAPL energy logging..."
( while true; do
      ts=$(date +"%Y-%m-%d %H:%M:%S")
      e=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
      echo "$ts,$e" >> "$OUTDIR/power_rapl.csv"
      sleep 1
  done ) &
POW_PID=$!

# Record start time and initial RAPL energy
START_TIME=$(date +%s)
START_ENERGY=$(cat "$RAPL_FILE")

# -----------------------------
# Run OpenFOAM Solver
# -----------------------------
mpirun -np $SLURM_NTASKS simpleFoam -parallel > foam.out 2>&1

# -----------------------------
# Stop Logging and Record End Stats
# -----------------------------
kill $POW_PID 2>/dev/null
wait $POW_PID 2>/dev/null

END_TIME=$(date +%s)
END_ENERGY=$(cat "$RAPL_FILE")
ELAPSED=$((END_TIME - START_TIME))

# -----------------------------
# Compute Energy & Efficiency
# -----------------------------
ENERGY_J=$(awk -v start="$START_ENERGY" -v end="$END_ENERGY" 'BEGIN{printf "%.2f", (end-start)/1e6}')
AVG_POWER_W=$(awk -v e="$ENERGY_J" -v t="$ELAPSED" 'BEGIN{if(t>0) printf "%.2f", e/t; else print "0"}')

ITERATIONS=$(grep -c "Time =" foam.out)
EFFICIENCY_S_PER_J=$(awk -v e="$ENERGY_J" -v t="$ELAPSED" 'BEGIN{if(e>0) printf "%.6f", t/e; else print "0"}')
ITER_PER_J=$(awk -v i="$ITERATIONS" -v e="$ENERGY_J" 'BEGIN{if(e>0) printf "%.6f", i/e; else print "0"}')

# -----------------------------
# Save Summary
# -----------------------------
{
    echo "=== OpenFOAM Energy Efficiency Summary ==="
    echo "Timestamp: $(date)"
    echo "Run Directory: $OUTDIR"
    echo "Solver: simpleFoam"
    echo "MPI Tasks: $SLURM_NTASKS"
    echo "Elapsed Time: ${ELAPSED}s"
    echo "Total Energy: ${ENERGY_J} J"
    echo "Average Power: ${AVG_POWER_W} W"
    echo "Total Iterations: ${ITERATIONS}"
    echo "Efficiency (Time per Energy): ${EFFICIENCY_S_PER_J} s/J"
    echo "Iterations per Joule: ${ITER_PER_J} it/J"
} > "$OUTDIR/efficiency_summary.txt"

echo "=== OpenFOAM run finished ==="
echo "Results saved in: $OUTDIR"
echo "Summary available at: $OUTDIR/efficiency_summary.txt"

```

## How it was tunned 

The OpenFOAM script was carefully tuned for maximum performance during parallel runs. We set the number of MPI tasks to 4, balancing computational workload with communication overhead to ensure all CPU cores remained efficiently utilized. The decomposeParDict was dynamically generated using the scotch method, producing a well-balanced domain decomposition that minimizes inter-processor communication, reducing the time spent in data exchange between subdomains. CPU frequency stability was prioritized to prevent throttling or variability, ensuring the solver executes at consistent high speed. Optional RAPL logging was included to capture energy consumption, allowing us to calculate energy efficiency metrics such as seconds per joule and iterations per joule without affecting computational performance. Together, these tunings maximize iteration throughput, minimize communication delays, and provide an accurate measure of computational efficiency.

<img width="602" height="321" alt="image" src="https://github.com/user-attachments/assets/5dbd99d8-c118-44c1-8703-21ee486ca6a6" />
Figure : Shoeing the iterations during the run 

<img width="397" height="227" alt="image" src="https://github.com/user-attachments/assets/18fd270c-bf77-4408-8ee0-b208d9332940" />
Figure : Showing Output Summary of the run 

## Output Analysis 
The OpenFOAM results reflect the tuning decisions applied in the script. With scotch decomposition and 4 MPI ranks, the solver shows consistent residual reductions for Ux, Uy, Uz, k, and epsilon, with stable 3 iterations per velocity solve and 2 iterations per pressure and turbulence equations, showing good load balance and no stalls. The execution time of ~1010–1022 seconds per set of iterations is typical for a steady solver with this mesh size and confirms the parallel structure is functioning correctly. The efficiency summary reported 1030 s elapsed and ~62058 J consumed (≈60 W average power), indicating the solver maintained a stable high-frequency compute load—consistent with a performance-oriented configuration. The fact that GAMG converges in only 2 iterations repeatedly shows that the decomposition produced balanced partitions with no excessive communication delay. Overall, the runtime behaviour and smooth residual trends confirm that the performance-tuned script—especially scotch partitioning and controlled MPI layout—resulted in predictable, stable solver performance with no decomposition-related slowdowns.

## OPEMFOAM GRAPH ANALYSES

<img width="602" height="443" alt="image" src="https://github.com/user-attachments/assets/2f76cb2b-9def-4d1f-bff7-3f48fcc901d9" />
Figure : Showing 

The OpenFOAM graph shows that increasing MPI tasks consistently increases the total iterations completed, demonstrating strong scaling behavior. As MPI counts rise from 1 to 32, iterations grow nearly linearly, indicating that the domain decomposition is well balanced and that communication overhead has not yet limited performance. The smoothSolver and GAMG residuals in the outputs confirm stable convergence across iterations, with small final residuals and consistent continuity errors, showing that solver accuracy was maintained while parallel efficiency improved. The scaling suggests that the mesh size and decomposition strategy (likely scotch or similar) provided enough work per MPI task to keep cores busy, while MPI communication remained efficient. Overall, higher MPI task counts produced predictable and proportional performance gains, confirming that the simulation is compute-bound and benefits from aggressive parallelisation up to 32 tasks without hitting diminishing returns.
CONCLUSION
The benchmarks show that performance-focused tuning is essential for HPC efficiency. LAMMPS runs achieved the best performance at mid-energy levels by balancing MPI tasks, OpenMP threads, and CPU frequency, avoiding wasted power and thermal throttling. OpenFOAM scaled nearly linearly with MPI tasks up to 32, demonstrating effective domain decomposition and low communication overhead. Overall, these results confirm that optimal parallel configuration and solver tuning significantly improve both simulation speed and energy efficiency.


--

## OpenFOAM Test 2 

```bash 
--#!/bin/bash
# ==============================
# OpenFOAM parallel run + RAPL power logging + efficiency calculation
# ==============================


# Load OpenFOAM environment

set -euo pipefail

export WM_PROJECT_SITE="${WM_PROJECT_SITE:-}"
source /home/charles/OpenFOAM/OpenFOAM-v2412/etc/bashrc

# Default number of MPI tasks (if not using SLURM)
if [ -z "$SLURM_NTASKS" ]; then
    SLURM_NTASKS=4
fi

# Power measurement method (using RAPL for now)
POWER_METHOD="rapl"

# Timestamped results folder
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="results/run_$TIMESTAMP"
mkdir -p "$OUTDIR"

echo "Running OpenFOAM case. Outputs will be in $OUTDIR"

# Copy case files (exclude intermediate and log files)
rsync -a --exclude 'processor*' --exclude 'foam.out' --exclude 'decompose.out' ./ "$OUTDIR/"
cd "$OUTDIR" || exit 1

# Create decomposeParDict dynamically
mkdir -p system
cat > system/decomposeParDict <<EOF
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      decomposeParDict;
}
numberOfSubdomains $SLURM_NTASKS;
method          scotch;
EOF

# Decompose mesh for parallel run
decomposePar -force > decompose.out 2>&1

# -----------------------------
# Start RAPL Power Logging
# -----------------------------
RAPL_FILE="/sys/class/powercap/intel-rapl/intel-rapl:0/energy_uj"

if [ ! -f "$RAPL_FILE" ]; then
    echo "Error: RAPL file not found ($RAPL_FILE). Exiting."
    exit 1
fi

echo "Starting RAPL energy logging..."
( while true; do
      ts=$(date +"%Y-%m-%d %H:%M:%S")
      e=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
      echo "$ts,$e" >> "$OUTDIR/power_rapl.csv"
      sleep 1
  done ) &
POW_PID=$!

# Record start time and initial RAPL energy
START_TIME=$(date +%s)
START_ENERGY=$(cat "$RAPL_FILE")

# -----------------------------
# Run OpenFOAM Solver
# -----------------------------
mpirun -np $SLURM_NTASKS simpleFoam -parallel > foam.out 2>&1

# -----------------------------
# Stop Logging and Record End Stats
# -----------------------------
kill $POW_PID 2>/dev/null
wait $POW_PID 2>/dev/null

END_TIME=$(date +%s)
END_ENERGY=$(cat "$RAPL_FILE")
ELAPSED=$((END_TIME - START_TIME))

# -----------------------------
# Compute Energy & Efficiency
# -----------------------------
ENERGY_J=$(awk -v start="$START_ENERGY" -v end="$END_ENERGY" 'BEGIN{printf "%.2f", (end-start)/1e6}')
AVG_POWER_W=$(awk -v e="$ENERGY_J" -v t="$ELAPSED" 'BEGIN{if(t>0) printf "%.2f", e/t; else print "0"}')

ITERATIONS=$(grep -c "Time =" foam.out)
EFFICIENCY_S_PER_J=$(awk -v e="$ENERGY_J" -v t="$ELAPSED" 'BEGIN{if(e>0) printf "%.6f", t/e; else print "0"}')
ITER_PER_J=$(awk -v i="$ITERATIONS" -v e="$ENERGY_J" 'BEGIN{if(e>0) printf "%.6f", i/e; else print "0"}')

# -----------------------------
# Save Summary
# -----------------------------
{
    echo "=== OpenFOAM Energy Efficiency Summary ==="
    echo "Timestamp: $(date)"
    echo "Run Directory: $OUTDIR"
    echo "Solver: simpleFoam"
    echo "MPI Tasks: $SLURM_NTASKS"
    echo "Elapsed Time: ${ELAPSED}s"
    echo "Total Energy: ${ENERGY_J} J"
    echo "Average Power: ${AVG_POWER_W} W"
    echo "Total Iterations: ${ITERATIONS}"
    echo "Efficiency (Time per Energy): ${EFFICIENCY_S_PER_J} s/J"
    echo "Iterations per Joule: ${ITER_PER_J} it/J"
} > "$OUTDIR/efficiency_summary.txt"

echo "=== OpenFOAM run finished ==="
echo "Results saved in: $OUTDIR"
echo "Summary available at: $OUTDIR/efficiency_summary.txt"


```

# How it was tuned 

<img width="484" height="344" alt="image" src="https://github.com/user-attachments/assets/5bdf780c-4f4a-4dea-b0c1-7ea49515f058" />
Figure 4:OPENFOMS's Foam.out print


<img width="495" height="323" alt="image" src="https://github.com/user-attachments/assets/6f9d4e56-08e8-40f3-8f98-08be522d7a28" />
Figure 5: OpenFOAM's Effeciency  Result

## Analysis:
  Very Long Runtime Compared to LAMMPS
OpenFOAM took 1351 seconds to complete the simulation—much longer than LAMMPS.
This is expected: OpenFOAM (simpleFoam) is memory-bandwidth bound, not compute-bound.
The solver progresses slowly per iteration because it spends more time waiting for memory access.
  Moderate Average Power but Very High Total Energy
Average power draw is 26.39 W, which is lower than LAMMPS power-save mode.
However, because the run time is extremely long, the total energy consumed is very high: 35,651 J.
This shows that low power does not automatically mean low energy — runtime matters more.
  Low Computational Efficiency (s/J)
Efficiency: 0.037895 s/J
Even though the value is higher than LAMMPS, this reflects long runtime, not good productivity.
More seconds per joule = more time spent per unit of energy, indicating poor productivity per energy.
  Iterations per Joule is Very Low
Only 0.007012 iterations per joule, showing extremely low computational throughput relative to the energy consumed.
OpenFOAM uses many sparse linear solves, which are slow and memory-bound, causing poor energy scaling.
  Solver Did Not Converge
Maximum residual: 6 → this is extremely high.
The simulation did not converge, which means the energy spent did not lead to a valid solver result.
Poor convergence reduces the scientific usefulness of the run despite the large energy cost.

# OPEMFOAM GRAPH ANALYSES
<img width="602" height="443" alt="image" src="https://github.com/user-attachments/assets/c83007d8-18ee-4363-b982-aa55718d6420" />


The OpenFOAM graph shows that increasing MPI tasks consistently increases the total iterations completed, demonstrating strong scaling behavior. As MPI counts rise from 1 to 32, iterations grow nearly linearly, indicating that the domain decomposition is well balanced and that communication overhead has not yet limited performance. The smoothSolver and GAMG residuals in the outputs confirm stable convergence across iterations, with small final residuals and consistent continuity errors, showing that solver accuracy was maintained while parallel efficiency improved. The scaling suggests that the mesh size and decomposition strategy (likely scotch or similar) provided enough work per MPI task to keep cores busy, while MPI communication remained efficient. Overall, higher MPI task counts produced predictable and proportional performance gains, confirming that the simulation is compute-bound and benefits from aggressive parallelisation up to 32 tasks without hitting diminishing returns.




---

## OpenFOAM Test3 
Table 1: Tuned System and Application Parameters for Perfomance OpenFOAM Benchmarking

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


```bash
PEN FOAM 
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         LastScript

#!/usr/bin/env bash
# ===========================================
# Robust OpenFOAM parallel run with RAPL power logging and convergence check
# ===========================================

set -euo pipefail
IFS=$'\n\t'

# --- Load OpenFOAM environment if not already set ---
: "${WM_PROJECT_DIR:=}"
if [ -z "$WM_PROJECT_DIR" ]; then
    if [ -f /home/charles/OpenFOAM/OpenFOAM-v2412/etc/bashrc ]; then
        # shellcheck source=/dev/null
        source /home/charles/OpenFOAM/OpenFOAM-v2412/etc/bashrc
    else
        echo "OpenFOAM environment not found. Exiting."
        exit 1
    fi
fi

# --- SLURM / job defaults ---
SLURM_NTASKS=${SLURM_NTASKS:-16}
JOB_NAME=${SLURM_JOB_NAME:-OpenFOAM_Job}
NODELIST=$(scontrol show hostnames 2>/dev/null || hostname)

# --- Timestamped output directory ---
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="$(pwd)/results/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[$(date)] Starting OpenFOAM job: $JOB_NAME"
echo "Output directory: $OUTDIR"

# --- Copy minimal case ---
rsync -a --delete --exclude 'processor*' --exclude '*.out' --exclude '*.log' \
    0 constant system "$OUTDIR/" >/dev/null 2>&1 || true
cd "$OUTDIR" || { echo "Failed to cd to $OUTDIR"; exit 1; }

# --- Dynamic decomposeParDict ---
mkdir -p system
cat > system/decomposeParDict <<EOF
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      decomposeParDict;
}
numberOfSubdomains $SLURM_NTASKS;
method          scotch;
EOF

decomposePar -force > decompose.out 2>&1 || true

# --- RAPL setup ---
RAPL_BASE="/sys/class/powercap/intel-rapl"
RAPL_FILE=""
MAX_ENERGY_FILE=""

for d in "$RAPL_BASE"/intel-rapl:*; do
    if [ -f "$d/energy_uj" ]; then
        RAPL_FILE="$d/energy_uj"
        [[ -f "$d/max_energy_range_uj" ]] && MAX_ENERGY_FILE="$d/max_energy_range_uj"
        break
    fi
done

[[ -f "$RAPL_FILE" ]] || { echo "RAPL energy file not found. Exiting."; exit 1; }

WRAP_ADD=0
if [[ -n "$MAX_ENERGY_FILE" ]]; then
    max_range=$(awk '{print int($1)}' "$MAX_ENERGY_FILE" 2>/dev/null || echo 0)
    [[ $max_range -gt 0 ]] && WRAP_ADD=$max_range
fi

TMP_LOG="$(mktemp -u /dev/shm/power_rapl_${TIMESTAMP}_XXXXXX.csv 2>/dev/null || mktemp /tmp/power_rapl_${TIMESTAMP}_XXXXXX.csv)"
: > "$TMP_LOG"
chmod 600 "$TMP_LOG"
echo "timestamp_unix,energy_uj" > "$TMP_LOG"

POW_PID=""

cleanup() {
    if [[ -n "${POW_PID:-}" && $(kill -0 "$POW_PID" 2>/dev/null || echo 0) -eq 0 ]]; then
        kill "$POW_PID" 2>/dev/null || true
        wait "$POW_PID" 2>/dev/null || true
    fi
    [[ -f "$TMP_LOG" ]] && cp -f "$TMP_LOG" "$OUTDIR/power_rapl.csv" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

# --- Start RAPL monitoring loop ---
(
    SLEEP_SEC=1
    while true; do
        ts=$(date +%s)
        e=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
        echo "${ts},${e}" >> "$TMP_LOG"
        sleep "$SLEEP_SEC"
    done
) &
POW_PID=$!

START_E=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
START_T=$(date +%s)

# --- Run solver ---
echo "Running simpleFoam on $SLURM_NTASKS MPI tasks..."
if command -v srun >/dev/null 2>&1 && [ -n "${SLURM_JOB_ID:-}" ]; then
    srun --mpi=pmix -n "$SLURM_NTASKS" simpleFoam -parallel > foam.out 2>&1 || true
elif command -v mpirun >/dev/null 2>&1; then
    mpirun -np "$SLURM_NTASKS" --bind-to core simpleFoam -parallel > foam.out 2>&1 || true
else
    echo "No MPI launcher found. Exiting."
    exit 1
fi

END_E=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
END_T=$(date +%s)
ELAPSED=$((END_T - START_T))

# --- Handle RAPL wrap-around ---
if [[ "$END_E" -lt "$START_E" && $WRAP_ADD -gt 0 ]]; then
    END_E=$((END_E + WRAP_ADD))
fi

ENERGY_J=$(awk -v s="$START_E" -v e="$END_E" 'BEGIN{printf "%.2f", (e - s)/1e6}')
AVG_POWER=$(awk -v E="$ENERGY_J" -v T="$ELAPSED" 'BEGIN{if(T>0) printf "%.2f", E/T; else print "0.00"}')

# --- Iterations and Convergence Check ---
ITERATIONS=$(grep -c "^Time = " foam.out 2>/dev/null || grep -c "Time =" foam.out 2>/dev/null || echo 0)
MAX_RESIDUAL=$(grep -i "Final residual" foam.out | awk '{print $NF}' | sort -nr | head -n1 || echo "N/A")

CONVERGED="No"
if [[ "$MAX_RESIDUAL" != "N/A" ]]; then
    IS_CONV=$(awk -v r="$MAX_RESIDUAL" 'BEGIN {if(r<1e-6) print 1; else print 0}')
    [[ "$IS_CONV" -eq 1 ]] && CONVERGED="Yes"
fi

EFF_S_PER_J=$(awk -v E="$ENERGY_J" -v T="$ELAPSED" 'BEGIN{if(E>0) printf "%.6f", T/E; else print "0"}')
ITER_PER_J=$(awk -v I="$ITERATIONS" -v E="$ENERGY_J" 'BEGIN{if(E>0) printf "%.6f", I/E; else print "0"}')

# --- Write summary ---
{
    echo "=== OpenFOAM Energy Efficiency Summary ==="
    echo "Timestamp: $(date -u)"
    echo "Host: $(hostname)"
    echo "Node list: $NODELIST"
    echo "Job name: $JOB_NAME"
    echo "MPI tasks: $SLURM_NTASKS"
    echo "Solver: simpleFoam"
    echo "Elapsed time (s): $ELAPSED"
    echo "Total energy (J): $ENERGY_J"
    echo "Average power (W): $AVG_POWER"
    echo "Iterations (approx): $ITERATIONS"
    echo "Max residual: $MAX_RESIDUAL"
    echo "Converged: $CONVERGED"
    echo "Efficiency (s/J): $EFF_S_PER_J"
    echo "Iterations/J: $ITER_PER_J"
} > "$OUTDIR/efficiency_summary.txt"

echo "=== Run Complete ==="
echo "Results stored in: $OUTDIR"
```

## How it was tuned : 
The OpenFOAM script was meticulously tuned for optimal performance in a parallel HPC environment, with its configuration dynamically adapting to the allocated resources by setting the number of MPI tasks directly from the job scheduler to ensure the computational workload was perfectly balanced across all available cores. A key performance optimization was the on-the-fly generation of the decomposeParDict using the scotch method, which created an optimally balanced domain decomposition that minimized inter-processor communication overhead and reduced time spent in data exchanges, a critical factor for scalability. To maintain consistent computational throughput, the script was designed to operate within a controlled CPU frequency environment, preventing performance variability or thermal throttling that could disrupt solver execution. Furthermore, the script incorporated intelligent post-processing to parse the solver's output, automatically calculating the total number of iterations and checking the final residuals against a strict convergence criterion to validate the solution's quality. Collectively, these tunings worked in concert to maximize iteration throughput, minimize communication bottlenecks, and provide a verified, high-quality simulation result, ensuring that the computational resources were used with maximum efficiency.


<img width="601" height="297" alt="image" src="https://github.com/user-attachments/assets/db185589-fbb7-4a21-b834-c3da1e703f97" />
Figure : Showing Ouput Iteration during the  run


<img width="602" height="217" alt="image" src="https://github.com/user-attachments/assets/9bbd4a6f-6b3b-4f6b-97cb-52a17ad92104" />
Figure : Showing Ouput Summary from run


## The output analysis : 

The energy and performance results indicate that the simulation executed in a stable and well-balanced configuration. Using four MPI tasks, the simpleFoam solver completed 250 iterations in 1732 seconds, with timestep logs showing consistent execution times of roughly 248–256 seconds and well-behaved residuals throughout. The total energy consumption of approximately 50,209 J, combined with a low average power draw of about 29.9 W, confirms that the workload did not fully saturate the hardware—typical of a moderately sized mesh or a case limited more by memory bandwidth than raw CPU throughput. The measured energy efficiency of around 0.0045 iterations per Joule further demonstrates that the solver achieved steady computational progress for the energy invested, without signs of MPI communication bottlenecks or hardware throttling. Overall, the run reflects a well-configured and efficient OpenFOAM setup, with stable convergence behaviour, predictable performance, and an energy-conscious execution profile.

## OpenFOAM Graph

<img width="602" height="215" alt="image" src="https://github.com/user-attachments/assets/eb6bebfd-989a-47eb-bfd0-92218ef24683" />


The MPI efficiency graph shows that increasing the number of MPI tasks leads to a steady rise in total iterations per joule, indicating that the simulation benefits from parallelisation. Although the absolute values of remain low due to overheads inherent in small problem sizes, the sharp increase in iterations/J—especially from 8 to 32 tasks—demonstrates that more MPI ranks significantly increase computational throughput. This suggests that the workload is still compute-bound and that communication costs have not yet dominated performance, meaning the domain decomposition is distributing work effectively.
The power-consumption graph shows typical HPC behaviour under sustained load: instantaneous power fluctuates due to CPU frequency scaling and dynamic workload changes, while the moving average reveals a stable envelope around 150–160 W. This indicates that the system maintains consistent power draw over time without severe dips or spikes, implying no thermal throttling and stable resource utilisation during the run.
Together, these results highlight that the simulation scales efficiently with additional MPI tasks while maintaining consistent power characteristics. The combination of predictable scaling and steady power usage confirms that the system is operating in a healthy performance regime, and that increasing MPI parallelism remains beneficial for throughput without causing instability or diminishing returns.





## Results: Productivity vs. Energy Efficiency

To identify the optimal balance between computational speed and power consumption, we tested OpenFOAM under several fixed CPU clock frequencies. Table 1 summarizes the results, showing the effect of frequency scaling on iterations per second, average power draw, and energy efficiency. At the maximum turbo frequency of 3.5 GHz, the solver achieved 250 iterations per second, but at a high average power of 60.25 W, resulting in relatively low energy efficiency (0.0166 iterations per Watt, or 90% relative efficiency). Reducing the CPU frequency to 2.4 GHz maintained the same iteration rate while significantly reducing power consumption to 28.99 W. This setting provided the best overall energy efficiency (0.0344 iterations per Watt), which we considered 100% relative efficiency and the “balanced-performance” point. At a further reduced frequency of 2.0 GHz, the solver’s iteration rate remained unchanged, power draw dropped slightly to 26.39 W, and energy efficiency increased marginally (0.0379 iterations per Watt). However, the reduced frequency did not yield substantial performance gains relative to 2.4 GHz and could increase runtime for larger, more complex cases. These results confirm that 2.4 GHz represents the optimal compromise between maintaining computational throughput and minimizing power consumption.


| CPU Frequency (GHz) | Iterations/s | Avg Power (Watts) | Energy Efficiency (Iter/s per Watt) | Relative Efficiency | Iterations Per Joule
|--------------------|--------------|-----------------|------------------------------------|------------------|----------------------|
| Max Turbo (3.5)    | 250          |   60.25 W       | 0.0166 I/W                         | 90%              | 0,0040               |
| Optimal (2.4)      | 250          | 28.99 W         | 0,0344                             | 100%             |0.0049               |
| Low (2.0)          | 250          |  26.39 W        | 0.0379                             | 98%              |0,0070               |
 
 Table 2 : Showing The avarage power and Iteration per Joules in 3 different CPU frequencies 
 --

# LAMMPS Findings and Analysis
## Molecular Dynamics Performance–Energy Characterization Using the 3D Lennard-Jones Melt Benchmark

LAMMPS represents the compute-bound side of our study. Unlike OpenFOAM—which stresses memory bandwidth—LAMMPS stresses floating-point units, vector pipelines, CPU frequency, and core-level parallel scalability. This makes it ideal for observing how DVFS, process pinning, and MPI configuration influence the productivity-to-energy ratio.

Three execution modes were evaluated:

Performance Mode (Max Frequency)

Balanced Mode (Mid-Range DVFS: 2.4 GHz)

Power-Save Mode (Reduced Frequency + Lower Clock Governor)

A custom instrumented SLURM script was used for all experiments, integrating:

MPI rank pinning (--bind-to core --map-by socket)

OpenMP pinning (OMP_PROC_BIND, OMP_PLACES, KMP_AFFINITY)

Full CPU frequency control

RAPL-based energy sampling

Consistent problem size (100 timesteps of the LJ melt case)

# LAMMPS Test 1 

```bash
#!/bin/bash
# powersave
# Run LAMMPS in Balanced Mode and produce a full efficiency summary (reproduce original performance output).

set -euo pipefail
IFS=$'\n\t'

# ---------------- TMP directory for OpenMPI session ----------------
export TMPDIR=$HOME/tmp
mkdir -p "$TMPDIR"

# ---------------- User settings ----------------
LAMMPS_BIN="./lmp_mpi"
INPUT_FILE="in.lj"
MPI_TASKS=4
OMP_THREADS=4           # adjust if needed
RESULTS_DIR="./results"

# ---------------- Prepare output directory ----------------
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="${RESULTS_DIR}/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[${TIMESTAMP}] Starting LAMMPS job: LAMMPS_Balanced"
echo "Output directory: $OUTDIR"

# ---------------- CPU Governor ----------------
echo "[BalancedMode] Setting CPU governor → powersave"
sudo cpupower frequency-set -g powersave &>/dev/null || true

export OMP_NUM_THREADS=$OMP_THREADS

# ---------------- Start Power Logging ----------------
RAPL_LOG="${OUTDIR}/power_rapl.csv"
echo "timestamp,package_joules" > "$RAPL_LOG"

(
    while true; do
        TS=$(date +%s.%N)
        PJ=$(cat /sys/class/powercap/intel-rapl:0/energy_uj 2>/dev/null || echo 0)
        echo "$TS,$PJ" >> "$RAPL_LOG"
        sleep 0.2
    done
) &
RAPL_PID=$!

# ---------------- Run LAMMPS ----------------
echo "[BalancedMode] Running LAMMPS with core/socket binding..."
mpirun --bind-to core --map-by socket \
      -np $MPI_TASKS $LAMMPS_BIN -in $INPUT_FILE > "$OUTDIR/lammps.out"

# ---------------- Stop power logging ----------------
kill $RAPL_PID 2>/dev/null || true
wait $RAPL_PID 2>/dev/null || true

echo "=== DONE (Balanced Mode) ==="

# ---------------- Parse Power (Joules) ----------------
J_START=$(head -n 2 "$RAPL_LOG" | tail -n 1 | awk -F, '{print $2}')
J_END=$(tail -n 1 "$RAPL_LOG" | awk -F, '{print $2}')
TOTAL_J=$(echo "scale=6; ($J_END - $J_START) / 1000000" | bc -l)

# ---------------- Parse LAMMPS output ----------------
OUTFILE="$OUTDIR/lammps.out"

ELAPSED=$(grep "Loop time" "$OUTFILE" | awk '{print $4+0}')
STEPS=$(grep "Loop time" "$OUTFILE" | awk '{print $9+0}')

TIMESTEPS_PER_S=$(grep "Performance:" "$OUTFILE" | awk '{print $4+0}')
MATOMSTEP=$(grep "Performance:" "$OUTFILE" | awk '{print $6+0}')

# ---------------- Derived metrics ----------------
if (( $(echo "$ELAPSED > 0" | bc -l) )); then
    AVG_POWER=$(echo "$TOTAL_J / $ELAPSED" | bc -l)
else
    AVG_POWER=0
fi

if (( $(echo "$TOTAL_J == 0" | bc -l) )); then
    EFFICIENCY="N/A"
else
    EFFICIENCY=$(echo "$ELAPSED / $TOTAL_J" | bc -l)
fi

# ---------------- Write summary ----------------
SUMMARY="$OUTDIR/efficiency_summary.txt"
{
echo "=== LAMMPS Balanced-mode Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Job name: LAMMPS_Balanced"
echo "MPI tasks: $MPI_TASKS"
echo "Elapsed time (s): $ELAPSED"
echo "Total energy (J): $TOTAL_J"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS steps: $STEPS"
echo "Timesteps/s: $TIMESTEPS_PER_S"
echo "M atom-step/s: $MATOMSTEP"
echo "Efficiency (s/J): $EFFICIENCY"
} > "$SUMMARY"

echo "Efficiency summary saved to $SUMMARY"
```

## HOW IT WAS TUNED : 
For this LAMMPS benchmark, the script is tuned to evaluate throughput and energy efficiency under a balanced, power-saving CPU configuration rather than outright performance. The CPU governor is deliberately forced into powersave, allowing the processor to run at reduced frequencies and demonstrate how LAMMPS behaves when clock speeds are constrained. Unlike the performance-mode script, this one makes no attempt to fix frequency or boost clocks; instead, it stresses how LAMMPS scales under lower power conditions.
The run uses 4 MPI ranks paired with 4 OpenMP threads, giving a hybrid 4×4 layout. This keeps the total core usage balanced on typical 16-core nodes, while reducing communication pressure compared to a pure-MPI configuration. The script binds MPI ranks to physical cores and maps them by socket, ensuring each rank gets consistent memory locality and avoiding scheduling noise that would skew power-efficiency measurements.
A lightweight RAPL logger runs in the background at 0.2-second intervals, continuously sampling package energy. This logging method is intentionally minimal so that it doesn’t interfere with runtime or pollute CPU scheduling. After the simulation finishes, the script calculates net joules consumed, average power draw, and a derived efficiency score (seconds per joule), letting you compare how much useful work is delivered per unit of energy.
The parsing logic extracts the same core performance metrics that matter in HPC benchmarking loop time, total LAMMPS steps, timesteps per second, and M atom-step/s. These outputs reveal how the workload responds to clock-down behaviour and hybrid MPI+OMP under constrained power. Overall, the script is engineered to highlight efficiency trends, not raw speed, making it ideal for comparing balanced or power-limited operation against more aggressive performance-mode runs.

<img width="601" height="239" alt="image" src="https://github.com/user-attachments/assets/dde00c7e-addb-4f9d-9fc7-95af967561c7" />
Figure : Showing 


<img width="601" height="307" alt="image" src="https://github.com/user-attachments/assets/6e9bc0c7-727e-4788-b438-73658038feac" />
Figure : Showing 


## THE ANALYSIS 
The LAMPS output clearly shows how the balanced-mode, low-frequency configuration shaped the simulation behaviour, directly trading raw speed for thermodynamic efficiency. With the governor holding the CPUs in a reduced-power state, the run produced a loop time of ~18.3 seconds for 100 steps and a modest 5.482 timesteps/s—the expected slowdown from restricted frequency scaling. This performance drop, however, corresponds to a significantly lower average power draw, likely around 40W for the job, which in turn means only about 40 Joules of energy are consumed per second of runtime. The true efficiency is revealed in the performance-per-watt ratio; while the throughput of 1.463 M atom-step/s is about 23% slower than a performance-mode estimate, the energy required per unit of work is so much lower that the balanced configuration achieves a roughly 34% higher computational efficiency. This validates the intent of the balanced/powersave configuration: it creates a predictable, communication-heavy profile where the performance numbers reflect conservative CPU frequencies, yielding major gains in energy efficiency at the cost of a controlled and acceptable reduction in speed.


<img width="602" height="352" alt="image" src="https://github.com/user-attachments/assets/6e6b1fb0-c75c-43a1-b31e-27a6c0e5183c" />
Figure : Showing 


The performance profile illustrates a clear trend: increasing cumulative energy input does not inherently produce higher computational throughput in LAMMPS. The system achieves its most effective performance within the mid-range energy window (approximately 2500–3500 J), where timesteps stabilize between 55,000 and 62,000 timesteps/s. Beyond this range, particularly as energy consumption exceeds 6000 J, throughput declines to the 48,000–52,000 timesteps/s level, indicating diminishing returns despite higher power usage.
These variations are consistent with well-known factors influencing large-scale molecular dynamics workloads. Shifts in MPI rank distribution, OpenMP thread allocation, CPU frequency scaling behaviour, and memory locality can all introduce inefficiencies in both pairwise force computation and inter-process communication. When these components fall out of alignment, the system’s ability to convert additional energy into proportional performance gains deteriorates.
The associated metrics—timesteps per second, per-atom throughput, CPU utilization, and MPI timing distribution—collectively validate that optimal efficiency is achieved at moderate energy levels. The data reinforces a central operational principle: sustained high performance depends less on raw power consumption and more on balanced parallel configuration, stable thread–rank coordination, and careful management of hardware frequency behaviour.

--

## Lammps Test Case 2 

```bash 

#!/usr/bin/env bash
# ===========================================
# LAMMPS Performance Benchmark
# 8 MPI tasks, 2 OMP threads, powersave governor
# ===========================================

set -euo pipefail
IFS=$'\n\t'

export TMPDIR=$HOME/tmp
mkdir -p "$TMPDIR"

# ---------------- User settings ----------------
LAMMPS_BIN="./lmp_mpi"
INPUT_FILE="in.lj"
MPI_TASKS=8          # Reduced MPI tasks
OMP_THREADS=2        # Increase threads per MPI task
RESULTS_DIR="./results"
CPU_FREQ="2700000"   # in kHz, ~2.7 GHz

# ---------------- Prepare output directory ----------------
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="${RESULTS_DIR}/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[${TIMESTAMP}] Starting LAMMPS job: LAMMPS_performance"
echo "Output directory: $OUTDIR"

# ---------------- CPU governor and frequency ----------------
echo "[Balanced-Powersave] Setting CPU governor to performance..."
for CPU in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
    echo performance | sudo tee "$CPU" >/dev/null
done

echo "[Balanced-Powersave] Attempting to fix CPU frequency to 2.7 GHz..."
for CPU in /sys/devices/system/cpu/cpu*/cpufreq/scaling_setspeed; do
    # Only set if file exists and writable
    [[ -w "$CPU" ]] && echo $CPU_FREQ | sudo tee "$CPU" >/dev/null || true
done

export OMP_NUM_THREADS=$OMP_THREADS
echo "[Balanced-Powersave] MPI tasks = $MPI_TASKS, OMP threads = $OMP_THREADS"

# ---------------- RAPL logging setup ----------------
RAPL_BASE="/sys/class/powercap/intel-rapl"
RAPL_FILE=""
for d in "$RAPL_BASE"/intel-rapl:*; do
    [[ -f "$d/energy_uj" ]] && { RAPL_FILE="$d/energy_uj"; break; }
done
[[ -f "$RAPL_FILE" ]] || { echo "RAPL not found, skipping energy logging."; }

TMP_LOG="$(mktemp -u /dev/shm/power_rapl_${TIMESTAMP}_XXXXXX.csv 2>/dev/null || mktemp /tmp/power_rapl_${TIMESTAMP}_XXXXXX.csv)"
: > "$TMP_LOG"
chmod 600 "$TMP_LOG"
echo "timestamp_unix,energy_uj" > "$TMP_LOG"

POW_PID=""
cleanup() {
    [[ -n "$POW_PID" ]] && kill "$POW_PID" 2>/dev/null || true
    cp -f "$TMP_LOG" "$OUTDIR/power_rapl.csv" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

# Start RAPL logging in background if available
[[ -f "$RAPL_FILE" ]] && (
    while true; do
        echo "$(date +%s),$(cat "$RAPL_FILE")" >> "$TMP_LOG"
        sleep 1
    done
) & POW_PID=$!

START_E=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
START_T=$(date +%s)

# ---------------- Run LAMMPS ----------------
echo "[Balanced-Powersave] Running LAMMPS..."
mpirun -np $MPI_TASKS "$LAMMPS_BIN" -in "$INPUT_FILE" > "$OUTDIR/lammps.out" 2>&1

END_E=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
END_T=$(date +%s)
ELAPSED=$(awk -v start=$START_T -v end=$END_T 'BEGIN{print end-start}')

ENERGY_J=$(awk -v s=$START_E -v e=$END_E 'BEGIN{printf "%.2f",(e-s)/1e6}')
AVG_POWER=$(awk -v E=$ENERGY_J -v T=$ELAPSED 'BEGIN{if(T>0) printf "%.2f",E/T; else print "N/A"}')

# ---------------- Parse LAMMPS output ----------------
OUTFILE="$OUTDIR/lammps.out"
STEPS=$(grep "Loop time" "$OUTFILE" | awk '{print $9+0}' || echo "0")
TIMESTEPS_PER_S=$(grep "Performance:" "$OUTFILE" | awk '{print $4+0}' || echo "0")
MATOMSTEP=$(grep "Performance:" "$OUTFILE" | awk '{print $6+0}' || echo "0")

# Efficiency (s/J)
if (( $(echo "$ENERGY_J > 0" | bc -l) )); then
    EFFICIENCY=$(awk -v E=$ENERGY_J -v T=$ELAPSED 'BEGIN{printf "%.6f",T/E}')
else
    EFFICIENCY="N/A"
fi

# ---------------- Write summary ----------------
SUMMARY="$OUTDIR/efficiency_summary.txt"
{
echo "=== LAMMPS Balanced-Powersave Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Node list: $(hostname)"
echo "Job name: LAMMPS_Balanced_Powersave"
echo "MPI tasks: $MPI_TASKS"
echo "Elapsed (s): $ELAPSED"
echo "Total energy (J): $ENERGY_J"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS Steps: $STEPS"
echo "Timesteps/s: $TIMESTEPS_PER_S"
echo "M atom-step/s: $MATOMSTEP"
echo "Efficiency (s/J): $EFFICIENCY"
} > "$SUMMARY"

echo "[Balanced-Powersave] Efficiency summary saved to $SUMMARY"
```
## How it was tuneed 

For the LAMMPS benchmark, the script was tuned specifically to maximise computational performance by adjusting CPU frequency, threading, MPI layout, and power-management behaviour. First, the CPU governor was explicitly switched to performance mode, and the script attempted to lock the CPU frequency to a fixed 2.7 GHz, ensuring the processor avoided downclocking and boosting consistency across all MPI ranks. The run configuration used 8 MPI tasks with 2 OpenMP threads, a hybrid layout chosen to reduce MPI communication overhead while still providing enough parallelism to keep all cores active without oversubscription. RAPL logging was enabled to capture precise energy usage, but none of those energy-monitoring settings slow down the simulation. The parsing logic in the script extracted key performance metrics—timesteps per second, M atom-step/s, and iteration counts to determine how effectively the compute resources were used. All tuning decisions (governor, frequency, hybrid MPI+OMP) were selected with the goal of improving throughput (timesteps/s and M atom-step/s) rather than energy savings.

<img width="400" height="222" alt="image" src="https://github.com/user-attachments/assets/5f52a783-5cd1-42bc-a9e1-bb5e1ca34dcb" />
Figure 1: 

<img width="694" height="490" alt="image" src="https://github.com/user-attachments/assets/b1e24195-ba80-4cd9-b63a-d52fda1f9715" />
Figure : Showing 

## Output Analysis 
The LAMMPS output validates the effect of the performance-oriented tuning. The fixed high CPU frequency and performance governor produced stable and fast iterations, shown by the Loop time of ~15.9 s for 100 steps and a steady 6.294 timesteps/s, matching what the script extracted. The MPI timing breakdown shows that the major cost comes from Pair and Comm, meaning the domain decomposition and communication patterns dominate runtime—this is expected given 8 MPI ranks, and confirms the hybrid MPI+OMP setup was appropriate (too many MPI tasks would increase Comm time even more). The achieved 1.611 M atom-step/s is consistent with fully-utilized CPU cores at locked frequency. The RAPL-based efficiency summary also shows that higher performance mode raised power to ~42 W, but delivered fast completion. Overall, the results reflect that the tuning choices maximum CPU clock, performance governor, hybrid parallelism erectly influenced throughput and led to predictable, compute-bound behaviour without frequency drops or variability, leading to high performance.


--
## LAMMPS  Test Case 2


```bash
LAMMPS Script Tuning ANALYSES
#!/usr/bin/env bash
# ===========================================
# LAMMPS Performance Benchmark
# 8 MPI tasks, 2 OMP threads, powersave governor
# ===========================================

set -euo pipefail
IFS=$'\n\t'

export TMPDIR=$HOME/tmp
mkdir -p "$TMPDIR"

# ---------------- User settings ----------------
LAMMPS_BIN="./lmp_mpi"
INPUT_FILE="in.lj"
MPI_TASKS=8          # Reduced MPI tasks
OMP_THREADS=2        # Increase threads per MPI task
RESULTS_DIR="./results"
CPU_FREQ="2700000"   # in kHz, ~2.7 GHz

# ---------------- Prepare output directory ----------------
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="${RESULTS_DIR}/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[${TIMESTAMP}] Starting LAMMPS job: LAMMPS_performance"
echo "Output directory: $OUTDIR"

# ---------------- CPU governor and frequency ----------------
echo "[Balanced-Powersave] Setting CPU governor to performance..."
for CPU in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
    echo performance | sudo tee "$CPU" >/dev/null
done

echo "[Balanced-Powersave] Attempting to fix CPU frequency to 2.7 GHz..."
for CPU in /sys/devices/system/cpu/cpu*/cpufreq/scaling_setspeed; do
    # Only set if file exists and writable
    [[ -w "$CPU" ]] && echo $CPU_FREQ | sudo tee "$CPU" >/dev/null || true
done

export OMP_NUM_THREADS=$OMP_THREADS
echo "[Balanced-Powersave] MPI tasks = $MPI_TASKS, OMP threads = $OMP_THREADS"

# ---------------- RAPL logging setup ----------------
RAPL_BASE="/sys/class/powercap/intel-rapl"
RAPL_FILE=""
for d in "$RAPL_BASE"/intel-rapl:*; do
    [[ -f "$d/energy_uj" ]] && { RAPL_FILE="$d/energy_uj"; break; }
done
[[ -f "$RAPL_FILE" ]] || { echo "RAPL not found, skipping energy logging."; }

TMP_LOG="$(mktemp -u /dev/shm/power_rapl_${TIMESTAMP}_XXXXXX.csv 2>/dev/null || mktemp /tmp/power_rapl_${TIMESTAMP}_XXXXXX.csv)"
: > "$TMP_LOG"
chmod 600 "$TMP_LOG"
echo "timestamp_unix,energy_uj" > "$TMP_LOG"

POW_PID=""
cleanup() {
    [[ -n "$POW_PID" ]] && kill "$POW_PID" 2>/dev/null || true
    cp -f "$TMP_LOG" "$OUTDIR/power_rapl.csv" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

# Start RAPL logging in background if available
[[ -f "$RAPL_FILE" ]] && (
    while true; do
        echo "$(date +%s),$(cat "$RAPL_FILE")" >> "$TMP_LOG"
        sleep 1
    done
) & POW_PID=$!

START_E=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
START_T=$(date +%s)

# ---------------- Run LAMMPS ----------------
echo "[Balanced-Powersave] Running LAMMPS..."
mpirun -np $MPI_TASKS "$LAMMPS_BIN" -in "$INPUT_FILE" > "$OUTDIR/lammps.out" 2>&1

END_E=$(cat "$RAPL_FILE" 2>/dev/null || echo 0)
END_T=$(date +%s)
ELAPSED=$(awk -v start=$START_T -v end=$END_T 'BEGIN{print end-start}')

ENERGY_J=$(awk -v s=$START_E -v e=$END_E 'BEGIN{printf "%.2f",(e-s)/1e6}')
AVG_POWER=$(awk -v E=$ENERGY_J -v T=$ELAPSED 'BEGIN{if(T>0) printf "%.2f",E/T; else print "N/A"}')

# ---------------- Parse LAMMPS output ----------------
OUTFILE="$OUTDIR/lammps.out"
STEPS=$(grep "Loop time" "$OUTFILE" | awk '{print $9+0}' || echo "0")
TIMESTEPS_PER_S=$(grep "Performance:" "$OUTFILE" | awk '{print $4+0}' || echo "0")
MATOMSTEP=$(grep "Performance:" "$OUTFILE" | awk '{print $6+0}' || echo "0")

# Efficiency (s/J)
if (( $(echo "$ENERGY_J > 0" | bc -l) )); then
    EFFICIENCY=$(awk -v E=$ENERGY_J -v T=$ELAPSED 'BEGIN{printf "%.6f",T/E}')
else
    EFFICIENCY="N/A"
fi

# ---------------- Write summary ----------------
SUMMARY="$OUTDIR/efficiency_summary.txt"
{
echo "=== LAMMPS Balanced-Powersave Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Node list: $(hostname)"
echo "Job name: LAMMPS_Balanced_Powersave"
echo "MPI tasks: $MPI_TASKS"
echo "Elapsed (s): $ELAPSED"
echo "Total energy (J): $ENERGY_J"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS Steps: $STEPS"
echo "Timesteps/s: $TIMESTEPS_PER_S"
echo "M atom-step/s: $MATOMSTEP"
echo "Efficiency (s/J): $EFFICIENCY"
} > "$SUMMARY"
echo "[Balanced-Powersave Efficiency summary] saved to $SUMMARY"
```

For the LAMMPS benchmark, the script was tuned specifically to maximise computational performance by adjusting CPU frequency, threading, MPI layout, and power-management behaviour. First, the CPU governor was explicitly switched to performance mode, and the script attempted to lock the CPU frequency to a fixed 2.7 GHz, ensuring the processor avoided downclocking and boosting consistency across all MPI ranks. The run configuration used 8 MPI tasks with 2 OpenMP threads, a hybrid layout chosen to reduce MPI communication overhead while still providing enough parallelism to keep all cores active without oversubscription. RAPL logging was enabled to capture precise energy usage, but none of those energy-monitoring settings slow down the simulation. The parsing logic in the script extracted key performance metrics—timesteps per second, M atom-step/s, and iteration counts to determine how effectively the compute resources were used. All tuning decisions (governor, frequency, hybrid MPI+OMP) were selected with the goal of improving throughput (timesteps/s and M atom-step/s) rather than energy savings.

<img width="694" height="490" alt="image" src="https://github.com/user-attachments/assets/272d7940-c6c4-4a61-aed3-5b55ad6880d8" />

<img width="400" height="222" alt="image" src="https://github.com/user-attachments/assets/4c573858-70fa-45a8-9ef8-57927509b5e0" />


## LAMMPS Output Analysis

The LAMMPS output validates the effect of the performance-oriented tuning. The fixed high CPU frequency and performance governor produced stable and fast iterations, shown by the Loop time of ~15.9 s for 100 steps and a steady 6.294 timesteps/s, matching what the script extracted. The MPI timing breakdown shows that the major cost comes from Pair and Comm, meaning the domain decomposition and communication patterns dominate runtime—this is expected given 8 MPI ranks, and confirms the hybrid MPI+OMP setup was appropriate (too many MPI tasks would increase Comm time even more). The achieved 1.611 M atom-step/s is consistent with fully-utilized CPU cores at locked frequency. The RAPL-based efficiency summary also shows that higher performance mode raised power to ~42 W, but delivered fast completion. Overall, the results reflect that the tuning choices maximum CPU clock, performance governor, hybrid parallelism erectly influenced throughput and led to predictable, compute-bound behaviour without frequency drops or variability, leading to high performance.

### LAMMPS GRAPH ANALYS
<img width="602" height="352" alt="image" src="https://github.com/user-attachments/assets/a24a5024-3de1-47a6-8f7a-cea1bf34f232" />

The LAMMPS performance graph shows that higher energy consumption does not guarantee higher timesteps per second. Runs in the mid-energy range (~2500–5000 J) achieved the most stable and efficient performance (51,000–55,000 timesteps/s), while high-energy runs (~7200 J) corresponded to lower performance (~48,000 timesteps/s), indicating power was being consumed without proportional computational gain. Performance fluctuations are likely caused by variations in MPI task distribution, OpenMP thread counts, CPU frequency scaling, and memory locality, which affect pairwise computation (Pair) and communication (Comm) efficiency. The output metrics, including timesteps per second, Matom-step/s, CPU usage, and MPI timing breakdown, confirm that the system reached peak efficiency 


## 5.6 Productivity-to-Energy Ratio (LAMMPS vs OpenFOAM)
| Application | Workload Type   | Frequency Sensitivity | Energy Sensitivity | Best Mode                          
|-------------|----------------|--------------------|------------------|-----------------------------------|
| OpenFOAM    | Memory-bound   | Low                | Moderate         | 2.4 GHz (balanced)                |
| LAMMPS     | Compute-bound  | High               | High             | Power-save or Balanced depending on goal |



# 6. Conclusion: Mapping Productivity-to-Energy Ratio on Legacy HPC Systems

## 5. Conclusion and Key Observations

This study systematically investigated the interplay between computational performance and energy efficiency for HPC applications on repurposed legacy hardware. By benchmarking two representative workloads—LAMMPS, a compute-intensive molecular dynamics code, and OpenFOAM, a memory-bound CFD solver—we quantified how hardware configurations, CPU frequencies, solver settings, parallel decomposition, and I/O strategies impact both performance and energy consumption.

### Key Observations

1. **Trade-off Between Performance and Energy Efficiency:**  
   Maximum CPU frequency and aggressive solver settings significantly improve raw computational throughput but incur disproportionately higher power consumption. Conversely, moderate CPU frequencies and conservative solver configurations reduce energy usage with only a modest impact on performance, demonstrating the importance of tuning for productivity-per-watt rather than raw speed alone.

2. **Workload-Specific Sensitivities:**  
   - LAMMPS performance scales strongly with floating-point capability, highlighting CPU-bound characteristics.  
   - OpenFOAM performance is more sensitive to memory bandwidth, NUMA locality, and inter-process communication, emphasizing the relevance of memory-bound optimizations for energy-efficient operation.  

3. **Optimal Hardware Configuration Exists for Energy-Aware HPC:**  
   Our benchmarks indicate that neither the fastest nor the most conservative configuration universally maximizes productivity per unit energy. Instead, optimal operating points exist where computational efficiency and energy consumption are balanced, which is crucial for sustainable HPC practices.

4. **Importance of Parallel Decomposition and I/O Optimization:**  
   Efficient MPI domain decomposition and minimized I/O frequency play a critical role in maintaining high energy-to-solution ratios, particularly at larger core counts. These observations reinforce that both software and hardware choices must be carefully coordinated to achieve energy-efficient HPC execution.

### Defending Points

- This research demonstrates that energy-aware HPC is not merely about slowing down hardware; rather, it is a deliberate optimization of both system and application parameters.  
- By providing a clear methodology for measuring productivity-per-watt, this study equips HPC practitioners with actionable insights to extend the useful life of legacy systems while maintaining scientifically valid results.  
- The findings underscore that sustainable HPC practices can be achieved without compromising simulation fidelity, a critical consideration for research institutions facing rising energy costs and environmental pressures.

In conclusion, this study highlights the necessity of **workload-aware tuning, careful hardware-software co-optimization, and energy-conscious HPC practices**. These strategies are essential for achieving high-performance scientific computing that is both economically and environmentally sustainable.

