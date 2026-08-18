# AMBA APB Protocol Design in Verilog

## 📌 Project Overview

This project implements a simplified **AMBA Advanced Peripheral Bus (APB) Slave Protocol** using **Verilog HDL**. The design follows the basic APB transaction flow and uses a **Finite State Machine (FSM)** to control the different phases of communication.

The APB slave supports:

* Read transactions
* Write transactions
* Memory-based data storage
* IDLE, SETUP, and ACCESS states
* APB control signals
* Verilog-based simulation and verification

The project was developed to understand the working of the **AMBA APB protocol**, RTL design, FSM implementation, and basic bus communication.

---

## 🚀 Features

* Designed using **Verilog HDL**
* Implements a simplified **APB Slave**
* Supports **8-bit address and data width**
* Supports **Read and Write operations**
* Uses an internal memory of `256 × 8 bits`
* Implements APB FSM with three states:

  * IDLE
  * SETUP
  * ACCESS
* Uses APB signals such as:

  * `PCLK`
  * `PRESETN`
  * `PSEL`
  * `PENABLE`
  * `PWRITE`
  * `PADDR`
  * `PWDATA`
  * `PRDATA`
  * `PREADY`

---

# 🏗️ APB Protocol Architecture

The APB protocol consists of a master and one or more peripheral slaves.

The master initiates communication using control signals, while the slave responds with read data and transfer-ready status.

### Basic APB Transaction Flow

```text
                +----------------+
                |   APB MASTER   |
                +----------------+
                  |            |
        Address   |            | Read Data
        Write Data|            |
        Control   |            |
                  v            ^
                +----------------+
                |   APB SLAVE    |
                |                |
                |   MEMORY       |
                +----------------+
```

---

# 🔌 APB Signals

| Signal        | Direction | Description                       |
| ------------- | --------- | --------------------------------- |
| `PCLK`        | Input     | APB clock signal                  |
| `PRESETN`     | Input     | Active-low reset signal           |
| `PSEL`        | Input     | Slave select signal               |
| `PENABLE`     | Input     | Enables the access phase          |
| `PWRITE`      | Input     | Indicates read or write operation |
| `PADDR[7:0]`  | Input     | Address bus                       |
| `PWDATA[7:0]` | Input     | Write data bus                    |
| `PRDATA[7:0]` | Output    | Read data bus                     |
| `PREADY`      | Output    | Indicates slave readiness         |

---

# 🔄 APB Finite State Machine

The APB protocol implementation uses three states.

## 1. IDLE State

In this state, no APB transfer is active.

```text
PSEL = 0
PENABLE = 0
```

When the master selects the slave:

```text
PSEL = 1
```

the FSM moves to the **SETUP** state.

---

## 2. SETUP State

The setup phase prepares the transfer.

```text
PSEL    = 1
PENABLE = 0
```

During this phase, the address and control signals are provided by the APB master.

On the next clock cycle, the FSM moves to the **ACCESS** state.

---

## 3. ACCESS State

The actual read or write operation occurs during this state.

```text
PSEL    = 1
PENABLE = 1
```

For a write operation:

```text
PWRITE = 1
```

The data from `PWDATA` is stored into memory.

For a read operation:

```text
PWRITE = 0
```

The data stored at `PADDR` is transferred to `PRDATA`.

---

# 📊 State Transition Diagram

```text
                     +--------+
                     |  IDLE  |
                     +--------+
                          |
                       PSEL=1
                          |
                          v
                     +--------+
                     | SETUP  |
                     |PSEL=1  |
                     |PENABLE=0
                     +--------+
                          |
                     Next Clock
                          |
                          v
                     +--------+
                     | ACCESS |
                     |PSEL=1  |
                     |PENABLE=1
                     +--------+
                       |      |
             Transfer  |      | New Transfer
             Complete  |      |
                       v      v
                     IDLE   SETUP
```

---

# 📝 RTL Design

The APB slave is implemented using Verilog.

```verilog
module APB(
    PCLK,
    PRESETN,
    PSEL,
    PENABLE,
    PADDR,
    PWDATA,
    PRDATA,
    PWRITE,
    PREADY
);

    input PCLK, PRESETN, PSEL, PENABLE, PWRITE;
    input [7:0] PADDR;
    input [7:0] PWDATA;

    output PREADY;
    output reg [7:0] PRDATA;

    parameter IDLE   = 2'b00;
    parameter SETUP  = 2'b01;
    parameter ACCESS = 2'b10;

    reg [7:0] mem [255:0];

    reg [1:0] current_state;
    reg [1:0] next_state;

    assign PREADY = 1'b1;

    always @(posedge PCLK or negedge PRESETN)
    begin
        if (!PRESETN)
        begin
            current_state <= IDLE;
            PRDATA <= 8'b00000000;
        end
        else
        begin
            current_state <= next_state;

            if (current_state == ACCESS && PSEL && PENABLE)
            begin
                if (PWRITE)
                    mem[PADDR] <= PWDATA;
                else
                    PRDATA <= mem[PADDR];
            end
        end
    end

    always @(*)
    begin
        case (current_state)

            IDLE:
            begin
                if (PSEL)
                    next_state = SETUP;
                else
                    next_state = IDLE;
            end

            SETUP:
            begin
                if (PSEL && PENABLE)
                    next_state = ACCESS;
                else
                    next_state = SETUP;
            end

            ACCESS:
            begin
                if (PSEL)
                    next_state = SETUP;
                else
                    next_state = IDLE;
            end

            default:
                next_state = IDLE;

        endcase
    end

endmodule
```

