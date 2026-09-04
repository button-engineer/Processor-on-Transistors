# 🖥️ 4-Bit Custom CPU Architecture (74HC Logic)

This repository contains the open-source design and physical implementation of a custom 4-bit CPU built using discrete **74HC-series** logic integrated circuits. The project covers the full engineering cycle: from developing and simulating the architecture in **Logisim-Evolution** to assembling hardware on breadboards with visual LED diagnostics.

---

## 📐 Key Specifications

* **Data Bus Width:** 4 bits[cite: 1].
* **Instruction Length:** 8 bits (1 byte)[cite: 1].
* **Opcode Size:** 4 bits (supports up to 16 unique instructions)[cite: 1].
* **Register File:** 4 general-purpose 4-bit registers (R0, R1, R2, R3)[cite: 1].
* **Memory Addressing:** 8-bit Program Counter and up to 256 bytes/cells of SRAM[cite: 1].
* **Hardware Base:** 74HC Series Logic (High-speed CMOS, DIP packages).
* **Simulation & Verification:** Logisim-Evolution.

---

## 🧩 8-Bit Instruction Format

Each instruction in the CPU occupies 8 bits[cite: 1]:

| Bits `[7:4]` | Bits `[3:2]` | Bits `[1:0]` |
| :---: | :---: | :---: |
| **Opcode (Instruction)**[cite: 1] | **Destination Register (R1)**[cite: 1] | **Source Register / Parameter (R2)**[cite: 1] |

### Instruction Encoding Logic:
1. **Opcode (4 bits):** Determines the operation to be performed by the CPU (e.g., `1011` for `LDI`, `0011` for `ADD`)[cite: 1].
2. **Register Addressing (2 bits each):** Used to select one of the 4 registers[cite: 1]:
   * `00` — R0
   * `01` — R1
   * `10` — R2
   * `11` — R3
3. **Result Storage:** Since there are no extra bits to encode a 3rd register destination, the result of arithmetic/logic operations is always written back to the first register specified in the instruction (R1)[cite: 1].
4. **Two-Byte Instructions:** Commands that require immediate data or an 8-bit memory address (such as `LD`, `ST`, `LDI`, `JMP`, `JZ`, `JC`) fetch an additional 8-bit byte from program memory during the next instruction byte/cycle[cite: 1].

---

## 📋 Instruction Set Architecture (ISA)

Complete list of all 16 processor instructions[cite: 1]:

| Opcode | Name | Syntax | Binary Code / Bytes | Description |
| :---: | :---: | :--- | :--- | :--- |
| `0000` | **NOP** | `NOP` | `00000000` | No operation. Skips one clock cycle; can be used for timing delays[cite: 1]. |
| `0001` | **LD** | `LD R1 [R3]` | `00010100`<br>`00000011` | Loads a value from SRAM into a register. The SRAM memory address (up to 256) is passed as the next 8-bit instruction byte[cite: 1]. |
| `0010` | **ST** | `ST [R4] R2` | `00100010`<br>`00000100` | Stores a register value into SRAM (SRAM address passed as the next byte)[cite: 1]. |
| `0011` | **ADD** | `ADD R0 R1` | `00110001` | Adds two 4-bit numbers from registers; stores the result in R0[cite: 1]. |
| `0100` | **ADC** | `ADC R0 R1` | `01000001` | Adds two 4-bit numbers from registers including the carry flag from the previous addition[cite: 1]. |
| `0101` | **SUB** | `SUB R0 R1` | `01010001` | Subtracts the value in R1 from the value in R0[cite: 1]. |
| `0110` | **AND** | `AND R0 R1` | `01100001` | Bitwise AND operation between values in registers 0 and 1[cite: 1]. |
| `0111` | **OR** | `OR R0 R1` | `01110001` | Bitwise OR operation between values in registers 0 and 1[cite: 1]. |
| `1000` | **XOR** | `XOR R0 R1` | `10000001` | Bitwise XOR operation between values in registers 0 and 1[cite: 1]. |
| `1001` | **INC_DEC** | `INC_DEC R0 1/0` | `10010001` / `10010000` | Increments (+1) or decrements (-1) the value in the specified register depending on the trailing bit[cite: 1]. |
| `1010` | **SHF** | `SHF R0 1/0` | `10100001` / `10100000` | Shift: multiplies by 2 (shift left, 1) or divides by 2 (shift right, 0) the value in the specified register[cite: 1]. |
| `1011` | **LDI** | `LDI R1 13` | `10110100`<br>`00001101` | Loads an immediate 4-bit constant into the register (constant passed in the next byte)[cite: 1]. |
| `1100` | **MOV** | `MOV R0 R1` | `11000001` | Moves (copies) a value from register R1 into register R0[cite: 1]. |
| `1101` | **JMP** | `JMP 123` | `11010000`<br>`01111011` | Unconditional jump to an 8-bit instruction address provided in the next byte[cite: 1]. |
| `1110` | **JZ** | `JZ R1 123` | `11100100`<br>`01111011` | Conditional jump to the target address if the specified register equals 0[cite: 1]. |
| `1111` | **JC** | `JC 123` | `11110000`<br>`01111011` | Conditional jump to the target address if a carry/overflow occurred[cite: 1]. |

---

## ⚙️ Hardware Implementation (74HC Logic) & ALU

### 1. ALU Implementation & Math Operations
* **Software Multiplication & Division:** Hardware implementation of multiplication and division units was deemed too transistor-heavy and complex[cite: 1]. Therefore, these operations are handled **software-side** using logical shifts (`SHF`), addition (`ADD`), subtraction (`SUB`), and loops[cite: 1].
* **Core Arithmetic Logic:** The ALU uses 4-bit adders (**74HC283**) paired with logic gates (XOR **74HC86**, AND **74HC08**, OR **74HC32**) to perform the complete set of arithmetic and logic operations[cite: 1].

### 2. 74HC IC Architecture Breakdown
To improve operational speed, signal stability, and layout density, the hardware utilizes CMOS **74HC** series ICs in DIP packages:
* **Register File (R0–R3) & IR:** Constructed using 4-bit / 8-bit D-type flip-flop latches (e.g., **74HC173** / **74HC374**).
* **Bus Control (3-State Buffers):** Uses tri-state bus transceivers/buffers (e.g., **74HC244** / **74HC245**) to manage data flow and prevent bus contention on the shared 4-bit data bus.
* **Program Counter (PC):** Built with cascaded 4-bit binary counters (**74HC161** / **74HC191**) providing an 8-bit address space.
* **Control Unit & Decoders:** Leverages **74HC138** / **74HC154** decoders to generate read/write enable control signals for registers, ALU, and memory.

### 3. Diagnostics & Clocking
* **LED Status Indicators:** Every critical control line, data bus, address line, register output (R0–R3), and status flag (Carry/Zero) is equipped with LEDs for real-time visual hardware debugging.
* **Clock Generator:** Features dual modes: single-step mode for manual instruction-by-instruction debugging and automated clock generation based on NE555 or 74HC14 Schmitt triggers.

---

## 📂 Repository Structure

```text
├── sim/                # Logisim-Evolution simulation files and schematics
├── hardware/           # KiCad / EasyEDA schematics, PCB layouts, and Bill of Materials (BOM)
├── docs/               # ISA specifications, timing diagrams, and control signal tables
└── README.md           # Main repository documentation
