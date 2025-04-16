# VSDSquadron-FPGA-Mini-Internship-program

### Task 1 - VSDSquadron FPGA Mini - Verilog and PCF Task Guide

#### Objective
To understand and document the provided Verilog code, create the necessary PCF file, and integrate the design with the VSDSquadron FPGA Mini board using the provided datasheet. (install tools as explained in datasheet) -  https://www.vlsisystemdesign.com/wp-content/uploads/2025/01/VSDSquadronFMDatasheet.pdf

---

### Step 1: Understanding the Verilog Code
- Access the Verilog code --> Find here https://github.com/thesourcerer8/VSDSquadron_FM/blob/main/led_blue/top.v
 ```bash
 //----------------------------------------------------------------------------
 //                                                                          --
 //                         Module Declaration                               --
 //                                                                          --
 //----------------------------------------------------------------------------
 module top (
   // outputs
   output wire led_red  , // Red
   output wire led_blue , // Blue
   output wire led_green , // Green
   input wire hw_clk,  // Hardware Oscillator, not the internal oscillator
   output wire testwire
 );
 
   wire        int_osc            ;
   reg  [27:0] frequency_counter_i;
 
   assign testwire = frequency_counter_i[5];
 
   always @(posedge int_osc) begin
     frequency_counter_i <= frequency_counter_i + 1'b1;
   end
 
 
 //----------------------------------------------------------------------------
 //                                                                          --
 //                       Counter                                            --
 //                                                                          --
 //----------------------------------------------------------------------------
 
 //----------------------------------------------------------------------------
 //                                                                          --
 //                       Internal Oscillator                                --
 //                                                                          --
 //----------------------------------------------------------------------------
   SB_HFOSC #(.CLKHF_DIV ("0b10")) u_SB_HFOSC ( .CLKHFPU(1'b1), .CLKHFEN(1'b1), .CLKHF(int_osc));
 
 
 //----------------------------------------------------------------------------
 //                                                                          --
 //                       Instantiate RGB primitive                          --
 //                                                                          --
 //----------------------------------------------------------------------------
   SB_RGBA_DRV RGB_DRIVER (
     .RGBLEDEN(1'b1                                            ),
     .RGB0PWM (1'b0), // red
     .RGB1PWM (1'b0), // green
     .RGB2PWM (1'b1), // blue
     .CURREN  (1'b1                                            ),
     .RGB0    (led_red                                       ), //Actual Hardware connection
     .RGB1    (led_green                                       ),
     .RGB2    (led_blue                                        )
   );
   defparam RGB_DRIVER.RGB0_CURRENT = "0b000001";
   defparam RGB_DRIVER.RGB1_CURRENT = "0b000001";
   defparam RGB_DRIVER.RGB2_CURRENT = "0b000001";
 
 endmodule
 ```

- ### Model Review
This Verilog module leverages an internal high-frequency oscillator to drive a frequency counter and control an RGB LED, with a dedicated test signal output for streamlined debugging.

#### Input -
   - `hw_clk`: Hardware oscillator clock input - unused within the module but reserved for optional external clocking.
