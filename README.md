# Week 5 - OpenROAD Flow Setup and Floorplan + Placement

## 🚗 vs. 👨‍✈️ OpenROAD vs. OpenROAD-flow-scripts

Before the installation, it's crucial to understand the difference between the **OpenROAD application** (the "engine") and **OpenROAD-flow-scripts** (the "driver").

  * **🚗 OpenROAD (The Engine):** This is the core C++ application. It's a single, powerful tool with a Tcl interface that can perform specific physical design tasks (like placement, routing, etc.).

  * **👨‍✈️ OpenROAD-flow-scripts (The Driver):** This is a set of **automation scripts** (Makefiles, Tcl, and Python) that *uses* the OpenROAD engine. It manages the entire RTL-to-GDSII flow, automatically calling OpenROAD (and other tools like Yosys for synthesis) in the correct sequence.

**For running a full design, we always want to use `OpenROAD-flow-scripts`.**

### Key Differences at a Glance

| Feature | 🚗 OpenROAD (The Application) | 👨‍✈️ OpenROAD-flow-scripts (The Automation) |
| :--- | :--- | :--- |
| **What it is** | A single, powerful C++ executable | A collection of Makefiles & Tcl scripts |
| **Primary Use** | Performing one specific PD task at a time (e.g., `global_placement`) | Running the *entire* automated RTL-to-GDSII flow |
| **How you use it** | In a Tcl shell or GUI, running manual commands | From the terminal, running a single `make` command |
| **Installation** | Complex, manual compilation from source | Simple `git clone` & running a setup script |

-----

## ⚙️ Installation of OpenROAD-flow-scripts

This guide details the process of building all the flow's tools (including OpenROAD and Yosys) from their original C++ source code. This method gives the latest features but takes significantly longer.

### 1\. Clone the Repository and Submodules

```bash
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts.git
cd OpenROAD-flow-scripts
```

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 01-09-09" src="https://github.com/user-attachments/assets/66d93f31-501a-48f8-9795-6a397ce7a326" />

  * **What this does (step-by-step):**
      * `git clone`: Downloads the main `OpenROAD-flow-scripts` repository, which contains all the automation scripts (Makefiles, Tcl, etc.).
      * `--recursive`: This is a **critical** flag. It tells Git to also download the source code of all the "submodules" nested inside, which includes the entire source for `tools/OpenROAD`, `tools/yosys`, and other required tools. 

### 2\. Run the Full Dependency Installer

```bash
sudo ./etc/DependencyInstaller.sh -all
```

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 01-09-42" src="https://github.com/user-attachments/assets/253d414a-de39-4046-b055-a9eae808ee46" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 01-09-51" src="https://github.com/user-attachments/assets/40d1ea3f-7d3d-443a-89c8-8e886bc023d3" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 01-16-07" src="https://github.com/user-attachments/assets/231e2890-866e-46a7-b041-bb2bca1a25b4" />


  * **What this does:**
      * `sudo`: Runs the script with "superuser" (administrator) privileges, which are required to install software system-wide.
      * `./etc/DependencyInstaller.sh`: This is the main setup script you are running.
      * `-all`: This flag tells the script to do *everything*. As your screenshots (`...01-09-42.jpg`, `...01-09-51.png`) show, this includes:
        1.  **Installing system packages:** It runs `apt update` and installs all required development libraries (like `build-essential`, `cmake`, `python3-dev`, `libboost`, etc.).
        2.  **Triggering Local Builds:** Instead of downloading pre-built binaries, this flag tells the script to build every tool from scratch.

### 3\. Build Process

```bash
./build_openroad.sh --local 
```

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 01-29-38" src="https://github.com/user-attachments/assets/e4b12614-8b64-4089-b1ef-4c535a7d4625" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 03-56-40" src="https://github.com/user-attachments/assets/be42596d-3a45-418e-8886-8882f266ea16" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 04-17-00" src="https://github.com/user-attachments/assets/274c84ae-dea1-41d6-a610-a42907042a98" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-26 05-01-46" src="https://github.com/user-attachments/assets/06dbb37b-2ca0-4488-8bc6-d90c5f5b09bf" />

 * **What this does:**
      * `./build_openroad.sh`: This is the dedicated shell script inside the `tools/OpenROAD` folder responsible *only* for compiling the OpenROAD C++ source code.
      * `--local`: This flag tells the script to install the final `openroad` executable into the local `tools/install/` directory within your `OpenROAD-flow-scripts` folder, rather than trying to install it system-wide (which would require `sudo`). This is exactly what the flow needs.

### 4\. Verify the installation 

```bash
source ./env.sh
openroad -help
yosys -help
```

<img width="1920" height="1080" alt="Screenshot from 2025-11-12 15-40-10" src="https://github.com/user-attachments/assets/ac91df1b-87e0-464d-bf3a-3777b2f615da" />

-----

## 🗺️ Example Flow: Floorplan + Placement for 'gcd' using 'nanogate45'

### 1. Synthesis (Stage 1)

