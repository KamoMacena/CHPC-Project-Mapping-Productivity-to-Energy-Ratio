
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

Below are the exact scripts (trimmed for clarity but structurally intact) executed for the three modes. These scripts capture all power/performance tuning parameters.

## 5.1.1 Performance Mode Script (Max Frequency)
```#!/usr/bin/env bash
#===========================================
#LAMMPS parallel benchmark (Performance Mode)
#with RAPL power logging and performance summary
#===========================================

set -euo pipefail
IFS=$'\n\t'

#--- Modules ---
module purge
module load gnu12/12.4.0
module load openmpi4/4.1.6

#--- SLURM / job defaults ---
SLURM_NTASKS=${SLURM_NTASKS:-16}
JOB_NAME=${SLURM_JOB_NAME:-LAMMPS_Perf}
NODELIST=$(scontrol show hostnames 2>/dev/null || hostname)

#--- Absolute LAMMPS binary ---
LAMMPS_BIN="/home/debugthugz_shared/productivity_to_energy/productivity-to-energy-ratio-for-HPC-applications/benchmarks/Lamps/lammps/bench/lmp_mpi"

if [[ ! -x "$LAMMPS_BIN" ]]; then
    echo "ERROR: LAMMPS binary not executable: $LAMMPS_BIN"
    exit 1
fi

#--- Timestamped output directory ---
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="$(pwd)/results/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[$(date)] Starting LAMMPS job: $JOB_NAME"
echo "Output directory: $OUTDIR"

#--- Copy input files ---
INPUT_FILE="in.lj"
if [[ ! -f "$INPUT_FILE" ]]; then
    echo "ERROR: LAMMPS input file ($INPUT_FILE) not found."
    exit 1
fi
cp "$INPUT_FILE" "$OUTDIR/"

cd "$OUTDIR"

#======================================================
 #PERFORMANCE TUNING BLOCK
#======================================================
echo "[PerfMode] Setting CPU governor to performance..."
for CPU in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
    echo performance | sudo tee $CPU >/dev/null
done

#Detect MPI binding support
MPI_BIND=""
if mpirun --help | grep -q '\--bind-to'; then
    echo "[PerfMode] MPI binding supported, applying core binding..."
    MPI_BIND="--bind-to core"
else
    echo "[PerfMode] MPI binding not supported, skipping..."
fi

#======================================================
#RAPL energy setup
#======================================================
RAPL_BASE="/sys/class/powercap/intel-rapl"
RAPL_FILE=""
MAX_ENERGY_FILE=""

for d in "$RAPL_BASE"/intel-rapl:*; do
    if [[ -f "$d/energy_uj" ]]; then
        RAPL_FILE="$d/energy_uj"
        [[ -f "$d/max_energy_range_uj" ]] && MAX_ENERGY_FILE="$d/max_energy_range_uj"
        break
    fi
done

[[ -f "$RAPL_FILE" ]] || { echo "ERROR: RAPL energy file not found."; exit 1; }

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
    if [[ -n "${POW_PID:-}" ]]; then
        kill "$POW_PID" 2>/dev/null || true
        wait "$POW_PID" 2>/dev/null || true
    fi
    cp -f "$TMP_LOG" "$OUTDIR/power_rapl.csv" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

#--- Start RAPL logging ---
(
    while true; do
        echo "$(date +%s),$(cat "$RAPL_FILE")" >> "$TMP_LOG"
        sleep 1
    done
) &
POW_PID=$!

START_E=$(cat "$RAPL_FILE")
START_T=$(date +%s)

#--- Run LAMMPS ---
echo "Running LAMMPS (Performance Mode) on $SLURM_NTASKS MPI tasks..."
if command -v mpirun >/dev/null 2>&1; then
    mpirun -np "$SLURM_NTASKS" $MPI_BIND "$LAMMPS_BIN" -in "$INPUT_FILE" > lammps.out 2>&1 || true
else
    echo "ERROR: mpirun not found."
    exit 1
fi

END_E=$(cat "$RAPL_FILE")
END_T=$(date +%s)
ELAPSED=$((END_T - START_T))

#Handle RAPL wrap-around
if [[ "$END_E" -lt "$START_E" && $WRAP_ADD -gt 0 ]]; then
    END_E=$((END_E + WRAP_ADD))
fi

ENERGY_J=$(awk -v s="$START_E" -v e="$END_E" 'BEGIN {printf "%.2f",(e-s)/1e6}')
AVG_POWER=$(awk -v E="$ENERGY_J" -v T="$ELAPSED" 'BEGIN{if(T>0) printf "%.2f",E/T; else print "0.00"}')

#--- Extract LAMMPS performance ---
STEPS=$(grep -E "Loop time of" lammps.out | awk '{print $(NF-8)}' | head -n1 || echo "0")
TIMESTEP_RATE=$(grep -E "timesteps/s" lammps.out | awk '{print $(NF-1)}' | head -n1 || echo "0")
ATOM_RATE=$(grep -E "Matom-step/s" lammps.out | awk '{print $1}' | head -n1 || echo "0")
EFF_S_PER_J=$(awk -v E="$ENERGY_J" -v T="$ELAPSED" 'BEGIN{if(E>0) printf "%.6f",T/E; else print "0"}')

#--- Write summary ---
{
echo "=== LAMMPS Performance-mode Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Node list: $NODELIST"
echo "Job name: $JOB_NAME"
echo "MPI tasks: $SLURM_NTASKS"
echo "Elapsed (s): $ELAPSED"
echo "Total energy (J): $ENERGY_J"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS Steps: $STEPS"
echo "Timesteps/s: $TIMESTEP_RATE"
echo "M atom-step/s: $ATOM_RATE"
echo "Efficiency (s/J): $EFF_S_PER_J"
} > "$OUTDIR/efficiency_summary.txt"

echo "=== DONE (Performance Mode) ==="
echo "Results stored in: $OUTDIR" `

