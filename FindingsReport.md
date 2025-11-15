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

## OpenFOAM TUNNING 


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
