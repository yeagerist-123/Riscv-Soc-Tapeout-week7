# RISC-V_vsd_AR_week7

## Installation of the ORFS

This has been done in the [Week 5]

## RTL to GDS for BabySoC

Main steps in the flow are:
1. Synthesis
2. Floorplan
3. Placement
4. CTS
5. Routing
6. Signoff

### Files required for the complete flow

For using ORFS for a new design we have to first create a directory `vsdbabysoc` in the `~/OpenROAD-flow-scripts/flow/designs/sky130hd` directory and the `~/OpenROAD-flow-scripts/flow/designs/src` directory. The purpose of creating the directory in the 2 directories is:

<img width="1920" height="1080" alt="w7-1" src="https://github.com/user-attachments/assets/9939a7fa-f970-4c48-bf78-6aacc4bd5257" />


1. `~/OpenROAD-flow-scripts/flow/designs/src/vsdbabysoc/`
   
  * **Purpose:** This is the **design source** directory. It is **PDK-independent**. It holds all the raw design files that describe your chip's logic and timing.
      * `*.v` / `*.sv` (Your Verilog or SystemVerilog RTL files)

We can take this one folder and target a different PDK (like `gf180`) without changing its contents. Now we have to copy the folder `module` from the VSDBabySoC directory which was installed in week3. The `module` folder would contain the verilog files


2. `~/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/`

  * **Purpose:** This is the **platform-specific configuration** directory. It tells the ORFS `Makefile` *how* to build your design specifically for the `sky130hd` PDK. It also generally contains any other files related to the design. We wil now copy the `lef`, `lib`, `gds`, `include`, `sdc` folders from the VSDBabySoC folder.

  - The `gds` folder would contain the files `avsddac.gds` and `avsdpll.gds`
  - The `include` folder would contain the files `sandpiper.vh`, `sandpiper_gen.vh`, `sp_default.vh` and `sp_verilog.vh`
  - The `lef` folder would contain the files `avsddac.lef` and `avsdpll.lef`
  - The `lib` folder would contain the files `avsddac.lib` and `avsdpll.lib`
  - The `sdc` folder would contain the `vsdbabysoc_synthesis.sdc` and the `vsdbabysoc_layout.sdc`

We have to copy the files(`macro.cfg` and `pin_order.cfg`) from the VSDBabySoC folder.

<img width="1920" height="923" alt="w7-2" src="https://github.com/user-attachments/assets/ed0aa7aa-d0e2-41b1-af6c-d5a9b45597f0" />


The key file we have to create here is **`config.mk`**. This `config.mk` file is what connects everything. It tells the flow:
  * **What design to build:** `DESIGN_NAME = vsdbabysoc`
  * **Where to find the source files:** `VERILOG_FILES = ./designs/src/vsdbabysoc/module/vsdbabysoc.v`
  * **Where to find the SDC file:** `SDC_FILE = ./designs/src/vsdbabysoc/vsdc/sdbabysoc_synthesis.sdc`
  * **Platform-specific settings:** `CORE_UTILIZATION = 50`, `PLACE_DENSITY = 0.6`

Contents of the `config.mk` file:

```make
export DESIGN_NICKNAME = vsdbabysoc
export DESIGN_NAME = vsdbabysoc
export PLATFORM    = sky130hd

# export VERILOG_FILES_BLACKBOX = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/IPs/*.v
# export VERILOG_FILES = $(sort $(wildcard $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/*.v))
# Explicitly list the Verilog files for synthesis
export VERILOG_FILES = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/module/vsdbabysoc.v \
                       $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/module/rvmyth.v \
                       $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/module/clk_gate.v

export SDC_FILE      = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/sdc/vsdbabysoc_synthesis.sdc

export vsdbabysoc_DIR = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)

export VERILOG_INCLUDE_DIRS = $(wildcard $(vsdbabysoc_DIR)/include/)
# export SDC_FILE      = $(wildcard $(vsdbabysoc_DIR)/sdc/*.sdc)
export ADDITIONAL_GDS  = $(wildcard $(vsdbabysoc_DIR)/gds/*.gds)
export ADDITIONAL_LEFS  = $(wildcard $(vsdbabysoc_DIR)/lef/*.lef)
export ADDITIONAL_LIBS = $(wildcard $(vsdbabysoc_DIR)/lib/*.lib)
# export PDN_TCL = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/pdn.tcl

# Clock Configuration (vsdbabysoc specific)
# export CLOCK_PERIOD = 20.0
export CLOCK_PORT = CLK
export CLOCK_NET = $(CLOCK_PORT)

# Floorplanning Configuration (vsdbabysoc specific)
export FP_PIN_ORDER_CFG = $(wildcard $(vsdbabysoc_DIR)/pin_order.cfg)
# export FP_SIZING = absolute

export DIE_AREA   = 0 0 1600 1600
export CORE_AREA  = 20 20 1590 1590

# Placement Configuration (vsdbabysoc specific)
export MACRO_PLACEMENT_CFG = $(wildcard $(vsdbabysoc_DIR)/macro.cfg)
export PLACE_PINS_ARGS = -exclude left:0-600 -exclude left:1000-1600: -exclude right:* -exclude top:* -exclude bottom:*
# export MACRO_PLACEMENT = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/macro_placement.cfg

export TNS_END_PERCENT = 100
export REMOVE_ABC_BUFFERS = 1

# Magic Tool Configuration
export MAGIC_ZEROIZE_ORIGIN = 0
export MAGIC_EXT_USE_GDS = 1

# CTS tuning
export CTS_BUF_DISTANCE = 600
export SKIP_GATE_CLONING = 1

# export CORE_UTILIZATION=0.1  # Reduce this value to allow more whitespace for routing.
```