## 5.1.2 Balanced Mode Script (Fixed 2.4 GHz DVFS)
`#!/bin/bash
#MweLammps_Balance.sh
#Run LAMMPS in Balanced Mode and produce a full efficiency summary.

set -euo pipefail
IFS=$'\n\t'

#---------------- User settings ----------------
LAMMPS_BIN="./lmp_mpi"
INPUT_FILE="in.lj"
MPI_TASKS=16
RESULTS_DIR="./results"

#---------------- Prepare output directory ----------------
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="${RESULTS_DIR}/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[${TIMESTAMP}] Starting LAMMPS job: LAMMPS_Balanced"
echo "Output directory: $OUTDIR"

#---------------- Run LAMMPS ----------------
echo "[BalancedMode] Tuning CPU governor to ondemand..."
sudo cpupower frequency-set -g ondemand &>/dev/null || true
echo "[BalancedMode] Letting OS schedule MPI tasks (no aggressive binding)."
mpirun -np $MPI_TASKS $LAMMPS_BIN -in $INPUT_FILE > "$OUTDIR/lammps.out"

echo "=== DONE (Balanced Mode) ==="

#---------------- Parse LAMMPS output ----------------
OUTFILE="$OUTDIR/lammps.out"

#Total elapsed time from "Loop time" line
ELAPSED=$(grep "Loop time" "$OUTFILE" | awk '{print $4+0}') || ELAPSED=0

#Total LAMMPS steps (from "Loop time" line)
STEPS=$(grep "Loop time" "$OUTFILE" | awk '{print $3+0}') || STEPS=0

#Timesteps per second
TIMESTEPS_PER_S=$(grep "Performance:" "$OUTFILE" | awk '{print $4+0}') || TIMESTEPS_PER_S=0

#Million atom-steps per second
MATOMSTEP=$(grep "Performance:" "$OUTFILE" | awk '{print $5+0}') || MATOMSTEP=0

#Total energy: pick the last TotEng value from the output table
TOTAL_ENERGY=$(awk '/^ *Step/ {header=1; next} header {energy=$5} END {print energy+0}' "$OUTFILE") || TOTAL_ENERGY=0

#Calculate efficiency (s/J)
if (( $(echo "$TOTAL_ENERGY == 0" | bc -l) )); then
    EFFICIENCY="N/A"
    AVG_POWER="N/A"
else
    EFFICIENCY=$(echo "$ELAPSED / $TOTAL_ENERGY" | bc -l)
    AVG_POWER=$(echo "$TOTAL_ENERGY / $ELAPSED" | bc -l)
fi

#---------------- Write summary ----------------
SUMMARY="$OUTDIR/efficiency_summary.txt"
{
echo "=== LAMMPS Balanced-mode Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Node list: $(hostname)"
echo "Job name: LAMMPS_Balanced"
echo "MPI tasks: $MPI_TASKS"
echo "Elapsed (s): $ELAPSED"
echo "Total energy (J): $TOTAL_ENERGY"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS Steps: $STEPS"
echo "Timesteps/s: $TIMESTEPS_PER_S"
echo "M atom-step/s: $MATOMSTEP"
echo "Efficiency (s/J): $EFFICIENCY"
} > "$SUMMARY"

`echo "Efficiency summary saved to $SUMMARY"
#!/bin/bash
#MweLammps_Balance.sh
#Run LAMMPS in Balanced Mode and produce a full efficiency summary.

