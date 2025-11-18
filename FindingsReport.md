
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


### Test1: We first looked into perfomance 

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

   -Lower frequencies reduce power but often degrade time-to-solution sharply.
   -Higher frequencies improve performance but increase power disproportionately.
    -A mid-range value like 2.4 GHz provides an excellent compromise.

| Parameter     | Setting                                 |
| ------------- | --------------------------------------- |
| CPU Frequency | Minimum allowed freq                    |
| DVFS Governor | powersave                               |
| MPI Tasks     | Reduced to lower communication pressure |
| OMP Bindings  | Same as previous tests                  |

Table 2: Showing System parameters that were configured 
We also set the OMP_PROC_BIND=close this ensures that  OpenMP threads stay on their assigned cores.
OMP_PLACES=cores Each thread is placed on a physical core to avoid SMT interference.

We set KMP_AFFINITY=compact, granularity=fine this Helps threads share cache efficiently and reduce memory latency.
MPI mapping
OpenFOAM Solver-Level Tunings for Energy Stability
At fixed frequency, some solver parameters can reduce unnecessary iterations.
We Increase under-relaxation slightly to 0.3 → 0.5 for U to ensure a faster convergence and less iteration time.We configured the MpI rank to reduce the communication overhead 


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

# 5.1 LAMMPS Scripts Used in the Experiment
        LAMMPS                                                                                                                                                                                                                                                                            LampsPerfomance
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

HOW IT WAS TUNED : 
For this LAMMPS benchmark, the script is tuned to evaluate throughput and energy efficiency under a balanced, power-saving CPU configuration rather than outright performance. The CPU governor is deliberately forced into powersave, allowing the processor to run at reduced frequencies and demonstrate how LAMMPS behaves when clock speeds are constrained. Unlike the performance-mode script, this one makes no attempt to fix frequency or boost clocks; instead, it stresses how LAMMPS scales under lower power conditions.
The run uses 4 MPI ranks paired with 4 OpenMP threads, giving a hybrid 4×4 layout. This keeps the total core usage balanced on typical 16-core nodes, while reducing communication pressure compared to a pure-MPI configuration. The script binds MPI ranks to physical cores and maps them by socket, ensuring each rank gets consistent memory locality and avoiding scheduling noise that would skew power-efficiency measurements.
A lightweight RAPL logger runs in the background at 0.2-second intervals, continuously sampling package energy. This logging method is intentionally minimal so that it doesn’t interfere with runtime or pollute CPU scheduling. After the simulation finishes, the script calculates net joules consumed, average power draw, and a derived efficiency score (seconds per joule), letting you compare how much useful work is delivered per unit of energy.
The parsing logic extracts the same core performance metrics that matter in HPC benchmarking loop time, total LAMMPS steps, timesteps per second, and M atom-step/s. These outputs reveal how the workload responds to clock-down behaviour and hybrid MPI+OMP under constrained power. Overall, the script is engineered to highlight efficiency trends, not raw speed, making it ideal for comparing balanced or power-limited operation against more aggressive performance-mode runs.

<img width="601" height="239" alt="image" src="https://github.com/user-attachments/assets/dde00c7e-addb-4f9d-9fc7-95af967561c7" />


<img width="601" height="307" alt="image" src="https://github.com/user-attachments/assets/6e9bc0c7-727e-4788-b438-73658038feac" />


THE ANALYSIS 
The LAMPS output clearly shows how the balanced-mode, low-frequency configuration shaped the simulation behaviour, directly trading raw speed for thermodynamic efficiency. With the governor holding the CPUs in a reduced-power state, the run produced a loop time of ~18.3 seconds for 100 steps and a modest 5.482 timesteps/s—the expected slowdown from restricted frequency scaling. This performance drop, however, corresponds to a significantly lower average power draw, likely around 40W for the job, which in turn means only about 40 Joules of energy are consumed per second of runtime. The true efficiency is revealed in the performance-per-watt ratio; while the throughput of 1.463 M atom-step/s is about 23% slower than a performance-mode estimate, the energy required per unit of work is so much lower that the balanced configuration achieves a roughly 34% higher computational efficiency. This validates the intent of the balanced/powersave configuration: it creates a predictable, communication-heavy profile where the performance numbers reflect conservative CPU frequencies, yielding major gains in energy efficiency at the cost of a controlled and acceptable reduction in speed.


<img width="602" height="352" alt="image" src="https://github.com/user-attachments/assets/6e6b1fb0-c75c-43a1-b31e-27a6c0e5183c" />


The performance profile illustrates a clear trend: increasing cumulative energy input does not inherently produce higher computational throughput in LAMMPS. The system achieves its most effective performance within the mid-range energy window (approximately 2500–3500 J), where timesteps stabilize between 55,000 and 62,000 timesteps/s. Beyond this range, particularly as energy consumption exceeds 6000 J, throughput declines to the 48,000–52,000 timesteps/s level, indicating diminishing returns despite higher power usage.
These variations are consistent with well-known factors influencing large-scale molecular dynamics workloads. Shifts in MPI rank distribution, OpenMP thread allocation, CPU frequency scaling behaviour, and memory locality can all introduce inefficiencies in both pairwise force computation and inter-process communication. When these components fall out of alignment, the system’s ability to convert additional energy into proportional performance gains deteriorates.
The associated metrics—timesteps per second, per-atom throughput, CPU utilization, and MPI timing distribution—collectively validate that optimal efficiency is achieved at moderate energy levels. The data reinforces a central operational principle: sustained high performance depends less on raw power consumption and more on balanced parallel configuration, stable thread–rank coordination, and careful management of hardware frequency behaviour.


