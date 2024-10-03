### Limitation of `timescale directive 
* we know tht `timesacle is compiler directive used to spevify time unit and time precision for simulation
* timescale directive can result in file order dependency problems
* syntax: `timescale <time_unit>/<time_precision>
* consider 3 files/modules of verilog,
  module A has 10ns/1ps , moudle B has no timescale definition and module C has 
  100ns/1ns, delays are multiplied by time unit , i.e in module A the value changes 
  from 0 to 1 at 10ns , n module c would change from 0 to 1 at 100ns.
  suppose ii compile module A first and then i compile module B , B is going to pick 
  timescale of module A
* The default timescale is 
      `timescale 1ns/1ps
* To fix this issue , we have to make sure every module instance taking part in 
 simulation have timesaacle defined, suppose one module has timescale defined , then 
 other moduled shud also have timescale defined, if none of the module has a 
 timesacle then fine.

      `timescale 1ns/1ps
      module A;
      initial begin
      $display("Module 1: Starting simulation...");
      #10;  // Delay of 10ns
      $display("Module 1: 10 nanoseconds have passed.");
      end
      endmodule


// no timescale
  
    module B;
    initial begin
    $display("Module 2: Starting simulation...");
    #5;  // Delay of 5 time units (50ps)
    $display("Module 2: 5 time units have passed (50 picoseconds).");
    end
    endmodule


    `timescale 100ns/10ps
    module C;
    initial begin
    $display("Module 3: Starting simulation...");
    #1;  // Delay of 1 time unit (100ns)
    $display("Module 3: 1 time unit has passed (100 nanoseconds).");
    end
    endmodule

### Time scale system tasks
* suppose these 3 modules are instantaited oon a top module , then there might be 
 confusion, $time is used to get the simulation runtime , when we display $display , 
 confusion coz $time displays only time , and not the string (ns,ps), how to know 
 whether it is ps or ns , coz some modules have no timescale defined (module B ) has 
 no definition it can be ps but we dk what it is exactly.
* $timeformat(units_number,prevcision_num,suffix_string,minimum_feild_width);
  
* units_number:Specifies the time unit for displaying time values.
It determines the base unit of time (e.g., nanoseconds, microseconds) that you want 
to use for output. The value is defined in time units such as:ps, ns,us,ms,s.

* precision_num:Defines the precision of the displayed time value.
This parameter specifies how many digits after the decimal point will be shown. For 
 example, if you set it to 2, the output will show two digits of precision.

* suffix_string:
An optional string that can be added to the end of the displayed time value.
This can be useful for adding a specific label or unit suffix to clarify the time unit being displayed.(ns,ps,ms,s)

* minimum_field_width: minimum feild character length required to display time 
value and string ,Specifies the minimum width of the output field for the timevalue.

Example:
Here’s an example demonstrating the usage of $timeformat in a Verilog module.

     module timeformat_example;
    initial begin
    // Set the time format
    $timeformat(2, 3, " ns", 10);  // Display time in nanoseconds, 3 decimal 
    places, with a suffix " ns", and minimum field width of 10.
     // Display current time in the specified format
     $display("Current time: %t", $time);
    #10; // Wait for 10 nanoseconds
    $display("After 10 ns: %t", $time);
    #5;  // Wait for another 5 nanoseconds
    $display("After another 5 ns: %t", $time);
    end
    endmodule


* Explanation of the Example:
Setting Time Format: The line $timeformat(2, 3, " ns", 10); sets the following:
Units: 2 (nanoseconds)
Precision: 3 (three digits after the decimal point)
Suffix: " ns" (adds the suffix " ns" after the time value)
Minimum Width: 10 (ensures the output is at least 10 characters wide)
* $time depends on time precison
  example:
  
      `timescale 1ns/1ps;
       module test_dut;
      time time_var;
      always
      #10 time_var=$time;
      initial
      begin
      $monitor("change in time: %t",time_var);
      #100 $finish;
      end
      endmodule
  
 now output is 0,10,20,30...

 instead  define a $timeformat 
 
 how?
 
    module unifomr_u();
    initial
    $timeformat(-9,1,"ps",4);
    test_dut dut():
    endmodule

 ![WhatsApp Image 2024-10-03 at 13 23 22_49a19f52](https://github.com/user-attachments/assets/3a3921ef-e115-4d87-bfc8-9aeba7e24a8f)
 so after this output is
change in time:    0.0 ps
change in time:   100.0 ps
change in time:   200.0 ps
change in time:   300.0 ps
change in time:   400.0 ps
change in time:   500.0 ps
change in time:   600.0 ps
change in time:   700.0 ps
change in time:   800.0 ps
change in time:   900.0 ps

 
### Local param or derived parameters
 * "localparam" is used to prevent values from values from being directly 
 overwritten from outside the modules.
* localparam is a keyword used to declare constants that are local to a specific 
 module. 

examples:- 

    module ram1#(parameter ASIZE=10,DSIZE=1)
    (inout {DSIZE-1:0]data,input [ASIZE-1:0]addr,inputen,rw_n);
    //memory depth is 2**ASIZE
    localparam MEM_DEPTH =2**10;
     ....
    endmodule
* here MEM_DEPTH is protected from incorrect settings by placing the MEM_DEPTH in a 
 localparam declaration.


### Generate block and procedural continous assignments
* Generate blocks provide control over creation of many types of module items. 
 Module items are  moduke_instance,continous assignment,procedural block,task 
 defiinition, function definition.
* added in 2001
  ### Generate block
  * "genvar" is an integer variable which must be a positive value.they may only be 
 used within a generate block, they may be declared inisde or outside of genrate 
 block, genvar variables oonly have a value dueing elaboration and do not exist 
 during simulation.
* used for creating multiple instances of modules or blocks of code conditionally or iteratively based on parameters.
* 3 types :- genrate for loop,if generate,if-else genrate, genrate case blocks
* many instances can be created using generate ;
* example

          module full_adder (
        input a,       // First input
       input b,       // Second input
      input cin,     // Carry input
      output sum,    // Sum output
       output cout    // Carry output
       );
      assign sum = a ^ b ^ cin;      // Sum calculation
      assign cout = (a & b) | (cin & (a ^ b)); // Carry out calculation
      endmodule

      // n-Bit Adder Module Using Generate Block
      module n_bit_adder #(parameter N = 4) (  // Parameter N defines the number of 
      bits
      input [N-1:0] A,        // N-bit input A
      input [N-1:0] B,        // N-bit input B
      input Cin,              // Carry input
      output [N-1:0] Sum,     // N-bit Sum output
      output Cout             // Carry output
      );
      wire [N-1:0] carry;      // Internal carry wires
      genvar i;  // Declare a genvar variable for the loop
      generate
          for (i = 0; i < N; i = i + 1) begin : adder_gen
            if (i == 0) begin
                // First adder, take the external carry input
                full_adder fa (
                    .a(A[i]),
                    .b(B[i]),
                    .cin(Cin),
                    .sum(Sum[i]),
                    .cout(carry[i])
                );
            end else if (i < N - 1) begin
                // Intermediate adders, take the carry from the previous adder
                full_adder fa (
                    .a(A[i]),
                    .b(B[i]),
                    .cin(carry[i - 1]),
                    .sum(Sum[i]),
                    .cout(carry[i])
                );
            end else begin
                // Last adder, produce the final carry output
                full_adder fa (
                    .a(A[i]),
                    .b(B[i]),
                    .cin(carry[i - 1]),
                    .sum(Sum[i]),
                    .cout(Cout)
                );
            end
        end
      endgenerate
      endmodule