* **Command Triggered:** `1_yosys_synthesis`
* **What it Did:** This stage uses **Yosys** to synthesize your design.
    * It reads your Verilog source code (`designs/src/gcd/gcd.v`).
    * It reads the "liberty" file (`.lib`) for the `nanogate45` PDK, which describes all the available standard cells (like `AND`, `OR`, `DFF`, etc.).
    * It converts your abstract Verilog code into a **netlist**—a specific list of those standard cells and how they are connected.
    * Your log shows it finished successfully: `End of script.`

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-07-09" src="https://github.com/user-attachments/assets/01df1936-4416-44f6-b36c-4668934d9d12" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-07-43" src="https://github.com/user-attachments/assets/cdec6a17-ae58-4fc2-beaa-7f033a7f3f76" />

### 2. Floorplan (Stage 2)

* **Command Triggered:** `2_1_floorplan`, `2_3_floorplan_tapcell`, `2_4_floorplan_pdn`
* **What it Did:** Immediately after synthesis, the floorplan stage began.
    * `2_1_floorplan`: This command initialized the chip's boundaries (the **die**) and the area for placing cells (the **core**). Your log shows it achieved a `56% utilization`. It also created the horizontal standard cell rows.
    * `2_3_floorplan_tapcell`: This inserted **tap cells** (`TAP-0004] Inserted 48 endcaps.`). These are special cells placed in the rows to provide a good connection to the power and ground grid, preventing a "latch-up" condition.
    * `2_4_floorplan_pdn`: This generated the **Power Distribution Network** (`[INFO PDN-0001] Inserting grid:`). This is the metal grid (VDD and GND) that delivers power to all the cells.

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-07-55" src="https://github.com/user-attachments/assets/d038547d-c3ca-4d32-9e3e-3776a03f3fea" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-08-42" src="https://github.com/user-attachments/assets/86e35869-c2ad-498e-86e7-764a5f1f1a0d" />

### 3. Placement (Stage 3)

* **Command Triggered:** `3_1_place_gp_skip_io`, `3_5_place_dp`
* **What it Did:** With the floorplan and power grid ready, the flow began placing the 551 standard cells from your synthesized design.
    * `3_1_place_gp...`: This is **Global Placement**. The tool finds the *optimal approximate* location for all cells to minimize wire length and congestion. At this stage, cells are allowed to overlap.
    * `3_5_place_dp...`: This is **Detailed Placement**. The tool takes the "illegal" global placement and legalizes it. It snaps all cells to the grid, ensures there are **zero overlaps** (`[INFO DPL-0312] Found 0 overlaps...`), and fine-tunes the locations to meet design rules.

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-09-00" src="https://github.com/user-attachments/assets/30982160-8931-4182-b345-a8ed4d63c783" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-09-14" src="https://github.com/user-attachments/assets/44fe4d19-8c8c-41a6-a6eb-3ab57338e72d" />

Here is a brief explanation of those results, perfect for adding to your Git repository.

---

## 🔬 Results: Floorplan, Placement, and Timing

### 1. Floorplan Visualization

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-11-22" src="https://github.com/user-attachments/assets/2a9bd1c6-3dec-4f67-86e6-baedb3545974" />

This screenshot shows the successful output of the **floorplanning stage** (`2_floorplan.odb`), loaded in the OpenROAD GUI.

* **Die and Core:** It displays the initialized die boundary (outer rectangle) and the core area (inner rectangle) where cells will be placed.
* **Standard Cell Rows:** The horizontal blue lines are the empty standard cell rows, which act as the "shelves" for holding the logic.
* **Power Grid (PDN):** The overlaid grid of thick pink and green lines is the Power Distribution Network, which delivers VDD (power) and GND (ground) across the chip.

### 2. Detailed Placement Visualization

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-13-39" src="https://github.com/user-attachments/assets/f15b33c6-a79b-43b2-b698-f7ac73aac373" />

This view shows the layout after the **detailed placement stage** (`3_place.odb`).

* **Placed Cells:** The core is now filled with hundreds of small red blocks, which are the individual standard cells (AND gates, flip-flops, etc.) from the `gcd` design.
* **Legalized Layout:** All cells have been "snapped" to the standard cell rows and are perfectly aligned with no overlaps. This is a "legal" placement.
* **Timing Prep:** The Tcl console at the bottom shows the commands used to load this design (`read_db ... 3_place.odb`) and begin running Static Timing Analysis (`sta::find_timing`).

### 3. Post-Placement Static Timing Analysis (STA) Report

<img width="1920" height="1080" alt="Screenshot from 2025-10-27 00-18-27" src="https://github.com/user-attachments/assets/774b1371-1f54-4ed6-ab32-25b57466ac9c" />

This screenshot shows the text report (`3_detailed_place.rpt`) generated after placement. This report analyzes if the design is fast enough to meet its clock speed.

* **Critical Path:** The report details the single worst-offending path in the design.
* **Timing Violation:** The most important line is at the bottom: **`slack (VIOLATED) -0.01`**.
* **Conclusion:** This indicates the design is **failing timing** by a very small margin (0.01ns). The "data arrival time" (0.37ns) is just slightly later than the "data required time" (0.37ns). This is a common result after placement, and the flow will attempt to fix this violation in the upcoming Clock Tree Synthesis (CTS) and routing stages.
