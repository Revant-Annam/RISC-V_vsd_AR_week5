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

## 🗺️ Example Flow: Floorplan + Placement for 'gcd'

With `OpenROAD-flow-scripts`, you don't run Tcl commands one by one. Instead, you **configure variables** in a design file and let `make` run the flow.

### 1\. Configuration (The `config.mk` file)

Every design (like the example `gcd` design) has a `config.mk` file. This is where you define your floorplan and placement goals.

Let's look at `designs/asap7/gcd/config.mk`. You would edit this file to control the flow.

**For Floorplanning:**
Instead of running `init_floorplan` manually, you set these variables:

```makefile
# Target utilization (e.g., 50%)
export FP_CORE_UTIL = 50

# Desired aspect ratio (1.0 = square)
export FP_ASPECT_RATIO = 1.0

# Margin between the core and the die boundary
export FP_CORE_MARGIN = 4.0
```

**For Placement:**
You can set a target placement density.

```makefile
# Target density for placement (e.g., 60%)
export PL_TARGET_DENSITY = 0.6
```

### 2\. Running the Flow (Floorplan + Placement)

To run the flow, you simply use `make` from the `flow/` directory. The Makefiles will automatically run synthesis, then floorplanning, then placement.

1.  **Navigate to the flow directory:**

    ```bash
    cd flow
    ```

2.  **Run the flow up to detailed placement:**
    This command tells the flow to run all steps *up to and including* detailed placement (`6_detail_placement`) for the `gcd` design using the `asap7` PDK.

    ```bash
    make 6_detail_placement DESIGN_CONFIG=../designs/asap7/gcd/config.mk
    ```

The flow will automatically execute:

1.  Synthesis (with Yosys)
2.  **Floorplan** (using your `config.mk` settings)
3.  Tap cell insertion
4.  I/O Placement
5.  **Global Placement**
6.  **Detailed Placement**

### 3\. Viewing the Results

Your original guide showed screenshots from the OpenROAD GUI. You can still do this\! The flow saves the result of *each step* in the `results/` directory.

To see the final placement:

1.  **Launch the OpenROAD GUI:**

    ```bash
    openroad
    ```

2.  **In the GUI, load the final placed design:**
    The flow saves the design database (`.odb`) or a standard `.def` file. You can load this to visually inspect the floorplan and placement.

    The resulting layout will show the core area, power grid, and all standard cells snapped to the rows with no overlaps, just like in your original screenshots.
