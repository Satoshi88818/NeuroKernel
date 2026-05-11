# NeuroKernel v8 — Hardened Autonomy

> **Mantra:** *"The OS is not just running on the hardware; it is constantly re-proving the hardware's existence."*

NeuroKernel v8 is a radiation-hardened, hardware-software co-designed operating system and Neuromorphic SoC architecture. It is specifically engineered for autonomous operation in extreme environments (Deep Space, High-Orbit, and Nuclear-Proximal) where silicon degradation is not a possibility, but a certainty.

Version 8 marks a shift from **Graceful Reconfiguration** to **Hardened Autonomy**, introducing temporal diversity, physical peripheral amputation, and self-distilling neural recovery.

---

## I. Architecture Overview

The v8 architecture is a vertically integrated stack where every layer—from the gate logic to the scheduling algorithm—is aware of the system's physical integrity.

### 1. Hardware Layer (ASIC/Verilog)
*   **Temporal Diversity Interlock [V8: 1-A]:** A staggered Triple Modular Redundancy (TMR) unit. Replicas B and C are delayed by 50 and 100 cycles respectively, ensuring that a single transient pulse (SEU) cannot corrupt all three streams at the same logical instruction.
*   **Credit-Based Bus Arbiter [V8: 1-B]:** Implements "Physical Bus Amputation." If a peripheral exceeds its transaction or interrupt budget (indicative of a "Babbling Idiot" fault), the ASIC physically disconnects the peripheral's `READY` line.
*   **Transparent RS-Engine [V8: 1-C]:** Hardware-accelerated Reed-Solomon (255,223) FEC-SRAM controller providing zero-latency error correction for the software stack.

### 2. Kernel Layer (Rust Microkernel)
*   **Ghost Scheduling [V8: 2-B]:** High-criticality tasks are dispatched through pseudo-randomly selected entry-point shims. This ensures that the same degraded silicon path is not hit repeatedly, reducing the probability of permanent-fault-induced task failure.
*   **XOR-Linked Shadow Stacks [V8: 2-A]:** Return addresses are XORed with a per-task, per-epoch nonce. This provides spatial and temporal separation for control-flow integrity, turning bit-flips into controlled exceptions.
*   **Damage-Map PUF [V8: 4-B]:** The chip's unique pattern of stuck-at faults is treated as a Physical Unclonable Function (PUF), used to cryptographically sign telemetry.

### 3. Fidelity Layer (Neuromorphic AI)
*   **Saliency-Aware Refresh [V8: 3-A]:** Neural weights are ranked by criticality. "Hub" neurons are refreshed every epoch via DMA, while "Leaf" neurons are only restored during critical degradation to save power and bus bandwidth.
*   **Self-Distillation [V8: 3-B]:** In `UltraDegraded` mode, healthy CNN filters re-train/distill weights for damaged filters in the background, allowing the system to "heal" its inference capabilities in-situ.

---

## II. Engineering Specifications

| Feature | Specification |
| :--- | :--- |
| **Language** | Rust (`no_std`, `nightly`) |
| **Target Architecture** | RISC-V 32-bit (Rad-Hardened SoC) |
| **Formal Logic** | P-TLA+ (Probabilistic TLA+) |
| **Verification** | Kani (Symbolic), SymbiYosys (ASIC Formal) |
| **ECC Implementation** | RS(255, 223) + CRC-32-C |
| **Scheduling** | EDF with Deadline Inheritance + Ghost Shims |

---

## III. Requirements

### Toolchain
- **Rust:** `nightly-2024-xx-xx`
- **Target:** `riscv32imac-unknown-none-elf`
- **Simulation:** `Icarus Verilog` (v12+) or `Verilator`
- **Formal Verification:** `TLA+ Toolbox`, `Kani Rust Verifier`

### Hardware (Target)
- NeuroKernel v8 Compliant SoC (or FPGA equivalent)
- 32 KB Sector-Addressable SRAM
- MRAM for Golden Image persistence

---

## IV. Deployment & Initialization

### 1. Bootstrap Sequence
The system must be initialized in a specific order to ensure FEC and PMP protections are active before the first task dispatch:
1.  **GF Table Init:** Initialize Galois Field tables for software RS fallback.
2.  **HW-FEC Probe [V8: 1-C]:** Probe `FEC_SRAM_STATUS_REG`. If present, enable hardware Reed-Solomon.
3.  **PUF Derivation [V8: 4-B]:** Read initial damage map and derive the epoch-1 cryptographic identity.
4.  **Spatial Allocator Init:** Query the ASIC heat map to blacklist "hot" memory sectors.
5.  **Scheduler Start:** Begin EDF dispatching with Ghost Shim rotation.

### 2. Compilation
```bash
# Build the Microkernel
cargo +nightly build --release -p nk-muk

# Simulate ASIC Logic
iverilog -g2012 -o nk_asic_v8 nk-asic/NKAsic.v
vvp nk_asic_v8
```

---

## V. Testing & Verification Methods

NeuroKernel v8 utilizes a "Tri-Layer Verification" strategy:

### 1. Formal Proof (Mathematical)
- **TLA+:** Proves that `P(HighCrit failure) < 10⁻⁹` even at a rate of 5 SEU/hour.
- **SymbiYosys:** Proves that the Temporal Stagger FIFO never bypasses the TMR voter.

### 2. Symbolic Execution (Software)
- **Kani:** Proves that the `XOR-Linked Shadow Stack` detects any single-bit flip and that the `puf_verify` function is constant-time.

### 3. Fault Injection (Simulation)
- **Verilog SEU Injection:** Random bit-flips are injected into the simulated ASIC's register file to verify TMR capture and majority voting correctness.

---

## VI. Future Enhancements

- **[V9: 1-A] 3D-Aware Spatial Scheduling:** Integration with 3D-stacked RAM to steer allocations away from vertically adjacent failing layers.
- **[V9: 2-B] Dynamic Distillation Kernels:** Allowing ground stations to upload new distillation "teachers" to adapt to unforeseen silicon aging patterns.
- **[V9: 4-C] Post-Quantum PUF:** Transitioning HMAC-SHA-256 to a quantum-resistant lattice-based signing scheme for multi-century missions.

---

**Contact:** *Lead Systems Architect — NeuroKernel Project By The Bunyip*  
**Status:** *Hardened Autonomy Verified — v8 Baseline Stable*