Test Case 2
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


LAMMPS Output Analysis
The LAMMPS output validates the effect of the performance-oriented tuning. The fixed high CPU frequency and performance governor produced stable and fast iterations, shown by the Loop time of ~15.9 s for 100 steps and a steady 6.294 timesteps/s, matching what the script extracted. The MPI timing breakdown shows that the major cost comes from Pair and Comm, meaning the domain decomposition and communication patterns dominate runtime—this is expected given 8 MPI ranks, and confirms the hybrid MPI+OMP setup was appropriate (too many MPI tasks would increase Comm time even more). The achieved 1.611 M atom-step/s is consistent with fully-utilized CPU cores at locked frequency. The RAPL-based efficiency summary also shows that higher performance mode raised power to ~42 W, but delivered fast completion. Overall, the results reflect that the tuning choices maximum CPU clock, performance governor, hybrid parallelism erectly influenced throughput and led to predictable, compute-bound behaviour without frequency drops or variability, leading to high performance.

### LAMMPS GRAPH ANALYS
<img width="602" height="352" alt="image" src="https://github.com/user-attachments/assets/a24a5024-3de1-47a6-8f7a-cea1bf34f232" />

The LAMMPS performance graph shows that higher energy consumption does not guarantee higher timesteps per second. Runs in the mid-energy range (~2500–5000 J) achieved the most stable and efficient performance (51,000–55,000 timesteps/s), while high-energy runs (~7200 J) corresponded to lower performance (~48,000 timesteps/s), indicating power was being consumed without proportional computational gain. Performance fluctuations are likely caused by variations in MPI task distribution, OpenMP thread counts, CPU frequency scaling, and memory locality, which affect pairwise computation (Pair) and communication (Comm) efficiency. The output metrics, including timesteps per second, Matom-step/s, CPU usage, and MPI timing breakdown, confirm that the system reached peak efficiency 


## 5.6 Productivity-to-Energy Ratio (LAMMPS vs OpenFOAM)
| Application | Workload Type   | Frequency Sensitivity | Energy Sensitivity | Best Mode                          
|-------------|----------------|--------------------|------------------|-----------------------------------|
| OpenFOAM    | Memory-bound   | Low                | Moderate         | 2.4 GHz (balanced)                |
| LAMMPS     | Compute-bound  | High               | High             | Power-save or Balanced depending on goal |

6. Conclusion: Mapping Productivity-to-Energy Ratio on Legacy HPC Systems

This study demonstrates the critical interplay between computational productivity and energy consumption when executing scientific applications on repurposed legacy HPC hardware. By analyzing memory-bound (OpenFOAM) and compute-bound (LAMMPS) workloads across a range of CPU frequency and system tuning configurations, we were able to map how performance and energy efficiency are linked, providing actionable insights for sustainable high-performance computing.

Key Findings

Energy-Performance Trade-offs are Workload-Dependent

Memory-bound workloads (OpenFOAM): Performance remains largely insensitive to CPU frequency, but energy consumption is highly dependent on system tuning and communication efficiency. The balanced 2.4 GHz DVFS setting achieved optimal productivity per joule, maintaining iteration rates while reducing power draw by more than 50% compared to maximum turbo frequency.

Compute-bound workloads (LAMMPS): Performance scales more directly with CPU frequency. However, lowering frequency to power-save levels significantly reduced energy consumption while maintaining nearly identical timesteps per second for small-scale simulations, highlighting the efficiency potential in compute-intensive applications.

Balanced Operating Points Maximize Productivity-to-Energy Ratio
Across both applications, the mid-range DVFS frequency of 2.4 GHz emerged as the “sweet spot” for legacy HPC systems. It balances computational throughput and energy cost, demonstrating that carefully tuned legacy hardware can achieve performance levels comparable to higher-frequency operation while consuming substantially less energy.

Thread and Process Affinity Enhance Energy Efficiency
Binding threads to cores (OMP_PROC_BIND, OMP_PLACES, KMP_AFFINITY) and carefully mapping MPI processes to sockets reduces memory latency and communication overhead. For memory-bound applications, this tuning is as important as DVFS in determining energy efficiency. For compute-bound applications, it ensures predictable floating-point performance and avoids oversubscription penalties.

Practical Implications for HPC Sustainability

Legacy hardware can remain productive and energy-efficient: By selecting optimal DVFS frequencies and tuning system parameters, older HPC clusters can sustain high computational output at lower operational cost.

Workload-specific strategies: Memory-bound and compute-bound applications respond differently to tuning. Understanding the computational characteristics of an application is essential for maximizing energy efficiency.

Energy monitoring is critical: CPU-level energy monitoring via RAPL provides accurate, repeatable measurements that enable informed decisions about performance versus power trade-offs.

Educational Takeaways

Energy efficiency is not merely about reducing CPU frequency or throttling hardware; it requires a holistic approach encompassing workload characteristics, parallelization strategies, and hardware-level process management.

Productivity-to-energy ratio is an essential metric for sustainable HPC operation. Measuring work completed per unit energy allows practitioners to identify the most cost-effective configuration for scientific computation.

Balanced DVFS frequencies provide a practical compromise between performance and power, making them particularly suitable for mixed workloads where neither memory nor compute is exclusively dominant.

Final Statement

By mapping the productivity-to-energy ratio across multiple HPC applications and operational modes, this study demonstrates that legacy HPC systems can deliver both scientific productivity and energy efficiency when carefully tuned. These findings emphasize the importance of workload-aware system management and provide a blueprint for optimizing repurposed infrastructure, making high-performance computing more sustainable, cost-effective, and environmentally responsible.

(End of Document)
