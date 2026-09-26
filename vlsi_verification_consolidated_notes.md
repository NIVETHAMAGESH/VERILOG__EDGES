
## 1. Core Concepts & Learnings

<table>
  <thead>
    <tr>
      <th style="background-color: #4CAF50; color: white; padding: 8px;">SystemVerilog Type</th>
      <th style="background-color: #4CAF50; color: white; padding: 8px;">Architectural Purpose & Key Takeaways</th>
      <th style="background-color: #4CAF50; color: white; padding: 8px;">Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 8px;"><b><span style="color: #2196F3;">Enum (enum)</span></b></td>
      <td style="padding: 8px;">Defines standardized transaction parameters (such as opcodes or responses) for sequence items to restrict generation to valid states. In the absence of an explicit data type, the default is <code>int</code>.</td>
      <td style="padding: 8px;"><i>Reviewed</i></td>
    </tr>
    <tr>
      <td style="padding: 8px;"><b><span style="color: #2196F3;">Struct (struct)</span></b></td>
      <td style="padding: 8px;">Packages data into a static format to cross the boundary between the dynamic UVM class domain and the static HDL interface domain. By default, structures are unpacked.</td>
      <td style="padding: 8px;"><i>Reviewed</i></td>
    </tr>
    <tr>
      <td style="padding: 8px;"><b><span style="color: #2196F3;">Typedef (typedef)</span></b></td>
      <td style="padding: 8px;">Aliasing an existing complex type to a user-defined name. Often used to alias structs inside shared packages so both the testbench and the BFM interface can recognize and compile the exact same data type.</td>
      <td style="padding: 8px;"><i>Reviewed</i></td>
    </tr>
  </tbody>
</table>

---

## 2. Detailed Breakdown & Code Snippets

### A. Enumerated Types (enum)
Enumerated types define a set of named values. 
They are heavily used in design for Finite State Machines (FSMs) and in testbenches for transaction opcodes or status flags.

**Example: Sequence Item Constraints**
```systemverilog
class mbus_seq_item extends uvm_sequence_item;
  // Enum used for operation codes driven by the master
  rand mbus_opcode_e MOPCODE;
  
  // Enum used for responses driven by the slave
  mbus_resp_e MRESP;
endclass
```

**Corner Cases & Pitfalls:**
*   **Ascending Order Interruption:** If you assign an unknown value (`'x`) or high-impedance (`'z`), you must explicitly assign a numeric value to the immediately following member.
*   **Duplicate Value Collisions:** An auto-incrementing member can accidentally collide with a subsequent explicitly defined member.
*   **Dynamic Casting:** An `int` cannot be directly assigned to an enum. You must use `$cast` to resolve the type incompatibility.

### B. Structures (struct)
Structures represent a collection of the same or different data types. A packed structure (`struct packed`) stores bit fields contiguously and acts as a vector, while an unpacked structure (`struct`) is non-contiguous and can hold arrays or reals.

**Example: Struct and Typedef Usage Across TB/HDL Boundary**
To avoid passing dynamic UVM classes directly into hardware interfaces, the testbench converts the class into a static struct before passing it to the Bus Functional Model (BFM).

```systemverilog
// 1. Interface domain importing the typedef struct
interface apb_driver_bfm (apb_if APB);
  import apb_shared_pkg::apb_seq_item_s; 
  
  task do_item(apb_seq_item_s req, int psel_index, output apb_seq_item_s rsp);
    APB.PADDR <= req.addr;
    APB.PWDATA <= req.data;
    rsp = req; 
    rsp.error = (psel_index < 0);
  endtask
endinterface
```

**Corner Cases & Pitfalls:**
*   **Arithmetic on Unpacked Structs:** Attempting to treat an unpacked structure like a vector is a compile error.
*   **Illegal Types in Packed Structs:** Packing a struct containing a `real` type or an unpacked array will fail compilation.

### C. User-Defined Types (typedef)
For maximum reusability, `typedef` declarations are typically placed inside a package (`_pkg`) and imported into RTL modules or testbench files.

**Example: Standard Queue Types**
```systemverilog
// Defining standard queue types for TLM or scoreboard modeling
typedef int int_queue_t[$]; 
int_queue_t dynamic_q; // Declares a queue of integers
```

**Corner Cases & Pitfalls:**
*   **Scope Resolution Operator (::):** `typedef` declarations are static by default. When enclosed within a class, they must be referenced from outside the class using the scope resolution operator (e.g., `packet::color_e c1;`).
*   **Forward Type Declaration:** When modeling UVM components, use `typedef class class_name;` to provide a forward declaration to the compiler, preventing circular dependency errors.

