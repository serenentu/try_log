# IE4756 Simple MIPS CPU Assignment

## Student Information
- **Student ID:** U2321215G (7-digit part: 2321215)
- `id % 9 = 7` → Sections encoded: **Section 7** and **Section ★**

---

## Files Changed and Sub-circuits Modified

### Part 1 (1) — Complete the Fast-Adder Circuit

| File | Sub-circuit |
|------|-------------|
| `IE4756-fast-adder.circ` | `add-32` |

The `add-32` sub-circuit was completed to perform 32-bit addition. It takes two 32-bit inputs and a carry-in, and produces a 32-bit sum and carry-out using Logisim's built-in 32-bit Adder component.

---

### Part 1 (2) — Complete the ALU

| File | Sub-circuit(s) |
|------|----------------|
| `IE4756-arithmetic-logic-no-adder.circ` | `eq (4-bit)`, `eq (8-bit)`, `eq (32-bit)`, `eq/lt unsigned unit`, `eq/lt unsigned (8-bit)`, `eq/lt unsigned (32-bit)`, `eq/lt signed (8-bit)`, `eq/lt signed (32-bit)`, `shiftl (8-bit)`, `shiftl (32-bit)`, `shiftr (8-bit)`, `shiftr (32-bit)`, `shiftl variable (32-bit)`, `shiftr variable (32-bit)` |
| `IE4756-mips-cpu.circ` | `ALU (32-bit)` |

The `IE4756-arithmetic-logic-no-adder.circ` file provides comparison (equal/less-than) and shift sub-circuits used by the ALU. The `ALU (32-bit)` sub-circuit in `IE4756-mips-cpu.circ` was completed to support all operations specified in the CPU control bit table (AND, OR, XOR, NOR, ADD, ADDU, SUB, SUBU, SLT, SLTU, SLL, SRL, SRA, LUI, and shift-variable variants), controlled by the 4-bit ALU control signal.

---

### Part 1 (3) — Add Support for `jalr` Instruction

| File | Sub-circuit(s) |
|------|----------------|
| `IE4756-mips-cpu.circ` | `CPU control unit`, `CPU (MIPS32)` |

The `CPU control unit` sub-circuit was modified to decode the `jalr` instruction (opcode `000000`, funct `001001`) and assert the **Jump Register** output signal while also asserting **Register Write** (to save PC+4 into `$rd`). The `CPU (MIPS32)` top-level circuit was updated to route the **Jump Register** control signal to the program counter multiplexer so that the next PC is loaded from register `$rs`.

---

### Part 2 — MIPS Assembly Encoding

| File | Description |
|------|-------------|
| `mips-encoding.txt` | Hex encodings for Section 7 and Section ★ |

#### Section 7

| Assembly | Hex |
|----------|-----|
| `ori $8, $1, 4` | `0x34280004` |
| `lw $5, 0($8)` | `0x8D050000` |
| `ori $2, $0, 0` | `0x34020000` |
| `ori $3, $0, 0` | `0x34030000` |
| `lui $1, 64` | `0x3C010040` |
| `ori $31, $1, 308` | `0x343F0134` |
| `j 0x00400084` | `0x08100021` |
| `lui $1, 4097` | `0x3C011001` |

#### Section ★

| Assembly | Hex |
|----------|-----|
| `bne $17, $0, -28` | `0x1620FFF9` |
| `ori $3, $4, 0` | `0x34830000` |
| `jr $31` | `0x03E00008` |
| `addi $29, $29, -4` | `0x23BDFFFC` |
| `sw $31, 0($29)` | `0xAFBF0000` |
| `lui $8, 4097` | `0x3C081001` |