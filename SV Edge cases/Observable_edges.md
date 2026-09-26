## 1. Module nestings are illegal instead instances(copies) are allowed

### NOT ALLOWED:

```verilog
module xxx();
    module yyy();
--------------------
    endmodule
endmodule 
```

> Modules communicate through `ports`

## 2. Default values

`reg` - x (unknown - empty storage box ) -- does not infer a FlipFlop

`wire` - z (high impedance - unconnected net)

## 3. The difference between the operators

`==` and `===`

`!=` and `!==`
>Left - Logical equivalence check 

>Right - case equivalence check ( sees for exact match of each bits - answers in 0/1 only)

## 4. Non-synthesizable Verilog constructs (few to be remembered)
*initial*

*UDPs*

*Fork and join*
>[Refer Here](https://asic-soc.blogspot.com/2013/06/synthesizable-and-non-synthesizable.html#google_vignette)

## 5. Operator that maps to specific hardwares
>Conditional operator - MUX

>always @( posedge or negedge ) - a storage element(Flip Flop)



## 6. Generic rule
Blocking assignments (=) for combo logic
Non-blocking assignments (<=) for sequential logic

## 7. Challenge 1 : Without using the "always" keyword illustrate its behaviour using "initial" construct

```verilog
always @()
   begin
     -----
   end	
```
```verilog
initial
    forever 
       begin
         ------
       end
```
Note :

`always` is `synthesizable` 

`forever` is `non-synthesizable`

## 8. Chanllenge 2 : Having a `full adder` module constructed
To have a 4-bit FA: instantiate 4 copies

To have a 8-bit FA: instantiate 8 copies

what if the adder is of 32 bits / 64 bits ?

>use : generate , endgenerate construct (synthesizable)

## 9. Ports connectivity

Leaving `output port` unconnected - Not an error

Leaving `input port` unconnected - Error ( since 'Z' induces unexpected behaviour)

## 10.
```verilog
wire A ;
input wire A ;   //statements mean the same
```
Eg: 
```verilog
wire B ;
assign B = sel;
wire B = sel;	
assign wire B = sel;
```
Note :

 `multiple assignments to a single net` can be resolved using `wand`/`wor` else it leads to `unknown value(X)`

`multiple assignments to a reg` eventually evaluates to `0/1` 


## 11. SYNTHESIS : The tool infers `logic from the HDL` source ,`maps` the inferred logic to the `technology library macros` and `optimizes` the circuit to meet `constraints`

## 12. Challenge 3: What does the following Verilog snippet maps to ?


```verilog
always@(posedge clk)
begin
    if(enb)
         q<= d;
end
```
>a FF  or a Latch ? --> A FlipFlop

Why?
The key is:

>`@(posedge clk)`

`posedge clk` means the block can execute only at the rising edge of the clock --> essentially a FF



## 13. casex and casez
casex - treats x and z as wildcard ( x in expression prevents match)

casez - treats z as wildcard (or ?)

## 14. Struct initialization
Lets say we have declared a struct of mixed variable types 

Eg:
```systemverilog
typedef struct 
{
real frequency ;
int cycle_count ;
logic fifo_full ;
} timing_info ;
```

`timing_info` - struct data type 

when we try to initialize an entire struct to xero

`ERROR`
```systemverilog
timing_info time_values;
time_values = '0;
```

it should be : `time_values = '{default:0};` -- structure assignment pattern (best practice)

Reason : real / int assignments may vary

0 in real - 0.0

0 in int - 0
>type problems can be avoided



## 15. Vector and Array difference
 `output logic cmd[2`] is equivalent to `output logic cmd[0:1]`
 --`unpacked array` containing two 1-bit logic elements

whereas
output logic `[1:0] cmd` - `two bit vector`