#### Output -
   - `led_red`, `led_blue`, `led_green`: Signals to control an RGB LED.
   - `testwire`: Debug signal sourced from the internal frequency counter.
 
 #### Internal Components
 - **Internal Oscillator (`SB_HFOSC`)**: Generates the clock signal that drives the frequency counter.
    - Purpose: This generates a stable internal clock signal
    - Configuration: Uses CLKHF_DIV = "0b10" (binary 2) for clock division
    - Control Signals:
        1. *CLKHFPU = 1'b1* : Enables power-up
        2. *CLKHFEN = 1'b1* : Enables oscillator
        3. *CLKHF* : Output connected to internal *int_osc* signal
 
 - **Frequency Counter**: A 28-bit counter that increments with each clock cycle - bit 5 (frequency_counter_i[5]) is used as a test signal output.
    - Implementation: 28-bit register (*frequency_counter_i*)
    - Operation: Increments on every positive edge of *int_osc*
    - Test functionality: Bit 5 is routed to *testwire* for monitoring
    - Purpose: Provides a way to verify oscillator operation and timing

 - **RGB LED Driver (`SB_RGBA_DRV`)**: Controls an RGB LED, with only the blue LED activated (`RGB2PWM = 1'b1`). The current settings are predefined.
    - Configuration:
        1. *RGBLEDEN = 1'b1* : Enables LED operation
        2. *RGB0PWM = 1'b0* : Red LED minimum brightness
        3. *RGB1PWM = 1'b0* : Green LED minimum brightness
        4. *RGB2PWM = 1'b1* : Blue LED maximum brightness
        5. *CURREN = 1'b1* : Enables current control
   
    - Current settings: All LEDs set to "0b000001" (minimum current)
      
    - Output connections:
       1. *RGB0* → *led_red*
       2. *RGB1* → *led_green*
       3. *RGB2* → *led_blue*
 
 ## Step 2: Creating the PCF File
 
 - Access the PCF File --> Find Here [VSDSquadronFM.pcf](https://github.com/thesourcerer8/VSDSquadron_FM/blob/main/led_blue/VSDSquadronFM.pcf)
 
 ### Pin Assignments
 | Signal      | Pin |
 |------------|----|
 | `led_red`  | 39 |
 | `led_blue` | 40 |
 | `led_green`| 41 |
 | `hw_clk`   | 20 |
 | `testwire` | 17 |
 
 Cross-reference these pin assignments with the VSDSquadron FPGA Mini board datasheet to verify correctness.
 [Datasheet](https://www.vlsisystemdesign.com/wp-content/uploads/2025/01/VSDSquadronFMDatasheet.pdf)
 ### Significance of Connections
 - **RGB LEDs**: Connected to FPGA output pins to enable color control.
 - **`hw_clk`**: External clock input for flexibility in timing.
 - **`testwire`**: Allows observation of internal signal behavior for debugging.

https://github.com/user-attachments/assets/2fa75078-3658-4e98-a3b3-437508bd3498

---

## Step 3: Integrating with the VSDSquadron FPGA Mini Board
 
 ### Reviewing the Datasheet
Consult the VSDSquadron FPGA Mini board datasheet to explore:
- Key features and pin configurations
- Signal routing and board-level connections
 
 ### Connecting the Board
 - Use a USB-C cable to connect the FPGA board to your computer.
 - Verify that the FTDI interface is correctly detected.
 
 ### Building and Flashing the Verilog Code
Find the make file here --> [Makefile](https://github.com/thesourcerer8/VSDSquadron_FM/blob/main/led_blue/Makefile)
 
 #### Steps to Program the FPGA:
 1. Run `make clean` to clear previous builds.
 2. Run `make build` to compile the design.
 3. Run `sudo make flash` to program the FPGA.
 4. Observe the RGB LED behavior to confirm successful programming. 
 
---

# Task 2: Implementing a UART loopback mechanism
 ## Objective:
 Implement a UART loopback mechanism where transmitted data is immediately received back, facilitating testing of UART functionality
 
 ## Contents:
 ### Step 1: Study the Existing Code
 UART (Universal Asynchronous Receiver-Transmitter) is a hardware communication protocol used for serial communication between devices. It consists of two main data lines: the TX (Transmit) pin and the RX (Receive) pin. Specifically, a UART loopback mechanism is a test or diagnostic mode where data, which is transmitted to the TX (transmit) pin is directly routed back to the RX (receive) pin of the same module. This allows the system to verify that the TX and RX lines function correctly without the need of an external device. The existing code can be found [here](https://github.com/ojasvi-shah/VSDSquadron-FM-Research-Internship-by-Ojasvi-Shah/blob/main/UARTexistingcode.v). It is sourced from [this repository](https://github.com/thesourcerer8/VSDSquadron_FM/tree/main/uart_loopback). For the analysis of this code, expand or collapse:
 
#### Analysis of Existing Code
 
 ##### Port Analysis:
 The module explains six ports:
 - Three **RGB LED outputs** (led_red, led_blue, led_green)
 - **UART transmit/receive pins** (uarttx, uartrx)
 - **System clock input** (hw_clk)
 
 ##### Internal Component Analysis
 1. **Internal Oscilliator** (SB_HFOSC)
 - Implements a high-frequency oscillator
 - Uses CLKHF_DIV = "0b10" for frequency division
 - Generates internal clock signal (int_osc)
 
 2. **Frequency Counter**
 - 28-bit counter (frequency_counter_i)
 - Increments on every positive edge of internal oscillator
 - Used for timing generation
 
 3. **UART Loopback**
 - Direct connection between transmit and receive pins
 - Echoes back any received UART data immediately
 
 4. **RGB LED Driver** (SB_RGBA_DRV)
 - Controls three RGB channels
 - Uses PWM (Pulse Width Modulation) for brightness control
 - Current settings configured for each channel
 - Maps UART input directly to LED intensity
 
 ##### Operation Analysis
 1. **UART Input Processing**
 - Received UART data appears on *uartrx* pin
 - Data is immediately looped back out through *uarttx*
 - Same data drives all RGB channels simultaneously
 2. **LED Control**
 - RGB driver converts UART signal to PWM output
 - All LEDs respond identically to input signal
 - Current limiting set to minimum (0b000001) for each channel
 3. **Timing Generation**
 - Internal oscillator provides clock reference
 - Frequency counter generates timing signals
 - Used for PWM generation and LED control
 
 ### Step 2: Design Documentation
 
 #### Block Diagram Illustrating the UART Loopback Architecture.

 ![image](https://github.com/user-attachments/assets/3447a27b-59fe-49e7-9c73-9a85f39c8a7d)

 #### Detailed Circuit Diagram showing Connections between the FPGA and any Peripheral Devices used.
 ![image](https://github.com/user-attachments/assets/af77ea52-38ef-415a-a724-43abf43bc207)
 
 ### Step 3: Implementation
 
 Steps to Transmit Code to FPGA Board
 
 1. Create the following files ([Makefile](https://github.com/ojasvi-shah/VSDSquadron-FM-Research-Internship-by-Ojasvi-Shah/blob/main/Makefile2), [uart_trx](https://github.com/ojasvi-shah/VSDSquadron-FM-Research-Internship-by-Ojasvi-Shah/blob/main/UARTexistingcode.v) - verilog, [top module](https://github.com/ojasvi-shah/VSDSquadron-FM-Research-Internship-by-Ojasvi-Shah/blob/main/uart-top.v) - verilog, [pcf file](https://github.com/ojasvi-shah/VSDSquadron-FM-Research-Internship-by-Ojasvi-Shah/blob/main/UART.pcf)) in a folder under VSDSquadronFM. In this case, I have named it *uart_loopback*.
 
 ![image](https://github.com/user-attachments/assets/e1ad1ffd-def9-4119-ab07-b40db477ef51)
 
 2. Then, go to terminal, and enter the commands below. Also connect the board to the VM and verify using the command *lsusb* [if the board is connected, you will see the text "Future Technology Devices International"] .
    > cd
    > cd VSDSquadron_FM
    > cd uart_loopback
 
 Then, your screen will look like the screenshot below.
 ![image](https://github.com/user-attachments/assets/87f284a7-a44a-4e70-a125-327b6fa15a59)
 
 3. After this, run the commands "make build", and "sudo make flash". Then, your screen will look like:
 ![image](https://github.com/user-attachments/assets/91c7c341-c19a-4add-a5fb-5f8529cc54eb)
 
 That is it. You have successfully finished transmitting the code.

 ### Step 4: Testing and Verification
 
 Testing and Verification
 
 1. For this,we will be using a software known as docklight, which can be downloaded from its website.
 
 2. Open Docklight - and verify that your system (not the VM) is connected to the right communication port - in my case it is COM7 and the default was COM1 - and if not, change it through tools > project settings. Also verify that speed is set to 9600.
 
 ![image](https://github.com/user-attachments/assets/467c3207-0137-45a8-8a53-1e1103269d2b)
 
 3. Then, you may double click on the small blue box below name in send sequences and enter a name, select a format and then type your message. Post this, click "Apply" and then verify that this has entered in send sequences. Then, click the arrow beside the name and verify the result is as follows:
 
 ![image](https://github.com/user-attachments/assets/47a7f864-2fa9-458e-9380-3b264b0f8904)
 
 ### Step 5: Documentation
 
 Block and Circuit Diagrams
 
 ![image](https://github.com/user-attachments/assets/132b8232-ae8d-48a9-a76c-33c0bad4e661)
 
 ![image](https://github.com/user-attachments/assets/ad0a1fa9-ab68-4cbe-81ae-18d9721a2315)
 
 Testing Results
 
 ![image](https://github.com/user-attachments/assets/47a7f864-2fa9-458e-9380-3b264b0f8904)
 
Video demonstrating Loopback Functionality
 
 https://github.com/user-attachments/assets/443cf339-d2ac-45a5-885c-c1fdc74a46ed
  
