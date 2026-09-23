1.Module nestings are illegal instead instances(copies) are allowed

eg :Not alowed

&#x20;    

&#x20;    module xxx();

&#x09;module yyy();



&#x09;-------------



&#x09;endmodule

&#x20;    endmodule 



Modules communicate through ports



2\. Default values



* &#x09;reg - x (unknown - empty storage box ) -- does not infer a FlipFlop
* &#x09;wire - z (high impedance - unconnected net)





3.The difference between the operators



== and ===

!= and !==



Left - Logical equivalence check 

Right - case equivalence check ( sees for exact match of each bits - answers in 0/1 only)





4.Non-synthesizable Verilog constructs (few to be remembered)



* &#x09;initial
* &#x09;UDPs
* &#x09;Fork and join



[Refer Here](https://asic-soc.blogspot.com/2013/06/synthesizable-and-non-synthesizable.html#google_vignette)



5.Operator that maps to specific hardwares



&#x09;Conditional operator - MUX

&#x09;always @( posedge or negedge ) - a storage element(Flip Flop)



6\. Generic rule



Blocking assignments (=) for combo logic

Non-blocking assignments (<=) for sequential logic



7.Challenge 1 : Without using the "always" keyword illustrate its behaviour using "initial" construct



always @()

   begin

      -----

   end	



initial

    forever 

       begin

         ------

       end





Note : always is synthesizable but forever is not



8.Chanllenge 2 : Having a full adder module constructed



To have a 4-bit FA: instantiate 4 copies

To have a 8-bit FA: instantiate 8 copies



what if the adder is of 32 bits / 64 bits ?



use : generate , endgenerate construct (synthesizable)



9.Leaving output port unconnected - Not an error

&#x20; Leaving input port unconnected - Error ( since 'Z' induces unexpected behaviour)



10.wire A ;

&#x20;  input wire A ;   statements mean the same



Eg: 

wire B ;

assign B = sel;	wire B = sel;	assign wire B = sel;



Note : multiple assignments to a single net can be resolved using wand/wor else it leads to unknown value(X)

&#x20;      multiple assignments to a reg eventually evaluates to 0/1 .





11\. SYNTHESIS :

&#x09;The tool infers logic from the HDL source ,maps the inferred logic to the technology library macros and optimizes the circuit to meet constraints



12\. Challenge 3: What does the following Verilog snippet maps to ?



always@(posedge clk)

begin

&#x09;if(enb)

&#x09;  q<= d;

end



a FF  or a Latch ? --> A latch : what if the 'enb' holds 0 --> infers a hidden storage(latch)



13\. casex and casez



casex - treats x and z as wildcard ( x in expression prevents match)

casez - treats z as wildcard (or ?)



14.Lets say we have declared a struct of mixed variable types 

Eg:



typedef struct 

{

&#x20;real frequency ;

&#x20;int cycle\_count ;

&#x20;logic fifo\_full ;

} timing\_info ;



when we try to initialize an entire struct to xero

we do : 



timing\_info time\_values;

time\_values = '0; ------- ERROR



it should be : time\_values = '{default:0}; -- structure assignment pattern (best practice)

Reason : real / int assignments may vary 

0 in real - 0.0

0 in int - 0

type problems can be avoided



15\. output logic cmd\[2] is equivalent to output logic cmd\[0:1] --unpacked array containing two 1-bit logic elements

&#x20;   output logic \[1:0] cmd - two bit vector



&#x20;







