### 1. Synthesis

```bash
cd OpenROAD-flow-scripts
source env.sh
cd flow
```

This is where we run our make files.

Command for synthesis:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
```

<img width="1920" height="1080" alt="w7-3" src="https://github.com/user-attachments/assets/8570c01f-34c0-4e68-a841-ca59aa76f328" />


The vsdbabysoc netlist:

<img width="1920" height="1080" alt="w7-4" src="https://github.com/user-attachments/assets/929dc6d7-807e-407b-9b45-e8044f20af9c" />


Statistics report:

<img width="1920" height="923" alt="w7-6" src="https://github.com/user-attachments/assets/581aeca3-8119-43cb-9a22-3d89a9a5f5e9" />


### 2. Floorplan

Command for synthesis:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan
```

<img width="1920" height="923" alt="w7-7" src="https://github.com/user-attachments/assets/488fdbfd-4ed3-42a8-9e2c-619be5c3af65" />


To open the GUI version we have to run the command:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
```
<img width="1920" height="923" alt="w7-8" src="https://github.com/user-attachments/assets/36e328e3-ca48-4233-a344-3f378475ed4a" />



### 3. Placement

Command for placement:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
```

<img width="1920" height="923" alt="w7-9" src="https://github.com/user-attachments/assets/3b32b977-840c-4598-8d8f-c473a779ae8e" />

<img width="1920" height="923" alt="w7-10" src="https://github.com/user-attachments/assets/40d3732e-e9c3-466e-9761-1f230a2e5029" />


To open the GUI version we have to run the command:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place
```

<img width="1920" height="923" alt="w7-11" src="https://github.com/user-attachments/assets/d797ce6c-ca3f-4a6c-b2e3-d46dcd4eab4c" />


Heatmap:(zoom in press z)

<img width="1920" height="923" alt="w7-12" src="https://github.com/user-attachments/assets/610a629f-5d03-495e-a25e-06791998e0da" />


### 4. CTS

Command for CTS:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts
```
<img width="1920" height="923" alt="w7-13" src="https://github.com/user-attachments/assets/6b375864-d7c0-41c5-b508-a38aa6b173e7" />

<img width="1920" height="923" alt="w7-14" src="https://github.com/user-attachments/assets/a5735ce1-5afa-4f1e-9f1c-6e0b8e8e4060" />


CTS Report:

<img width="1920" height="923" alt="w7-16" src="https://github.com/user-attachments/assets/a4d40610-5b6d-4772-b379-207dfd7b0158" />


Hold analysis report:

<img width="1920" height="923" alt="w7-17" src="https://github.com/user-attachments/assets/fc01c7ff-9422-48ec-8a6f-22abaf7872d8" />


Setup analysis report:

<img width="1920" height="923" alt="w7-18" src="https://github.com/user-attachments/assets/5dd1fa77-959c-4c6f-9d59-0f165306557f" />


CTS:

<img width="1920" height="923" alt="w7-15" src="https://github.com/user-attachments/assets/933068fd-5874-44ff-a39b-9a446d2c1e9a" />


### 5. Routing

Command for ROUTE:
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route
```

<img width="1920" height="923" alt="w7-19" src="https://github.com/user-attachments/assets/3fedadc2-8931-4bfe-b8bf-c90b7b8d388b" />

<img width="1920" height="923" alt="w7-20" src="https://github.com/user-attachments/assets/bef3bd33-e9c1-4bce-bf58-3f3c5d9648f6" />
