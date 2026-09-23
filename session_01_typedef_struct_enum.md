# Session 01: SystemVerilog Types — `typedef`, `struct`, and `enum`

**Date:** 2026-09-23  
**Topic:** SystemVerilog Data Types in RTL Design & Verification Testbenches  
**Target Domain:** RTL Design, SVA, and UVM Testbench Architectures  

---

## 1. Executive Overview

SystemVerilog introduces user-defined types (`typedef`), aggregate data structures (`struct`), and enumerated types (`enum`) to elevate hardware verification and RTL design abstraction. 

| Construct | Primary Design Use Case | Primary Testbench (TB) Use Case | Key Constraint / Rule |
| :--- | :--- | :--- | :--- |
| **`enum`** | State Machine (FSM) states, control logic flags | Transaction opcodes, status codes, coverage bins | Strictly typed; assigning raw integers requires dynamic `$cast()` |
| **`struct`** | Module I/O signal bundling, packed control registers | Bridging dynamic UVM classes to static HDL BFM interfaces | Unpacked structs cannot perform vector/arithmetic operations |
| **`typedef`** | Parameterized width aliases, standard package types | Queue/array shortcuts, forward class declarations | Class-scoped typedefs are static and accessed via `::` |

---

## 2. Enumerated Types (`enum`)

`enum` defines a set of named integer constants. By default, an `enum` without an explicit data type defaults to `int` starting at value `0`.

### A. Design & Testbench Code Examples

```systemverilog
// DESIGN: FSM State encoding using 4-state 'logic' type
// Assigning 'x allows 'x-propagation debug during simulation
typedef enum logic [1:0] {
  IDLE    = 2'b00,
  READ    = 2'b01,
  WRITE   = 2'b10,
  ILLEGAL = 'x
} state_e;

// TESTBENCH: Sequence item transaction opcode
typedef enum bit [3:0] {
  OP_ADD = 4'h0,
  OP_SUB = 4'h1,
  OP_MUL = 4'h2,
  OP_NOP = 4'hF
} opcode_e;
```

### B. Corner Cases & Common Pitfalls

#### 1. Ascending Order Interruption ('x / 'z member)
When assigning `'x` or `'z` to an `enum` member, the subsequent member **must** be assigned an explicit value because the compiler cannot auto-increment from an un-incrementable value.
```systemverilog
// ILLEGAL: Simulator cannot increment from 'x for S1
// enum integer {IDLE, XX='x, S1, S2} state;

// LEGAL: Explicit value given to S1
enum integer {IDLE, XX='x, S1=2'b01, S2=2'b10} state;
```

#### 2. Duplicate Value Collisions
Auto-incrementing values can accidentally collide with explicitly assigned subsequent values.
```systemverilog
// ILLEGAL: 'b' is 7 -> 'c' auto-increments to 8 -> collides with d=8!
// enum {a=0, b=7, c, d=8} alphabet;

// CORRECT: Ensure unique values across all labels
enum {a=0, b=7, c=8, d=9} alphabet;
```

#### 3. Base Type Bit-Width Overflow
An explicit member value exceeding the declared base type bit-width results in a compilation error.
```systemverilog
// ILLEGAL: 'h13 (19 in decimal) exceeds 4-bit limit
// enum bit [3:0] {RED = 'h13, GREEN, BLUE} color;
```

#### 4. Type Incompatibility & Dynamic Casting (`$cast`)
An `enum` can be directly assigned to an `int`, but an `int` **cannot** be directly assigned to an `enum`. You must use `$cast()`.
```systemverilog
int raw_opcode = 1;
opcode_e op;

// op = raw_opcode; // ILLEGAL: Compile error / warning

// LEGAL: Dynamic type cast check
if (!$cast(op, raw_opcode)) begin
  $error("Dynamic cast failed: %0d is not a valid opcode_e member", raw_opcode);
end
```

#### 5. Built-in Enum Methods
SystemVerilog provides built-in methods for enum traversal: `.first()`, `.last()`, `.next()`, `.prev()`, `.num()`, and `.name()`.
```systemverilog
opcode_e op = OP_ADD;
$display("Opcode Name: %s, Value: %0d", op.name(), op); // Returns "OP_ADD", 0
op = op.next(); // Advances to OP_SUB
```

---

## 3. Structures (`struct`)

A `struct` represents a collection of heterogeneous data types referenced as a single bundle or via individual member fields.

### A. Packed vs. Unpacked Comparison

