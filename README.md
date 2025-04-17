# VSDSquadron-FPGA-Mini-Internship-program

### Task 1 - VSDSquadron FPGA Mini - Verilog and PCF Task Guide

#### Model Overview
This task involves analyzing and documenting the provided Verilog code, creating the necessary Physical Constraints File (PCF), and integrating the design with the VSDSquadron FPGA Mini board. The integration should follow the setup and installation procedures outlined in the official datasheet: https://www.vlsisystemdesign.com/wp-content/uploads/2025/01/VSDSquadronFMDatasheet.pdf

---

### Step 1: Understanding the Verilog Code
- Access the Verilog code from the following link --> https://github.com/thesourcerer8/VSDSquadron_FM/blob/main/led_blue/top.v
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

- ### Model Overview
This Verilog module utilizes an internal high-frequency oscillator to operate a frequency counter and manage an RGB LED, while also providing a dedicated test output signal for simplified debugging.

#### Input
hw_clk: Hardware oscillator clock input (currently unused in the module; reserved for optional external clock sources).

#### Outputs -
	- led_red, led_blue, led_green: Output signals used to control the RGB LED.
	- testwire: A debug output connected to the internal frequency counter.

#### Internal Components

- **Internal Oscillator (SB_HFOSC)
	- Function: Generates an internal clock signal to drive the frequency counter.
	- Configuration:
	- CLKHF_DIV = "0b10": Sets clock division factor (binary 2).
	- Control signals:
		1 CLKHFPU = 1'b1: Powers up the oscillator.
	 2 CLKHFEN = 1'b1: Enables the oscillator.
		3 *CLKHF* : Output connected to internal *int_osc* signal

- **Frequency Counter

	- Structure: A 28-bit register named frequency_counter_i.
	- Operation: Increments on every rising edge of int_osc.
	- Debug Use: Bit 5 (frequency_counter_i[5]) is routed to the testwire output.
	- Purpose: Helps verify oscillator activity and monitor timing behavior.
- **RGB LED Driver (SB_RGBA_DRV)
	- Role: Drives the RGB LED outputs, with only the blue channel active.
   
	- Settings:
		1 RGBLEDEN = 1'b1: Enables the LED driver.
		2 RGB0PWM = 1'b0: Red LED set to off/minimum.
		3 RGB1PWM = 1'b0: Green LED set to off/minimum.
		4 RGB2PWM = 1'b1: Blue LED turned on.
		5 CURREN = 1'b1: Enables current control circuitry.
	- Current Configuration: All RGB channels set to "0b000001" (minimal current).

	- Output Mappings:
		1 RGB0 → led_red
		2 RGB1 → led_green
		3 RGB2 → led_blue
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
 #### Significance of Connections
 - **RGB LEDs**: Connected to the FPGA output pins for color control.
 - **hw_clk**: External clock input to provide flexibility in timing.
 - **testwire**: Provides insight into internal signal behavior for debugging purposes.

---

## Step 3: Integrating with the VSDSquadron FPGA Mini Board

### Reviewing the Datasheet
Consult the datasheet of the VSDSquadron FPGA Mini board to review:
- Key features and pin configurations.
- Signal routing and board-level connections.

### Connecting the Board
- Use a USB-C cable to connect the FPGA board to your computer.
- Ensure that the FTDI interface is detected correctly.

### Building and Flashing the Verilog Code
You can find the makefile here: Makefile

#### Steps to Program the FPGA:
1. Run make clean to clear any previous builds.
2. Run make build to compile the design.
3.Run sudo make flash to program the FPGA.
4. Observe the RGB LED behavior to confirm that the FPGA has been successfully programmed.

https://github.com/user-attachments/assets/2fa75078-3658-4e98-a3b3-437508bd3498
 
---

