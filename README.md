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

This guide shows how to install the **automation scripts**, which is the recommended method for running designs. This process does not require building OpenROAD from scratch; it downloads pre-compiled binaries.

### 1\. Install Prerequisites

First, install the basic dependencies needed to run the flow scripts (note: this is much simpler than building the app).

```bash
sudo apt update
sudo apt install build-essential git python3 python3-pip python3-venv -y
```

### 2\. Clone the Repository

Clone the `OpenROAD-flow-scripts` repository from GitHub. This contains all the Makefiles and Tcl scripts.

```bash
git clone https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts.git
cd OpenROAD-flow-scripts
```

### 3\. Run the Setup Script

This is the most important step. The included scripts will automatically download the pre-built binaries for all required tools (OpenROAD, Yosys, Magic, etc.) and place them in the `tools/` directory.

```bash
./etc/DependencyInstaller.sh
```

> **Note:** You can also use Docker for an even more isolated environment, but this local installation is straightforward and fast.

After this step, your environment is fully set up to run the flow.

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
