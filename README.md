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

#### What the `add-32` sub-circuit must do

It is a **32-bit adder with carry-in and overflow detection**. Think of it as a signed integer addition unit: given two 32-bit operands and an optional carry-in, it produces a 32-bit sum, a carry-out bit, and a signed-overflow bit.

#### Port reference (open the circuit in Logisim to see these)

| Pin location | Direction | Width | Name | Meaning |
|---|---|---|---|---|
| `(60, 100)` | Input | 32 bit | **A** | First operand |
| `(150, 100)` | Input | 32 bit | **B** | Second operand |
| `(190, 270)` | Input | 1 bit | **Cin** | Carry-in (tie to `0` for a plain add) |
| `(260, 100)` | Output | 32 bit | **Sum** | A + B + Cin (lower 32 bits) |
| `(150, 270)` | Output | 1 bit | **Cout** | Carry-out of bit 31 |
| `(170, 270)` | Output | 1 bit | **Overflow** | Signed two's-complement overflow |

#### What is already wired

The current circuit already contains a **32-bit Adder** component (Logisim's built-in `#Arithmetic → Adder`) at position `(200, 190)` with all three data paths connected:

```
Pin A  (60,100)  ──────────────────► Adder input 1  (left-top)
Pin B (150,100)  ──────────────────► Adder input 2  (left-bottom)
Pin Cin(190,270) ──────────────────► Adder carry-in (bottom-left)
Adder sum output ──────────────────► Pin Sum (260,100)
Adder carry-out  ──────────────────► Pin Cout (150,270)
```

The addition, carry-out, and carry-in paths are **done**. The only unfinished part is:

```
Constant 0x0 ──────────────────────► Pin Overflow (170,270)   ← PLACEHOLDER — needs real logic
```

#### Step-by-step: how to compute the Overflow output

Signed overflow occurs when the carry into bit 31 **differs** from the carry out of bit 31.  
The formula is:

```
Overflow = Cin_31  XOR  Cout_31
```

where  
- **Cout_31** = the `Cout` pin already wired (carry out of the whole 32-bit adder), and  
- **Cin_31** = the carry that entered bit 31, i.e. the carry-out of the lower 31-bit addition.

**In Logisim, do this:**

1. **Delete** the wire from the `0x0` Constant to the Overflow pin `(170, 270)`.  
   (Right-click the wire → Delete, or select it and press Delete.)

2. **Add a second Adder** (31-bit) to compute the carry into bit 31:
   - Drag a new `#Arithmetic → Adder` onto the canvas and set its width to **31**.
   - Wire **A[30:0]** (bits 0–30 of Input A) into its input 1: use a **Splitter** on Input A to extract bits 0–30.
   - Wire **B[30:0]** (bits 0–30 of Input B) into its input 2: use a **Splitter** on Input B to extract bits 0–30.
   - Wire **Cin** into this 31-bit Adder's carry-in.
   - The **carry-out** of this 31-bit Adder is **Cin_31**.

3. **Add an XOR gate** (2-input, 1-bit):
   - One input ← carry-out of the 31-bit Adder (Cin_31).
   - Other input ← `Cout` (carry-out of the 32-bit Adder, already wired to `(150,270)`).
   - Output of XOR → Pin **Overflow** `(170, 270)`.

4. **Remove** (or leave disconnected) the constant `0x0` that used to feed the Overflow pin.

> **Tip — using Splitters to extract bits:**  
> Place a Splitter, set *Incoming* = 32, set only the bits you need (0–30) to output group `0`, set bit 31 to `none`.  
> Connect the 32-bit input wire to the Splitter's combined side; take the 31-bit group-0 side to the 31-bit Adder input.

#### How the 32-bit Adder inside the MIPS CPU is used

The `add-32` component appears three times in `IE4756-mips-cpu.circ`:

| Location in CPU | Purpose |
|---|---|
| `(810, 220)` | PC + 4 (next sequential instruction address) |
| `(170, 250)` | Branch target: PC+4 + sign-extended offset |
| `(440, 1010)` in ALU | ALU addition (ADD / ADDU / ADDI / LW / SW) |

All three callers tie Cin to `0`, so the carry-in path works unchanged; only the Overflow output matters for detecting signed arithmetic overflow in the ALU.

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