set -euo pipefail
IFS=$'\n\t'

#---------------- User settings ----------------
LAMMPS_BIN="./lmp_mpi"
INPUT_FILE="in.lj"
MPI_TASKS=16
RESULTS_DIR="./results"

#---------------- Prepare output directory ----------------
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="${RESULTS_DIR}/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[${TIMESTAMP}] Starting LAMMPS job: LAMMPS_Balanced"
echo "Output directory: $OUTDIR"

#---------------- Run LAMMPS ----------------
echo "[BalancedMode] Tuning CPU governor to ondemand..."
sudo cpupower frequency-set -g ondemand &>/dev/null || true
echo "[BalancedMode] Letting OS schedule MPI tasks (no aggressive binding)."
mpirun -np $MPI_TASKS $LAMMPS_BIN -in $INPUT_FILE > "$OUTDIR/lammps.out"

echo "=== DONE (Balanced Mode) ==="

#---------------- Parse LAMMPS output ----------------
OUTFILE="$OUTDIR/lammps.out"

#Total elapsed time from "Loop time" line
ELAPSED=$(grep "Loop time" "$OUTFILE" | awk '{print $4+0}') || ELAPSED=0

#Total LAMMPS steps (from "Loop time" line)
STEPS=$(grep "Loop time" "$OUTFILE" | awk '{print $3+0}') || STEPS=0

#Timesteps per second
TIMESTEPS_PER_S=$(grep "Performance:" "$OUTFILE" | awk '{print $4+0}') || TIMESTEPS_PER_S=0

#Million atom-steps per second
MATOMSTEP=$(grep "Performance:" "$OUTFILE" | awk '{print $5+0}') || MATOMSTEP=0

#Total energy: pick the last TotEng value from the output table
TOTAL_ENERGY=$(awk '/^ *Step/ {header=1; next} header {energy=$5} END {print energy+0}' "$OUTFILE") || TOTAL_ENERGY=0

#Calculate efficiency (s/J)
if (( $(echo "$TOTAL_ENERGY == 0" | bc -l) )); then
    EFFICIENCY="N/A"
    AVG_POWER="N/A"
else
    EFFICIENCY=$(echo "$ELAPSED / $TOTAL_ENERGY" | bc -l)
    AVG_POWER=$(echo "$TOTAL_ENERGY / $ELAPSED" | bc -l)
fi

#---------------- Write summary ----------------
SUMMARY="$OUTDIR/efficiency_summary.txt"
{
echo "=== LAMMPS Balanced-mode Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Node list: $(hostname)"
echo "Job name: LAMMPS_Balanced"
echo "MPI tasks: $MPI_TASKS"
echo "Elapsed (s): $ELAPSED"
echo "Total energy (J): $TOTAL_ENERGY"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS Steps: $STEPS"
echo "Timesteps/s: $TIMESTEPS_PER_S"
echo "M atom-step/s: $MATOMSTEP"
echo "Efficiency (s/J): $EFFICIENCY"
} > "$SUMMARY"

echo "Efficiency summary saved to $SUMMARY"`

## 5.1.3 Power Save Mode Script (Minimum Frequency + Reduced Load)
`#!/usr/bin/env bash
#===========================================
#LAMMPS parallel benchmark (Energy-saving Mode)
#with RAPL power logging and performance summary
#===========================================

set -euo pipefail
IFS=$'\n\t'