# Task 2: Implementing a UART loopback mechanism
 ## Objective:
 Implement a UART loopback mechanism where transmitted data is immediately received back, facilitating testing of UART functionality
 
 ## Contents:
 ### Step 1: Study the Existing Code
 UART (Universal Asynchronous Receiver-Transmitter) is a hardware communication protocol used for serial communication between devices. It consists of two main data lines: the TX (Transmit) pin and the RX (Receive) pin. Specifically, a UART loopback mechanism is a test or diagnostic mode where data, which is transmitted to the TX (transmit) pin is directly routed back to the RX (receive) pin of the same module. This allows the system to verify that the TX and RX lines function correctly without the need of an external device. The existing code can be found here --> https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/blob/main/UARTexistingcode.v. It is sourced from here --> https://github.com/thesourcerer8/VSDSquadron_FM/tree/main/uart_loopback.
 
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
 
 1. Create the following files ([Makefile](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/blob/main/Makefile_2), [uart_trx](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/blob/main/UARTexistingcode.v) - verilog, [top module](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/blob/main/uart_top.v) - verilog, [pcf file](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/blob/main/UART.pcf)) in a folder under VSDSquadronFM. I have named it --> UART_loopback.
 
 ![Image](https://github.com/user-attachments/assets/ec577eba-3870-43f7-87aa-e97f466ca54a)
 
 2. Then, go to terminal, and enter the commands below. Also connect the board to the VM and verify using the command *lsusb* [if the board is connected, you will see the text "Future Technology Devices International"] .
    > cd
    
    > cd VSDSquadron_FM
    
    > cd UART_loopback
    
    > lsusb 
 
 Then, your screen will look like the screenshot below.
 ![Image](https://github.com/user-attachments/assets/00fbe780-f2d1-4ce8-8141-50569b26b08a)
 
 3. After this, run the commands
    > make build
    
    > sudo make flash
    
 Then, your screen will look like:
 ![Image](https://github.com/user-attachments/assets/2a573e90-e7a8-4203-8ab2-0df9072f4cac)
 
 ### Step 4: Testing and Verification
 
 Testing and Verification
 
 1. For this,we will be using a software known as docklight, which can be downloaded from its website.
 
 2. Open Docklight - and verify that your system (not the VM) is connected to the right communication port - in my case it is COM7 and the default was COM1 - and if not, change it through tools > project settings. Also verify that speed is set to 9600.
 
 3. Then, you may double click on the small blue box below name in send sequences and enter a name, select a format and then type your message. Post this, click "Apply" and then verify that this has entered in send sequences. Then, click the arrow beside the name and verify the result.
 
 ### Step 5: Documentation
 
 Block and Circuit Diagrams
 
 ![image](https://github.com/user-attachments/assets/132b8232-ae8d-48a9-a76c-33c0bad4e661)
 
 ![image](https://github.com/user-attachments/assets/ad0a1fa9-ab68-4cbe-81ae-18d9721a2315)
 
 Testing Results
 
 ![image](https://github.com/user-attachments/assets/47a7f864-2fa9-458e-9380-3b264b0f8904)
 
Video demonstrating Loopback Functionality
 
https://github.com/user-attachments/assets/8a3fa89f-49ba-4c42-a519-6b057bce6cab
  
# Task 3: Developing a UART Transmitter Module
## Objective: 
Develop a UART transmitter module capable of sending serial data from the FPGA to an external device.
## Contents:
### Step 1: Study the Existing Code

A UART transmitter module is a hardware component that enables serial communication from an FPGA to external devices by converting parallel data into sequential bits . This module serves as a fundamental interface for sending data between the FPGA and external devices such as computers, microcontrollers, or other electronic equipment. The code for this module can accessed here --> https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/tree/main/Task_3. It is sourced from this repository --> https://github.com/thesourcerer8/VSDSquadron_FM/tree/main/uart_tx.

### Module Analysis

#### Module Overview
This is a VHDL implementation of an 8N1 UART transmitter designed for FPGAs. It handles asynchronous serial data transmission with the following configuration:
- 8 data bits
- No parity bit
- 1 stop bit

#### State Machine Operation
1. **IDLE State (STATE_IDLE)**
- Maintains TX line high (idle condition)
- Waits for senddata trigger
- Resets txdone flag

2. **STARTTX State (STATE_STARTTX)**
- Transmits start bit (logic low)
- Loads transmission buffer with txbyte
- Immediately transitions to TXING state

3. **TXING State (STATE_TXING)**
- Sends data bits sequentially
- Shifts buffer right for next bit
- Counts transmitted bits (0–7)
- Continues until all 8 bits are sent

4. **TXDONE State (STATE_TXDONE)**
- Sends stop bit (logic high)
- Sets txdone flag
- Returns to IDLE state

### Step 2: Design Documentation

#### Block Diagram

![image](https://github.com/user-attachments/assets/cca7a246-abb8-4d8a-a695-85d04eb5b15a)

#### Circuit Diagram

![image](https://github.com/user-attachments/assets/44ab6bc7-145c-4a0c-87e8-e98966497478)

### Step 3: Implementation

#### Steps to Transmit the Code

1. Create the following [files](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/tree/main/Task_3) in a folder under VSDSquadron_FM.
2. Then, open terminal and through the commands "cd"; "cd VSDSquadron_FM" and "cd uart_transmission" enter the folder "uart_transmission", where you have created the files.
3. Post this, you may verify that the board is connected through "lsusb" command.
4. After this, run "make build" and "sudo make flash".

That is all. The code is transmitted.

### Step 4: Testing and Verification

1. Install, and then open PuTTy.
2. Verify that the correct port is connected through serial communication (COM 7 in my case)
3. Then, check that a series of "D"s are generated and the RGB LED is blinking (switching between red, green and blue) .

### Step 5: Documentation

#### UART Transmission in Operation

https://github.com/user-attachments/assets/1da62013-2543-4feb-b6bd-689d24fed912

#### Block and Circuit Diagram :

![image](https://github.com/user-attachments/assets/cca7a246-abb8-4d8a-a695-85d04eb5b15a)

![image](https://github.com/user-attachments/assets/44ab6bc7-145c-4a0c-87e8-e98966497478)

# Task 4: Implementing a UART Transmitter that Sends Data Based on Sensor Inputs
## Objective:
Implement a UART transmitter that sends data based on sensor inputs, enabling the FPGA to communicate real-time sensor data to an external device.

## Contents:
### Step 1: Study the Existing Code

#### Module Analysis

#### Architecture Overview
The *UART_tx_sense* module implements a complete **UART transmitter** designed specifically for **sensor data transmission**. The architecture consists of three main components:
1. **Data Buffer Management**
2. **UART Protocol Controller**
3. **Transmission Control Logic**

#### Operation Flow
1. **Data Acquisition**
- Sensor data arrives with valid signal assertion
- Module captures data during IDLE state
- 32-bit data buffer stores incoming sensor readings
2. **Transmission Protocol**
- *START*: Generates UART start bit (low)
- *DATA*: Transmits 8 bits sequentially
- *STOP*: Ensures proper termination with high bit
3. **Status Indication**
- *ready* signal indicates ability to accept new data
- *tx_out* provides continuous UART stream
- State transitions ensure reliable data transfer

#### Port Analysis
1. **Clock and Reset**
- *clk*: Drives all sequential operations
- *reset_n*: Active-low asynchronous reset
2. **Data Interface**
- *data*: 32-bit wide input for sensor readings
- *valid*: Handshake signal indicating valid data
3. **UART Interface**
- *tx_out*: Serial output following UART protocol
4. **Status Interface**
- *ready*: Indicates module's ability to accept new data

#### Internal Component Analysis
1. **State Machine Controller**
- Manages transmission protocol states
- Controls data flow through the module
- Ensures proper UART framing
2. **Data Buffer**
- Stores incoming sensor data
- Provides data stability during transmission
- Handles data synchronization
3. **Transmission Controller**
- Manages bit-by-bit transmission
- Controls UART protocol timing
- Handles start/stop bit generation

### Step 2: Design Documentation

#### Block and Circuit Diagram

![image](https://github.com/user-attachments/assets/86d868e5-3f3f-4c62-8c55-901ab252ea13)

![image](https://github.com/user-attachments/assets/34a020a3-91ff-43dd-8fc8-39a092931793)

### Step 3: Implementation

#### Steps to Transmit the Code

1. Create the following [files](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/tree/main/uart_tx_sense) in a folder under VSDSquadron_FM.
2. Then, open terminal and through the commands "cd"; "cd VSDSquadron_FM" and "cd uart_transmission" enter the folder "UART_tx_sense", where you have created the files.
3. Post this, you may verify that the board is connected through "lsusb" command.
4. After this, run "make build" and "sudo make flash".

### Step 4: Testing and Verification

#### Steps of Testing and Verification

1. Open PuTTy.
2. Verify that the correct port is connected through serial communication (COM 7 in my case)
3. Then, check that a series of "D"s are generated and the RGB LED is red.

### Step 5: Documentation

#### Block and Circuit Diagrams:

![image](https://github.com/user-attachments/assets/86d868e5-3f3f-4c62-8c55-901ab252ea13)

![image](https://github.com/user-attachments/assets/e46df768-02aa-4daa-a115-b936e452d363)

#### Video Demonstrating System Transmitting Data
    
https://github.com/user-attachments/assets/beabcf14-6793-4307-8e80-da8231b29e00

> note: Here you cannot see the LED blinking as the time intervals between each 0 and 1 are very tiny
