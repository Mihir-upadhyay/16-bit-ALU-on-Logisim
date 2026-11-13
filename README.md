# 16-Bit Custom CPU in Logisim

A fully functional, 16-bit processor built from scratch in Logisim, based on the classic M. Morris Mano computer architecture. This project features a custom Arithmetic Logic Unit (ALU), a synchronous two-phase control unit, and a complete custom Instruction Set Architecture (ISA) capable of executing memory-reference and register-reference commands, including loops and conditional branching.

## Features
* **16-Bit Architecture:** 16-bit data bus and 12-bit memory addressing, supporting up to 4096 words of RAM.
* **Custom ALU:** Performs hardware addition, bitwise AND, bitwise complement (NOT), and circular shifts.
* **Hardware 2's Complement:** Executes subtraction natively via `CMA` and `INC` combinations.
* **Synchronous Two-Phase Clocking:** Custom-built Sequence Counter utilizing falling-edge D Flip-Flops to eliminate simulator race conditions.
* **Turing Complete:** Supports unconditional branching (`BUN`), memory storing (`STA`), and conditional execution.

## Instruction Set Architecture (ISA)
| Opcode (Hex) | Mnemonic | Type | Description |
| :--- | :--- | :--- | :--- |
| `0XXX` | `AND` | Memory | Bitwise AND RAM data with Accumulator (AC) |
| `1XXX` | `ADD` | Memory | Add RAM data to AC |
| `2XXX` | `LDA` | Memory | Load RAM data into AC |
| `3XXX` | `STA` | Memory | Store AC data into RAM |
| `4XXX` | `BUN` | Memory | Branch Unconditionally to address |
| `7001` | `HLT` | Register | Halt the master clock |
| `7020` | `INC` | Register | Increment AC by 1 |
| `7040` | `CIL` | Register | Circulate left (Shift Left / Multiply by 2) |
| `7080` | `CIR` | Register | Circulate right (Shift Right / Divide by 2) |
| `7200` | `CMA` | Register | Complement AC (Bitwise NOT) |