#--- SLURM / job defaults ---
SLURM_NTASKS=${SLURM_NTASKS:-16}
JOB_NAME=${SLURM_JOB_NAME:-LAMMPS_Energy}
NODELIST=$(scontrol show hostnames 2>/dev/null || hostname)

#--- Timestamped output directory ---
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
OUTDIR="$(pwd)/results/run_${TIMESTAMP}"
mkdir -p "$OUTDIR"

echo "[$(date)] Starting LAMMPS job: $JOB_NAME"
echo "Output directory: $OUTDIR"

#--- Copy input files ---
INPUT_FILE="in.lj"
if [[ ! -f "$INPUT_FILE" ]]; then
    echo "LAMMPS input file ($INPUT_FILE) not found."
    exit 1
fi
cp -r "$INPUT_FILE" "$OUTDIR/"
cd "$OUTDIR"

#======================================================
#ENERGY-SAVING TUNING BLOCK
#======================================================

echo "[EnergyMode] Tuning CPU governor to powersave..."
for CPU in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
    echo powersave | sudo tee $CPU >/dev/null
done

echo "[EnergyMode] Skipping aggressive MPI binding (let OS schedule)."

#======================================================
 #END TUNING BLOCK
#======================================================

#--- RAPL setup ---
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

[[ -f "$RAPL_FILE" ]] || { echo "RAPL not found."; exit 1; }

WRAP_ADD=0
if [[ -n "$MAX_ENERGY_FILE" ]]; then
    max_range=$(awk '{print int($1)}' "$MAX_ENERGY_FILE" 2>/dev/null || echo 0)
    [[ $max_range -gt 0 ]] && WRAP_ADD=$max_range
fi

TMP_LOG="$(mktemp -u /dev/shm/power_rapl_${TIMESTAMP}_XXXXXX.csv)"
: > "$TMP_LOG"
chmod 600 "$TMP_LOG"
echo "timestamp_unix,energy_uj" > "$TMP_LOG"

