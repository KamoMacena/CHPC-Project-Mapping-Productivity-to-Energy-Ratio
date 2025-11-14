# Cluster System Information Report

**Host:** smshost  
**Date:** Fri Nov 14 03:44:04 SAST 2025  

---

## 1. Basic System Info
| Field | Value |
|-------|-------|
| Hostname | smshost |
| Uptime | 10 days 13:56 |
| Load Average | 0.01 / 0.08 / 0.10 |
| OS | Rocky Linux 8.10 (Green Obsidian) |
| Kernel Platform | el8 |
| Support End | 2029-05-31 |

---

## 2. CPU Information
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

---

## 3. Memory Information
| Field | Value |
|-------|-------|
| Total RAM | 31 GiB |
| Used | 2.1 GiB |
| Free | 10 GiB |
| Buff/Cache | 18 GiB |
| Available | 26 GiB |
| Swap Total | 15 GiB |
| Swap Used | 1.0 GiB |

---

## 4. GPU / Accelerator
| Field | Value |
|-------|-------|
| NVIDIA GPU | None detected |
| Drivers | Not installed |

---

## 5. Network Interfaces
| Interface | State | IP Address | Notes |
|-----------|-------|------------|-------|
| lo | UP | 127.0.0.1 | Loopback |
| eno1 | UP | 10.128.23.188 | Main LAN |
| eno2 | UP | 10.10.10.10 | Cluster private network |
| ib0 | DOWN | — | Mellanox IB (ConnectX-3) |
| virbr0 | DOWN | 192.168.122.1 | Virtual bridge |

### PCI Network Hardware
| Device | Model |
|--------|-------|
| 02:00.0 | Intel I350 NIC |
| 02:00.3 | Intel I350 NIC |
| 82:00.0 | Mellanox ConnectX-3 |



---

## 7. Power (RAPL)
| Zone | Type | Energy (uJ) |
|------|------|-------------|
| intel-rapl:0 | package-0 | 16,511,888,631 |
| intel-rapl:0:0 | core | 43,998,797,599 |
| intel-rapl:0:1 | dram | 20,217,413,493 |
| intel-rapl:1 | package-1 | 61,947,548,996 |
| intel-rapl:1:0 | core | 21,046,507,595 |
| intel-rapl:1:1 | dram | 63,360,429,543 |

---

## 8. SLURM Node Status
| Node | CPUs | State | Reason |
|------|------|-------|--------|
| compute00 | 16 | DOWN* | Troubleshooting node |
| compute01 | 16 | DOWN* | Manual reset |
| compute02 | 16 | DOWN* | Not responding |



## 9. Loaded Software Modules
| Module Category | Modules Loaded |
|-----------------|----------------|
| Compilers | gnu12/12.4.0 |
| MPI | openmpi4/4.1.6 |
| Core Tools | autotools, hwloc, libfabric, ucx, prun |
| Math & Sci | openblas, petsc, fftw, scalapack, mumps, trilinos |
| Profiling | tau, scalasca, scorep, PAPI |

---

## 10. MPI Information
| Field | Value |
|-------|-------|
| MPI Implementation | OpenMPI 4.1.6 |
| Path | /opt/ohpc/pub/mpi/openmpi4-gnu12/4.1.6/bin/mpirun |

---
