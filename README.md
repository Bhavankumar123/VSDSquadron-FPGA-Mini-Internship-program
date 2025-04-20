# VSDSquadron-FPGA-Mini-Internship-program

Before diving in, once you install Virtual box and set it up and upload your vdi, do this --> click on your FPGA vdi once --> Go to settings --> Go to general --> click Advanced --> make S̲hared Clipboard and Dr̲ag'n'drop bidirectional

![Image](https://github.com/user-attachments/assets/778feec6-99f6-4fd1-b2dc-f1e090153b98)

This allows you to copy and paste anything from the host pc to vdi and vise versa which might help you save a lot of time in the future activities, Trust me I know!!

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
2. Then, open terminal and through the commands "cd"; "cd VSDSquadron_FM" and "uart_tx_sense", where you have created the files.
3. Post this, you may verify that the board is connected through "lsusb" command.
4. After this, run "make build" and "sudo make flash".

That is all. The code is transmitted.

### Step 4: Testing and Verification

1. Install, and then open PuTTy.
2. Verify that the correct port is connected through serial communication (COM 6 in my case)
3. Then, check that a series of "D"s are generated and the RGB LED is blinking (switching between red, green and blue) .

### Step 5: Documentation

#### UART Transmission in Operation

https://github.com/user-attachments/assets/c0ae2d22-7b22-439d-a75a-ba3b6db8431a

#### Block and Circuit Diagram :

![image](https://github.com/user-attachments/assets/cca7a246-abb8-4d8a-a695-85d04eb5b15a)

![image](https://github.com/user-attachments/assets/44ab6bc7-145c-4a0c-87e8-e98966497478)

# Task 4: Implementing a UART Transmitter that Sends Data Based on Sensor Inputs
## Objective:
Implement a UART transmitter that sends data based on sensor inputs, enabling the FPGA to communicate real-time sensor data to an external device.

## Contents
### Step 1: Study the Existing Code
#### Module Analysis
#### Architecture Overview
The uart_tx_sense module implements a fully functional UART transmitter tailored for transmitting sensor data. It is structured around three core architectural blocks:
--> Data Buffer Management
--> UART Protocol Controller
--> Transmission Control Logic

#### Operation Flow
1. **Data Acquisition**
- Captures sensor data when valid signal is asserted
- ccepts data during the IDLE state
- Stores readings in a 32-bit internal buffer

2. **Transmission Protocol**
- *START* : Sends a low start bit
- *DATA* : Transmits 8 bits in sequence
- *STOP* : Ends transmission with a high bit

3. **Status Indication**
- *ready* signal indicates data can be received
- *tx_out* carries continuous UART stream
- Controlled state transitions ensure smooth data flow

#### Port Analysis
1. **Clock and Reset**
- *clk*: System clock input
- *reset_n*: Active-low asynchronous reset

2. **Data Interface**
- *data*: 32-bit input from sensor
- *valid*: Indicates valid data is present

3. **UART Interface**
- *tx_out*: Serial output conforming to UART format

4. **Status Interface**
- *ready*: Signals readiness to receive new data

#### Internal Component Analysis
1. **State Machine Controller**
- Governs transition between UART states
- Directs data sequencing and protocol framing

2. **Data Buffer**
- Temporarily holds incoming sensor data
- Maintains consistency during transmission

3. **Transmission Controller**
- Sends data bit by bit
- Generates appropriate timing for UART
- Manages start and stop bit control

### Step 2: Design Documentation

#### Block and Circuit Diagram

![image](https://github.com/user-attachments/assets/86d868e5-3f3f-4c62-8c55-901ab252ea13)

![image](https://github.com/user-attachments/assets/34a020a3-91ff-43dd-8fc8-39a092931793)

### Step 3: Implementation

#### Steps to Transmit the Code

1. Create the following [files](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/tree/main/uart_tx_sense) in a folder under VSDSquadron_FM.
2. Then, open terminal and through the commands "cd"; "cd VSDSquadron_FM" and "cd uart_transmission" enter the folder "uart_tx_sense", where you have created the files.
3. Post this, you may verify that the board is connected through "lsusb" command.
4. After this, run "make build" and "sudo make flash".

### Step 4: Testing and Verification

#### Steps of Testing and Verification

1. Open PuTTy.
2. Verify that the correct port is connected through serial communication (COM 64 in my case)
3. Then, check that a series of "D"s are generated and the RGB LED is red.

### Step 5: Documentation

#### Block and Circuit Diagrams:

![image](https://github.com/user-attachments/assets/86d868e5-3f3f-4c62-8c55-901ab252ea13)

![image](https://github.com/user-attachments/assets/e46df768-02aa-4daa-a115-b936e452d363)

#### Video Demonstrating System Transmitting Data
    
https://github.com/user-attachments/assets/2960c9a1-985c-49cc-9d47-1166a2c4a903

> note: Here you cannot see the LED blinking as the time intervals between each 0 and 1 are very tiny

# Task 5: Real-Time Sensor Data Acquisition and Transmission System
## Objective:
This theme revolves around building systems that interface with multiple sensors to gather data, process it within the FPGA, and communicate the results to external devices using protocols like UART. The implementation is structured through the following steps:
1. Conduct comprehensive research on the chosen theme.​
2. Formulate a detailed project proposal outlining the system's functionality, required components, and implementation strategy.
3. Execute the project plan by developing, testing, and validating the system.​
4. Document the entire process comprehensively and create a short video demonstrating the project's functionality.

### Step 1: Literature Review

Personally, I have a lot of experience in arduino and know how to code pretty well, I made this project on my own after looking at this. This is an Automatic LED Lighting system that can turn the led on whenever it detects motion.
 
 ### Step 2: Define System Requirements
 
 **Necessary Hardware Components and Software Tools:**   For this project we need --> an LED( Light Emitting diode ), an HC-SR04 ultrasonic sensor, a programming board ( VSDSquadron FPGA Mini ), connecting wires and [docklight](https://docklight.de/)(Software)
 
 ### Step 3: Design System Architecture
**Block Diagrams:**

![Image](https://github.com/user-attachments/assets/efd7e83f-5b2b-4c94-8996-430fd6355031)

![image](https://github.com/user-attachments/assets/267aa322-212f-4008-af22-9c1ed9a61ab1)

 ### Step 4: Implementation

1. Write the code found [here](https://github.com/Bhavankumar123/VSDSquadron-FPGA-Mini-Internship-program/tree/main/Automatic_led) in a folder named *Automatic_led* under the main *VSDSquadron_FM* folder.
2. Then through *make build* and *sudo make flash* flash the code to the FPGA.
3. Then connect the components as follows:

   ![Image](https://github.com/user-attachments/assets/6828d24b-3608-4e3a-aa38-cdd194e7c84d)

4. Open docklight and choose serial communication with 9600 BAUDS and 1 stop bit. Ensure that the correct port is connected
5. Then, test with your hand or another object. If the hand/object approaches the sensor at a distance closer than 5 cm, the LED will Glow. You can verify the distance through docklight.
