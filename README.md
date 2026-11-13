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