---

# ✍️ Write Transaction

A write operation takes place when:

```text
PSEL    = 1
PENABLE = 1
PWRITE  = 1
```

The data available on `PWDATA` is written into the internal memory location specified by `PADDR`.

### Example

```text
PADDR  = 8'h10
PWDATA = 8'hAA
PWRITE = 1
```

Operation:

```text
mem[8'h10] = 8'hAA
```

---

# 📖 Read Transaction

A read operation takes place when:

```text
PSEL    = 1
PENABLE = 1
PWRITE  = 0
```

The data stored in the memory location specified by `PADDR` is transferred to `PRDATA`.

### Example

```text
PADDR  = 8'h10
PWRITE = 0
```

If:

```text
mem[8'h10] = 8'hAA
```

Then:

```text
PRDATA = 8'hAA
```

---

# 🧪 Verification

The design can be verified using a Verilog testbench.

The following scenarios should be tested:

### Reset Test

Verify that the FSM returns to the IDLE state when:

```text
PRESETN = 0
```

Expected result:

```text
current_state = IDLE
PRDATA = 0
```

---

### Write Transaction Test

Test the following sequence:

```text
PSEL = 1
PENABLE = 0
PWRITE = 1
PADDR = Address
PWDATA = Data
```

Then move to ACCESS:

```text
PENABLE = 1
```

Expected result:

```text
mem[PADDR] = PWDATA
```

---

### Read Transaction Test

Perform a read from a previously written memory location.

Example:

```text
Write:

Address = 10
Data    = AA
```

Then:

```text
Read:

Address = 10
```

Expected output:

```text
PRDATA = AA
```

---

# 📈 Simulation Waveform

The waveform verifies the APB signal transitions during read and write operations.
<img width="1361" height="393" alt="image" src="https://github.com/user-attachments/assets/b8712ab2-e63d-409c-8af7-839fe471cc6d" />


The important signals to observe are:

```text
PCLK
PRESETN
PSEL
PENABLE
PWRITE
PADDR
PWDATA
PRDATA
PREADY
```

### Expected APB Timing

```text
Clock        _|‾|_|‾|_|‾|_|‾|_

PSEL         ______|‾‾‾‾‾‾‾‾

PENABLE      __________|‾‾‾‾

PWRITE       ______|‾‾‾‾‾‾‾‾

                SETUP   ACCESS
```

The transaction follows the APB sequence:

```text
IDLE → SETUP → ACCESS
```

---

# 🛠️ Tools Used

* **Verilog HDL**
* **QuestaSim / ModelSim**
* **gVim / Text Editor**
* **GTKWave** (optional for waveform viewing)

---

# 📂 Project Structure

```text
AMBA-APB-Protocol-Design
│
├── rtl
│   └── APB.v
│
├── tb
│   └── APB_tb.v
│
├── simulation
│   └── waveform.png
│
├── docs
│   └── APB_FSM.png
│
└── README.md
```

---

# 🎯 Learning Outcomes

Through this project, the following concepts were implemented and understood:

* AMBA APB Protocol
* RTL Design
* Verilog HDL
* Finite State Machine Design
* Synchronous Digital Design
* APB Read Transactions
* APB Write Transactions
* Memory Interfacing
* Simulation and Debugging
* Timing Diagram Analysis

---

# 🔮 Future Improvements

Possible improvements for this project include:

* Implementing configurable wait states using `PREADY`
* Adding `PSLVERR` error response
* Supporting wider address and data buses
* Implementing multiple APB slave interfaces
* Adding assertion-based verification
* Developing a SystemVerilog testbench
* Adding functional coverage
* Integrating the APB slave with a larger SoC design

---

# 👨‍💻 Author

**Venkatesh R S**

Electronics and Communication Engineer
Interested in **VLSI Design & Verification, Verilog, SystemVerilog, and Embedded Systems**.

* LinkedIn: [Venkatesh R S](https://www.linkedin.com/in/venkatesh-r-shettar-15a048259)
* GitHub: [Venkii17](https://github.com/Venkii17)


			
		

	