## Prerequisites
* [Logisim 2.7.1](http://www.cburch.com/logisim/) (or Logisim-Evolution).

## How to Run
1. Clone the repository and open `main_cpu.circ` in Logisim.
2. Open the RAM component and load your desired Hex program starting at address `000`.
3. Reset the simulation (`Ctrl + R`).
4. Enable Auto-Tick (`Ctrl + K`) or manually step through the clock cycles to trace data execution.

# Design and Implementation of a 16-Bit Instruction Set Processor
**Based on the M. Morris Mano Architecture**

## 1. Abstract
This report details the design, implementation, and verification of a 16-bit custom Central Processing Unit (CPU) constructed entirely using standard logic gates in the Logisim simulation environment. The architecture is based on the M. Morris Mano computer system, utilizing a common bus system, a custom Arithmetic Logic Unit (ALU), and a hardwired Control Unit. The processor successfully executes memory-reference instructions, register-reference instructions, and basic data routing, proving Turing completeness through unconditional branching and memory-write capabilities.

## 2. System Architecture
The CPU is designed around a unified 16-bit system bus that handles the routing of instructions, data, and addresses between the memory and the physical registers.

### 2.1 Register Configuration
The system relies on specific registers to decouple memory latency from ALU execution:
* **Program Counter (PC - 12 bits):** Tracks the memory address of the next instruction.
* **Address Register (AR - 12 bits):** Acts as the sole gatekeeper for RAM address lines, preventing bus collisions during execution phases.
* **Instruction Register (IR - 16 bits):** Holds the fetched instruction. It is divided into an Opcode (Bits 12-14), an Address (Bits 0-11), and an Indirect Flag (Bit 15).
* **Accumulator (AC - 16 bits):** The primary target for all ALU operations.
* **Data Register (DR - 16 bits):** Buffers data fetched from RAM to ensure stable ALU inputs.

### 2.2 The Arithmetic Logic Unit (ALU)
The ALU is constructed using an 8-to-1 Multiplexer controlled by a 3-bit `ALU_SELECT` tunnel. It processes inputs from the Accumulator and the Data Register.
* **Input 0 (`000`):** AC AND DR
* **Input 1 (`001`):** AC + DR (Arithmetic Addition)
* **Input 2 (`010`):** Pass DR (Used for LDA)
* **Input 4 (`100`):** Complement AC (Bitwise NOT)
* **Input 5 (`101`):** Shift Right (CIR)

## 3. Timing and Control Unit
The Control Unit is hardwired, utilizing a combination of instruction decoders and a timing sequence counter to generate exact control signals.

### 3.1 The Instruction Cycle
The execution of any instruction follows a strict timing sequence dictated by the $T_0$ to $T_5$ clock states:
* **Fetch Phase ($T_0 - T_1$):** The PC address is moved to the AR. RAM outputs the instruction to the IR, and the PC increments.
* **Decode Phase ($T_2$):** The IR's opcode is pushed through a 3-to-8 Decoder, isolating the active instruction (e.g., $D_1$ for ADD, $D_3$ for STA).
* **Execute Phase ($T_3 - T_5$):** Control signals are generated based on the boolean logic of $D_n \cdot T_n$.

### 3.2 Two-Phase Clocking & Race Condition Mitigation
Initial designs revealed a critical race condition: asynchronous control signal degradation. Because the Sequence Counter and the operational registers were both triggering on the rising edge of the master clock, the counter would reset to $T_0$ before the Accumulator could securely latch the $T_5$ data, resulting in dropped values. 

This was resolved by implementing Two-Phase Clocking. The registers were left to trigger on the rising edge (The Action Phase), while the D Flip-Flops in the Sequence Counter were modified to trigger exclusively on the falling edge (The Cleanup Phase). This decoupled the state change from the data latch, ensuring absolute signal stability.

## 4. Hardware Implementations & Mathematical Logic

### 4.1 Subtraction via 2's Complement
To optimize hardware real estate, a dedicated subtractor circuit was omitted. Subtraction is performed via software algorithms utilizing the `CMA` and `INC` hardware instructions.
To compute `A - B`:
1. Load `B` into the AC.
2. Execute `CMA` (Complement AC).
3. Execute `INC` (Increment AC). *AC now holds the true negative of B.*
4. Execute `ADD A`.

### 4.2 Clock Gating (The HLT Instruction)
The `HLT` instruction (`7001`) prevents the CPU from inadvertently fetching and executing raw data. It is implemented via clock gating. A master Start/Stop D Flip-Flop connects to the main clock via an AND gate. When the control unit decodes $D_7 \cdot T_3 \cdot B_0$, the resulting signal clears the Flip-Flop, forcing the output to `0`. This physically blocks the clock pulse from reaching the Sequence Counter, gracefully freezing the CPU.

## 5. Debugging and Fault Analysis
During the build phase, several major architectural faults were diagnosed and repaired:
* **The High-Z MUX Fault (`UUUU` state):** The primary ALU Multiplexer outputted undefined states. This was traced to a floating Enable pin on the MUX. By overriding the Include Enable property in Logisim, continuous data routing was restored.
* **The Floating Bus Fault (`FFFF` write error):** During the `STA` execution, memory slots were overwritten with `FFFF` instead of AC data. Analysis showed the RAM Write Enable (WE) pin was active, but the System Bus multiplexer was not commanded to read the Accumulator at $T_4$. The control wire $D_3 \cdot T_4$ was rerouted to force the bus selection to the AC, successfully grounding the data path.

## 6. Conclusion
The construction of this 16-bit processor successfully demonstrates the core principles of von Neumann architecture. By independently handling the memory bottlenecks, ALU routing, and complex timing race conditions, the resulting CPU provides a stable, Turing-complete foundation capable of being expanded with I/O peripherals and deeper pipelining in the future.
