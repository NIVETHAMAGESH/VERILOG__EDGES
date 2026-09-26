FRONTEND : spec - arch - behavioural\_model - rtl - verif-syn(GLS) - dft->

BACKEND  : ->fp - Placement- CTS - routing - STA - PD - DRC - LVS - GDSII





LOGIC:

unsigned - 4 state (0,1,x,z)

acts as reg as well as net/wire

a logic variable cannot be driven by multiple drivers such as when you are modeling a bidirectional bus.



Corner case:

```systemverilog

**logic a;**



**always\_ff @(posedge clk)**

&#x20;   **a <= x \& y;**



**assign a = x | y;**

**```**

Here:



always\_ff drives a

assign also drives a



So trying to drive the same logic from two places.



Diff clocks:

```systemverilog

logic a;



always\_ff @(posedge clk)

&#x20;   a <= x \& y;



always\_ff @(posedge clk2)

&#x20;   a <= x | y;

```

So trying to drive the same logic from two places.though the clk differs is illegal.

















module block1(input logic clk1);

&#x20;   logic a;



&#x20;   always\_ff @(posedge clk1)

&#x20;       a <= x;

endmodule





module block2(input logic clk2);

&#x20;   logic a;



&#x20;   always\_ff @(posedge clk2)

&#x20;       a <= y;

endmodule



block1.a

block2.a are completely different 







Multiple assign statements to different bits



This one is interesting:



logic \[3:0] a;



assign a\[1:0] = x;

assign a\[3:2] = y;



Generally okay because you're driving different parts of the vector.



1\. Two always\_comb blocks → same logic

2\. always\_comb + always\_ff

3\. always\_ff + always\_ff, same clock

4\. Two always\_ff blocks, different clocks

5\. always\_ff + continuous assignment





























Within TB - talk using - mailboxes

btw TB and Design - talk using - Interfaces 



typedef <type> <var\_t>

Eg:



typedef bit\[3:0] word\_t;

word\_t address\_word;

word\_t data\_word;



now address\_word and data\_word will have the type of bit\[3:0]



typedef - can be used in places where we have mix of data types / certain data should have some defined types or values



Enum is more similar to struct 



comparison:





