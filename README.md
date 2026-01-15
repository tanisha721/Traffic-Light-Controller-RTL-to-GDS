# Traffic-Light-Controller-RTL-to-GDS
In this Project i have created a Traffic Light Controller using Verilog Code with the help of testbench to simulate with the waveform anaylsis followed by creating a gatelevel netlist of it then physical design steps such as routing floorplanning and placement on 90nm technology.
## Tools used 
cadance stimulator [waveform generation]
cadance genus [gate level netlist generation]
cadance innovus [physical design]
verilog code [programming language]

## Working and Verilog Code
module tlc (
    input clk,          // Main system clock
    input reset,        // Active-high asynchronous reset
    output reg red,     // Output signal for Red light
    output reg yellow,  // Output signal for Yellow light
    output reg green    // Output signal for Green light
);

    // Internal registers for state tracking and timing
    reg [1:0] state;    // Tracks current light (Red, Green, or Yellow)
    reg [3:0] count;    // 4-bit counter to manage duration of each light

    // State encoding (using parameters for readability)
    parameter RED    = 2'b00,
              GREEN  = 2'b01,
              YELLOW = 2'b10;

    // --- State Transition Logic (Sequential Block) ---
    // This handles the "brain" of the controller: when to switch lights.
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            // Reset to default state: Red light starts the cycle
            state <= RED;
            count <= 0;
        end else begin
            // Increment counter on every clock cycle
            count <= count + 1;
            
            case (state)
                RED: begin
                    // Hold Red for 6 clock cycles (0 to 5)
                    if (count == 5) begin
                        state <= GREEN;
                        count <= 0; // Reset counter for the next light
                    end
                end
                
                GREEN: begin
                    // Hold Green for 6 clock cycles (0 to 5)
                    if (count == 5) begin
                        state <= YELLOW;
                        count <= 0;
                    end
                end
                
                YELLOW: begin
                    // Hold Yellow for 4 clock cycles (0 to 3)
                    if (count == 3) begin
                        state <= RED;
                        count <= 0;
                    end
                end
                
                default: state <= RED; // Fallback safety state
            endcase
        end
    end

    // --- Output Logic (Combinational Block) ---
    // This decodes the current state into actual light signals.
    always @(*) begin
        // Default: All lights off to prevent multiple lights on at once
        red = 0; yellow = 0; green = 0;
        
        case (state)
            RED:    red = 1;    // Turn on Red light
            GREEN:  green = 1;  // Turn on Green light
            YELLOW: yellow = 1; // Turn on Yellow light
        endcase
    end

endmodule
 this code creates an automated cycle that mimics a real-world traffic light. It works by combining three main "human" concepts into digital logic:The Schedule (FSM): The code defines a strict sequence. It can only move from Red $\rightarrow$ Green $\rightarrow$ Yellow, and then back to Red. It uses "States" to remember exactly where it is in that loop at all times.The Stopwatch (Counter): Instead of using a clock on the wall, it counts "pulses" from the chip's heartbeat. It stays on Red and Green for 6 pulses each, but switches Yellow off after only 4 pulses because yellow lights need to be shorter for safety.The Switchboard (Output Logic): This part acts as the physical connection to the bulbs. It looks at the current state and makes sure only one light is "ON" (1) while the others are "OFF" (0), preventing dangerous overlaps.The Panic Button (Reset): For safety, a reset signal acts as a "fail-safe." No matter where the timer is, it immediately forces the system back to Red to stop all traffic and start fresh.

 ## Verilog Testbench
 module tlc_tb;

    // Testbench signals
    reg clk;
    reg reset;
    wire red;
    wire yellow;
    wire green;

    // Instantiate the DUT (Device Under Test)
    traffic_light_controller DUT (
        .clk(clk),
        .reset(reset),
        .red(red),
        .yellow(yellow),
        .green(green)
    );

    // Clock generation: 10 ns period
    always #5 clk = ~clk;

    // Test sequence
    initial begin
        // Initialize signals
        clk = 0;
        reset = 1;

        // Apply reset
        #20;
        reset = 0;

        // Run simulation
        #200;

        // End simulation
        $stop;
    end

    // Monitor output changes
    initial begin
        $monitor("Time=%0t | RED=%b YELLOW=%b GREEN=%b",
                 $time, red, yellow, green);
    end

endmodule

## Cadance sdc file
1. Clock Definition
Defines a 100MHz clock (10ns period) with a 50% duty cycle (rises at 0ns, falls at 5ns)
create_clock -name clk -period 10.0 -waveform {0 5} [get_ports clk]

2. Clock Jitter/Reliability
Accounts for 0.2ns of "uncertainty" (jitter or skew) to give the design some safety margin
set_clock_uncertainty 0.2 [get_clocks clk]

3. Input Delays (Arrival Times)
Tells the tool that data for control and data signals arrives 2ns after the clock edge
This ensures the chip has enough time to "catch" the data before the next beat
set_input_delay 2.0 -clock clk [get_ports {wr_en rd_en}]
set_input_delay 2.0 -clock clk [get_ports din*]

4. Output Delays (Required Times)
Sets a 2ns requirement for data to be ready at the pins before the next clock cycle
set_output_delay 2.0 -clock clk [get_ports dout*]
set_output_delay 2.0 -clock clk [get_ports {empty full}]

5. Electrical Characteristics (Drive Strength)
The clock has "infinite" drive (0 resistance) to prevent the tool from buffering it prematurely
set_drive 0 [get_ports clk]
Standard drive strength for input pins to model real-world signal behavior
set_drive 1 [get_ports {wr_en rd_en}]
set_drive 1 [get_ports din*]

6. Electrical Characteristics (Capacitive Load)
Models the "weight" (capacitance) that the output pins have to push against
set_load 0.1 [get_ports dout*]
set_load 0.05 [get_ports {empty full}]

7. Exceptions
Tells the tool NOT to worry about timing the Reset signal. 
Since reset is asynchronous, it doesn't need to meet a specific 10ns clock deadline.
set_false_path -from [get_ports rst_n]

## Author 
Tanisha Singh
B.tech Electronics and Communication Engineering

