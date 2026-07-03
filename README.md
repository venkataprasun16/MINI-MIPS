# Mini MIPS Processor & Hardware Acceleration
This repository showcases projects in computer organization and digital design—spanning foundational digital arithmetics, a complete multi-cycle 32-bit MIPS processor, FPGA hardware accelerators, and MIPS assembly algorithms. 

All hardware IP was implemented in **Verilog**, synthesized with **Xilinx Vivado**, and validated on the **PYNQ-Z2 FPGA board**.

---

## Repository Structure

The architecture is categorized strictly by functional domain.

```text
MINI-MIPS/
│
├── 01_Digital_Logic_and_Arithmetics/
│   ├── src/
│   │   ├── full_adder.v
│   │   ├── comparator.v
│   │   ├── multiplier.v
│   │   ├── divider.v
│   │   └── fibonacci_fsm.v      # 64-bit continuous state sequence generator
│   └── tb/
│       └── tb_fibonacci_fsm.v   # Testbenches
│
├── 02_MIPS_Processor/
│   ├── src/
│   │   ├── defs.vh              # Opcode, func, and syscall macro definitions
│   │   ├── Computer.v           # Top-level: memory arbitration & cycle counting
│   │   ├── Memory.v             # 1024-byte unified memory (READ, WRITE, SUBWORD_WRITE)
│   │   ├── Processor.v          # 3-cycle FSM: fetch, execute, writeback & stall logic
│   │   ├── RegisterFile.v       # 32 general-purpose registers
│   │   └── ALU.v                # Integer arithmetic, branching, and address computation
│   └── tb/
│       └── Computer_tb.v        # Validation including subword variants
│
├── 03_FPGA_Acceleration/
│   ├── vector_addition/
│   │   ├── vector_add_accelerator.v
│   │   └── vitis_vector_add.c   
│   ├── matrix_vector_mul/
│   │   ├── mat_vec_mul_accelerator.v
│   │   └── vitis_mat_vec_mul.c 
│   └── graph_path_finding/
│       ├── path_finder_accelerator.v
│       └── vitis_path_finder.c  
│
├── 04_MIPS_Assembly/
│   └── src/
│       ├── array_histogram.s
│       ├── bisection_root.s
│       └── taylor_tan.s
│
└── README.md
```
## 1. Digital Logic & Arithmetic Units
- **Combinational Logic**: Multi-bit adders and cascading comparators.
- **Complex Arithmetic**: Custom divider and multiplier units.
- **Iterative Computation**: 64-bit Fibonacci sequence generator managing continuous state transitions and large register widths.

## 2. 32-bit MIPS Processor
A 3-cycle multi-cycle MIPS processor (Fetch/Decode → Execute → Writeback/PC Update) with a unified memory architecture and execution-priority memory arbitration.

**Instruction Set**
- **Arithmetic & Logic**: `add`, `sub`, `and`, `or`, `xor`, `nor`, `sll`, `sra`, `slt`
- **Control Flow**: `beq`, `bne`, `blez`, `j`, `jal`, `jr`
- **Load/Store**: `lw`, `sw`, `lui`
- **Big-Endian Subwords**: `lb`, `lbu`, `lh`, `lhu`, `sb`, `sh`

**Key Features**
- **Subword Memory Encoding**: Dynamic extraction and sign-extension (`lb`/`lh`), alongside read-modify-write protocols (`sb`/`sh`) for isolated byte storage.
- **I/O Handshaking**: `SYS_read` and `SYS_write` syscalls trigger pipeline stalls, halting execution cleanly until an `input_value_valid` signal is pulled from the environment.

## 3. FPGA Hardware Acceleration
Linear algebra workflows offloaded to the FPGA PL (Programmable Logic), controlled by the ARM Cortex-A9 PS via AXI-Lite Slave Registers. Validated via cross-compiled C programs using Xilinx Vitis IDE.

| Application | Hardware Approach | Speedup over ARM |
|---|---|---|
| **Vector Addition** | `generate for` loops for 512 parallel adds (1 cycle) + 9-cycle tree reduction. | ~150–200× |
| **Matrix-Vector Mul** | Row inner products via 4-level tree reduction (7 cycles per row). | ~16× |
| **Graph Pathfinding** | Boolean matrix multiplication ($A^k$) using packed 32-bit vector bitwise-ANDs (8 rows/cycle). | ~1000× |

## 4. MIPS Assembly Programming
Mathematical algorithms formulated in standard MIPS assembly for the QtSPIM simulator.

- **Histogram Generator**: Uses File I/O (`syscall 13, 14, 16`) to parse newline-delimited positive integers into ASCII bucket arrays.
- **Bisection Root Finder**: Recursive search leveraging IEEE-754 single-precision floating-point instructions (`lwc1`, `add.s`, `c.lt.s`, `bc1t`).
- **Maclaurin Series Expansion**: Floating-point calculator evaluating $\tan(x)$ via sequential $\sin(x)$ and $\cos(x)$ series expansions.