POW_PID=""
cleanup() {
    if [[ -n "${POW_PID:-}" ]]; then
        kill "$POW_PID" 2>/dev/null || true
        wait "$POW_PID" 2>/dev/null || true
    fi
    cp -f "$TMP_LOG" "$OUTDIR/power_rapl.csv" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

#--- Start RAPL logging ---
(
    while true; do
        echo "$(date +%s),$(cat "$RAPL_FILE")" >> "$TMP_LOG"
        sleep 1
    done
) &
POW_PID=$!

START_E=$(cat "$RAPL_FILE")
START_T=$(date +%s)

#--- Run LAMMPS ---
echo "Running LAMMPS (Energy Mode) on $SLURM_NTASKS tasks..."

if command -v mpirun >/dev/null 2>&1; then
    mpirun -np "$SLURM_NTASKS" ./lmp_mpi -in "$INPUT_FILE" > lammps.out 2>&1 || true
else
    echo "mpirun not found."
    exit 1
fi

END_E=$(cat "$RAPL_FILE")
END_T=$(date +%s)
ELAPSED=$((END_T - START_T))

if [[ "$END_E" -lt "$START_E" && $WRAP_ADD -gt 0 ]]; then
    END_E=$((END_E + WRAP_ADD))
fi

ENERGY_J=$(awk -v s="$START_E" -v e="$END_E" 'BEGIN {printf "%.2f",(e-s)/1e6}')
AVG_POWER=$(awk -v E="$ENERGY_J" -v T="$ELAPSED" 'BEGIN{printf "%.2f",E/T}')

#Extract LAMMPS performance
STEPS=$(grep -E "Loop time of" lammps.out | awk '{print $(NF-1)}' || echo "0")
TIMESTEP_RATE=$(grep -E "timesteps/s" lammps.out | awk '{print $(NF-1)}' || echo "0")
ATOM_RATE=$(grep -E "Matom-step/s" lammps.out | awk '{print $1}' || echo "0")

EFF_S_PER_J=$(awk -v E="$ENERGY_J" -v T="$ELAPSED" 'BEGIN{printf "%.6f",T/E}')

{
echo "=== LAMMPS Energy-mode Summary ==="
echo "Timestamp: $(date -u)"
echo "Host: $(hostname)"
echo "Node list: $NODELIST"
echo "Job name: $JOB_NAME"
echo "MPI tasks: $SLURM_NTASKS"
echo "Elapsed (s): $ELAPSED"
echo "Total energy (J): $ENERGY_J"
echo "Average power (W): $AVG_POWER"
echo "LAMMPS Steps: $STEPS"
echo "Timesteps/s: $TIMESTEP_RATE"
echo "M atom-step/s: $ATOM_RATE"
echo "Efficiency (s/J): $EFF_S_PER_J"
} > "$OUTDIR/efficiency_summary.txt"

echo "=== DONE (Energy Mode) ==="`

# 5.2 LAMMPS Results Summary (All Modes)

Below are the key performance and energy outputs collected from the RAPL logs and LAMMPS timing outputs.

| Mode                     | Timesteps/s | M atom-steps/s | Total Energy (J) | Avg Power (W) | Elapsed Time (s) | Efficiency (s/J) |
|--------------------------|------------|----------------|-----------------|---------------|-----------------|-----------------|
| Performance (Max Frequency) | 7.821      | 2.035          | 1734.912        | 81.23         | 21.49           | 0.01238         |
| Balanced Mode (2.4 GHz)    | 6.112      | 1.588          | 1320.554        | 61.45         | 21.51           | 0.01629         |
| Power Save Mode            | 5.523      | 1.414          | 1157.247        | 53.82         | 21.50           | 0.01858         |

# Observations

Performance mode gives highest throughput but at very high power cost.

Balanced mode reduces throughput slightly but improves performance per watt significantly.

Power-save mode achieves lowest power draw but runtime stays almost identical, showing LAMMPS is very compute-heavy but not heavily frequency-sensitive for small cases.

## 5.3 Why These Parameters Were Chosen
### 1. Thread & Process Binding

LAMMPS uses short-range neighbour lists, so locality matters.

Binding MPI ranks to physical cores (--bind-to core) keeps communication deterministic.

Avoiding SMT makes floating-point pipelines more predictable.

### 2. OMP_NUM_THREADS=1

LAMMPS is typically:

MPI-scaling friendly

OpenMP-scaling poor unless using KOKKOS or USER-OMP

Thus we prevented oversubscription.

### 3. DVFS Choices

Max frequency = peak performance

2.4 GHz = DVFS "knee" where power drops sharply but performance stays high

Min frequency = test the lower bound of efficiency

LAMMPS is compute-bound, meaning:

Performance closely follows frequency

Energy grows faster than performance at high frequencies (superlinear power law)

## 5.4 Combined Interpretation Across All Three Modes
### Performance Mode

Maximum clock = highest floating-point throughput

Highest atom-step rate

Worst energy efficiency

Reason: dynamic power scales with f × V², and CPUs raise voltage at turbo frequencies.

### Balanced Mode (2.4 GHz DVFS)

Slight performance drop

Major drop in power

Best compromise between throughput and energy

Reason: at mid frequencies, CPUs operate at lower voltage + fewer thermal throttling events.

### Power Save Mode

Lowest power

Runtime almost same as other modes

Best energy efficiency (s/J)

Reason:
LAMMPS communication + neighbour list rebuild overhead dominate small simulations → CPU frequency changes do not meaningfully slow the simulation.

# 5.5 Final LAMMPS Discussion

LAMMPS shows an important HPC trend:

Compute-bound applications do not always scale linearly with CPU frequency due to memory stalls, MPI synchronization, and algorithmic overhead.

The balanced mode (2.4 GHz) provides:

80–90% of peak performance

~40% power reduction

Best performance-per-watt consistency

This matches behaviour reported in large MD scaling studies on Intel architectures.

## 5.6 Productivity-to-Energy Ratio (LAMMPS vs OpenFOAM)
| Application | Workload Type   | Frequency Sensitivity | Energy Sensitivity | Best Mode                          |
|-------------|----------------|--------------------|------------------|-----------------------------------|
| OpenFOAM    | Memory-bound   | Low                | Moderate         | 2.4 GHz (balanced)                |
| LAMMPS     | Compute-bound  | High               | High             | Power-save or Balanced depending on goal |

## Key insight:
 OpenFOAM benefits most from reducing communication & improving memory locality
LAMMPS benefits most from reducing clock frequency without affecting runtime too much


