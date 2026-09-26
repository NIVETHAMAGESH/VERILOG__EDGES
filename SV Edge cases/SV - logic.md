# FRONTEND : Specification - Architecture - Behavioural_model - RTL - Verification -Synthesis(GLS) - DFT->
# BACKEND  : ->Floorplanning - Placement - CTS - Routing - STA - PD - DRC - LVS - GDSII


## LOGIC:
>unsigned - 4 state (0,1,x,z) 

>Acts as reg as well as net/wire

>a logic variable cannot be driven by multiple drivers such as when you are modeling a bidirectional bus.

## Corner case:
### case 1:
```systemverilog
logic a;
always_ff @(posedge clk)
    a <= x & y;
assign a = x | y;
```
Here:
`always_ff` drives a
`assign` also drives a
trying to drive the same logic from two places is illegal.

### case 2:Diff clocks:
```systemverilog
logic a;
always_ff @(posedge clk)
    a <= x & y;
always_ff @(posedge clk2)
    a <= x | y;
```
trying to drive the same logic from two places though the clk differs is illegal.


Assume `clk1` and `clk2` are completely different and asynchronous clocks.
## 1. What happens to the already stored value?
 initially:
```text
a = 0
```
Then `clk1` has a rising edge:
```text
clk1 ↑
a <= x
```
If:
```text
x = 10
```
then:
```text
a = 10
```
The value remains stored:
```text
a = 10
```
It does **not** disappear just because time passes.
Now suppose `clk2` has a rising edge:

```text
clk2 ↑
a <= y
```

If:
```text
y = 20
```

then conceptually:
```text
a = 20
```

So the intuitive timeline is:

```text
              clk1 edge
                  │
                  ▼
             a = 10
                  │
                  │ holds 10
                  │
              clk2 edge
                  │
                  ▼
             a = 20
```

### Important intuition

The previously stored value is not automatically erased when another clock arrives.

A new clock event would cause a new update **if the hardware architecture actually supported that behavior**.

## 2. Then why can't we simply write two `always_ff` blocks?

The problem is that a normal flip-flop does not have two unrelated clocks.

A conventional flip-flop looks like:

```text
             ┌──────────┐
D ──────────►│          │
             │    FF    │──── Q
CLK ────────►│          │
             └──────────┘
```

It has:

- one data input
- one clock input
- one output

But the RTL above is effectively asking for:

```text
             ┌──────────┐
x ──────────►│          │
clk1 ───────►│          │
             │    ???   │──── a
clk2 ───────►│          │
y ──────────►│          │
             └──────────┘
```

That is **not an ordinary flip-flop**.

Therefore:

```systemverilog
always_ff @(posedge clk1)
    a <= x;

always_ff @(posedge clk2)
    a <= y;
```
is not a valid way to describe `one normal sequential storage element`.

`always_ff` is intended to enforce the idea that a variable has a single sequential/procedural driver.

## 3. What if we implement it using two flip-flops?
```systemverilog
logic a1;
logic a2;

always_ff @(posedge clk1)
    a1 <= x;

always_ff @(posedge clk2)
    a2 <= y;
```

Now the hardware is clear:

```text
                 ┌───────┐
x ──────────────►│  FF1  │──── q1
clk1 ───────────►│       │
                 └───────┘


                 ┌───────┐
y ──────────────►│  FF2  │──── q2
clk2 ───────────►│       │
                 └───────┘
```

For example:

```text
a1 = 10
a2 = 20
```

Now if we want a single output `a`, we need to explicitly decide which value should appear at `a`.

For example:

```systemverilog
assign a = select ? a1 : a2;
```

Hardware:

```text
a1 ──┐
     │
     ├── MUX ───► a
     │
a2 ──┘
      ▲
      │
    select
```

Now the hardware behavior is well-defined.

## 4. The important distinction: `logic` is not a physical flip-flop
```systemverilog
logic a;
```
It does NOT by itself mean:

```text
        ┌──────┐
        │  FF  │
        └──────┘
```
The surrounding RTL determines what hardware is inferred.

> `logic` is a data type.  
> `always_ff`, `always_comb`, `assign`, module connections, etc. determine how it is driven and what hardware may be inferred.

## Clock-Domain Crossing (CDC)

If information needs to move from the `clk1` domain to the `clk2` domain, we normally use a CDC mechanism.

For example, conceptually:

```text
             CLK1 DOMAIN                    CLK2 DOMAIN

            ┌─────────┐                   ┌─────────┐
x ─────────►│   FF    │── a_clk1 ──CDC──► │   FF    │──► a_clk2
            └─────────┘                   └─────────┘
                 ▲                              ▲
               clk1                           clk2
```

Depending on the signal and protocol, CDC can involve:

- synchronizer flip-flops
- handshake protocols
- asynchronous FIFOs
- pulse synchronization
- clock-domain-specific control logic

The exact solution depends on what information is being transferred.

## suppose :
```systemverilog
module block1(input logic clk1);
    logic a;

    always_ff @(posedge clk1)
        a <= x;
endmodule
```
```systemverilog
module block2(input logic clk2);
    logic a;

    always_ff @(posedge clk2)
        a <= y;
endmodule
```
>`block1.a` `block2.a` are completely different 



>Multiple `assign statements to different bits is legal`
>Generally legal since `different parts of the vector` are driven.
```systemverilog
logic [3:0] a;
assign a[1:0] = x;
assign a[3:2] = y;
```
## ILLEGAL drives for a single logic 
1. Two always_comb blocks → same logic
2. always_comb + always_ff
3. always_ff + always_ff, same clock
4. Two always_ff blocks, different clocks
5. always_ff + continuous assignment
----