or
  
      module n_bit_adder #(parameter N = 4) (  // Parameter N defines the number of 
      bits
      input [N-1:0] A,        // N-bit input A
      input [N-1:0] B,        // N-bit input B
      input Cin,              // Carry input
      output [N-1:0] Sum,     // N-bit Sum output
      output Cout             // Carry output
      );
      wire [N-1:0] carry;      // Internal carry wires
      genvar i;  // Declare a genvar variable for the loop
      generate
        for (i = 0; i < N; i = i + 1) begin : adder_gen
            `FULL_ADDER_INSTANCE(i);  // Use the macro to instantiate the full adder
        end
      endgenerate
      endmodule

* in the same way write "genrate if-else"

### Procedural continous assignments
* procedural continous assignments are procedural statements that allow expressions 
 to be driven on to a reg type variable continously
* in this type of assignment assign ,deassign,force and relase
* assign: The assign procedural continous assignment statement shlal overide all 
 procedural assignments to a variable.
* deasssign: The deassign procedural statement shall end a procedural continous 
 assignment to a variable
* force: A force statement to a variable shall ovveride shaall override a 
 procedural assignmney or procedural continous assignmnent that takes place on the 
variable until a release procedural statement is executed on the variable.
Used to override the normal operation of a signal by forcing it to a specific value 
 regardless of other drivers.
* releaase: A relaease statement is used to release the value of the variable tht 
 was forced with a value.

#### Self-checking testbench and automatic task
* in verilog the testbench u r writeing are called linear testbench, genrate 
 stimulus,  drive stimulus and monitoring output.
* A self checking testbench is an automated/inntelligent testbench in which the DUT 
 outputs are captured and compared with their reference values in order to verify 
 the design at the command mode.( also samples and compares values are additional)
* The environment doesn't involve much interaction of the user with the waveform 
 output

example : 
module normal_tb;
    reg clk;
    reg D;
    wire Q;
    // DUT instantiation
    dff dut (
        .D(D),
        .clk(clk),
        .Q(Q)
    );
    initial begin
        clk = 0;
        // Stimulus generation
        D = 0;
        #10; D = 1; #10; D = 0; #10; D = 1;
        // Observing Q
        $monitor("At time %t, D = %b, Q = %b", $time, D, Q);
    end
 always #5 clk = ~clk; // Clock generation
endmodule


     module self_checking_tb;
    reg clk;
    reg D;
    wire Q;
    // DUT instantiation
    dff dut (
        .D(D),
        .clk(clk),
        .Q(Q)
    );
         initial begin
        clk = 0;
        // Initialize input and apply reset
        D = 0; #10; 
        check(0); // Check expected output
        D = 1; #10; 
        check(1); // Check expected output
        D = 0; #10; 
        check(0); // Check expected output
        D = 1; #10; 
        check(1); // Check expected output
        $finish;
    end
    always #5 clk = ~clk; // Clock generation
    task check(input expected);
        #1; // Wait for a delta cycle
        if (Q !== expected) 
            $display("Test failed! Expected Q = %b, but got Q = %b", expected, Q);
        else 
            $display("Test passed! D = %b, Q = %b", D, Q);
    endtask
    endmodule


### Automatic task
* static tasks: All the declared arguments are statically allocated, THese items 
are statically allocated. These items shall be shared across all uses of the task 
 executing concurrently.
* Automatic task: All the declarred arguments are dynmaically allocated for each 
 con-current invocation. ex: task automatic .....endtask
* Each time task is called new stack/memory  is allocated in automatic task


### Named events and stratified event queue
* Named events: it is a new data type in addition to nets and register.
* Features od named events:
   1. It shall not hold any data
   2. it can be made to occur any instant of time
   3. It has no time duration
* Process synchronization is done using named events.
* event declaration: event variable;
* event ocntrolled statement: @(variable) begin    end
* Event triggering statemnt : ->variable;  event can be triggered at any time
* example:  process synchronization
                  
      module named_event_example;
        // Declare a named event       
           event my_event;
      // Process 1: Trigger the event
       initial begin
       // Wait for some time before triggering the event
        #10;
       -> my_event;  // Trigger the named event
      $display("my_event triggered at time %t", $time);
      end
      // Process 2: Wait for the event to be triggered
      initial begin
       // Wait for the named event to be triggered
       wait(my_event);
      $display("my_event received at time %t", $time);
      end
      endmodule
### Verilog stratified event queue
 * Every change in a value of a net or variable in the ckt being simulated is 
 constidered an event, event can occur at different times.
  ![image](https://github.com/user-attachments/assets/fc7a4b9d-0487-457d-b44f-20efd4ca59ee)

### Code coverage , how much of the rtl code is covered
* Code coverage is the metric genrated auomatically from design source in RTL or 
 gates,While a high level of code coverage is required by most verification plans, 
 it does not necssarily indicate correctness of your design.
* It only measure how often certain aspects of the source are exercised while 
 running a suite of tests.
* Missing code coverage is usally an idnication of 2 things either unused code or 
 holes in the tests. Code coverage is tool dependent.
* code coverage types:
  1. Stateemnt coverage
  2. branch coverage
  3. conditioon coverage
  4. expression coverage
  5. toggle coverage
  6. FSM coverage
* Statemnet coverage: The metric for statemnt coverage is the count of 
 execution of statements during simulation. It includes Blocking,NBA,Continuous 
 assignments, %statement coverage=(hits/bins)x100;
* Branch coverage: The metric for branch coverage is the count of execution of the 
 branches like "if","case" and ternary operator during simulation.
 if branch: it includes true branch and false branch, if else is missing , a hidden 
 branch"all false" is added.
 case branch: it includes case items as branches
 Branch coverage= (hits/bins)x100
* Condition Coverage: The metric for condition coverage is the count of descision 
made in the branches "if" && "ternary" using expressions during simulation.
Branch Coverage ensures that all decision outcomes are executed.
Condition Coverage ensures that every condition within a decision has been tested 
in both its true and false states.
Condition coverage worls on the principple of UDP(user defined primitive 
 algorithm) && FEC(focus exoression coverage) based algorithm.
If expression in if else , then it is condition coverage, ie, else if(a===h) , this 
 is condition coverage
* Expression Coverage: The metruc for expression coverage is the count of activity 
 of expressions on the right hand side of assignment statemnts. It works on the UDP 
 and FEC based algotithm, Expression not supported for vectors.
* Toggle coverage: chcks whether every bit has changed from 0 to 1 or 1 to 0, 
Toggle coverage is the baility to count and collect changes of state on specified 
 nodes, standard toggle 1 to 0 , 0 to 1 , extended coverage 0,1,z
* FSM Coverage: FSM state-counts if each state is covered at least once. FSM 
 Transitions-counts if all possible transitions are covered including transitions 
 too idle.





 