| Property | Packed Structure (`struct packed`) | Unpacked Structure (`struct`) |
| :--- | :--- | :--- |
| **Memory Allocation** | Contiguous bit vector in memory | Non-contiguous, simulator-dependent alignment |
| **Allowed Field Types** | Integral types only (`bit`, `logic`, `byte`, `int`) | Any data type (including `real`, `string`, unpacked arrays) |
| **Vector & Math Operations** | Acts as a single bit vector (bitwise, arithmetic allowed) | Illegal to perform math operations on the whole struct |
| **Signedness** | Can be explicitly declared `signed` or `unsigned` | Signing an unpacked struct is illegal |
| **Default Assignment** | Assigned via bit slice or `'{...}` format | Assigned field-by-field or via `'{...}` pattern |

### B. Design & Testbench Code Examples

```systemverilog
// PACKED STRUCT FOR DESIGN (Module I/O bundling)
typedef struct packed {
  logic [11:0] length;
  logic [63:0] address;
} pcie_header_s;

module pcie_endpoint (
  input  pcie_header_s hdr_in,  // Entire struct passed as single I/O port
  output logic         valid
);
  assign valid = (hdr_in.length > 0);
endmodule

// UNPACKED STRUCT FOR UVM TB / BFM BRIDGING
typedef struct {
  bit [31:0] addr;
  bit [31:0] data;
  bit        write_en;
  int        delay;
} bus_seq_item_s; // Static struct passed across TB to HDL BFM boundary
```

### C. Corner Cases & Common Pitfalls

#### 1. Arithmetic on Unpacked Structs
Attempting to treat an unpacked structure as a numerical vector causes a compiler error.
```systemverilog
struct { bit [7:0] a; bit [7:0] b; } unpk_s;
// unpk_s = unpk_s + 8'h12; // ILLEGAL: Unpacked structure cannot be used in arithmetic
```

#### 2. Non-Integral Types in Packed Structs
Including `real`, `shortreal`, or unpacked arrays inside a `struct packed` is illegal.
```systemverilog
// ILLEGAL: 'real' cannot be packed into a bit vector
// struct packed { real voltage; bit [7:0] id; } p_struct;
```

#### 3. Member Assignment & Default Initializers
Struct initialization uses the `'{...}` aggregate literal syntax.
```systemverilog
typedef struct {
  int addr = 32'hFF;
  int data;
  byte crc[4] = '{default:8'h00};
} bus_s;

bus_s b1 = '{addr: 32'h1000, data: 32'hA0A0, crc: '{1, 2, 3, 4}};
```

---

## 4. User-Defined Types (`typedef`)

`typedef` creates an alias for an existing data type or complex declaration, enabling uniform scope definition across testbenches and design packages.

### A. Design & Testbench Examples

```systemverilog
// Package for shared protocol types
package apb_pkg;
  typedef logic [31:0] apb_addr_t;
  typedef logic [31:0] apb_data_t;

  typedef struct packed {
    apb_addr_t paddr;
    apb_data_t pwdata;
    logic      pwrite;
  } apb_trans_s;
endpackage

// Class-scoped forward declaration to avoid circular dependencies
typedef class uvm_sequence_item;
```

### B. Corner Cases & Common Pitfalls

#### 1. Class-Scoped Typedefs
Typedefs declared inside a class are static by default and must be accessed outside using the scope resolution operator `::`.
```systemverilog
class packet;
  typedef enum {RED, GREEN, BLUE} color_e; // Class-scoped enum
  int addr;
endclass

// Outside scope:
packet::color_e pkt_color = packet::RED; // LEGAL
// packet::addr = 10;                     // ILLEGAL: 'addr' is non-static
```

#### 2. Forward Class Declaration (`typedef class`)
When two classes refer to each other (circular dependency), use a forward type declaration before the first class definition:
```systemverilog
typedef class driver; // Forward declaration

class monitor;
  driver drv_h;      // Uses handle before full driver definition
endclass

class driver;
  monitor mon_h;
endclass
```

---

## 5. Daily Summary Checklist for GitHub Upload

- [x] Mastered `enum` syntax, automatic incrementing rules, and `$cast` dynamics.
- [x] Understood `struct packed` vs `struct unpacked` memory layout & usage guidelines.
- [x] Learned `typedef` scope resolution (`::`) and forward class resolution (`typedef class`).
- [x] Built testbench-to-HDL interface bridging architectures using package structs.

---
*Generated for GitHub Repository Documentation*
