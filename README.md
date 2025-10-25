# Week 5 - OpenROAD Flow Setup and Floorplan + Placement

## **OpenROAD Installation Guide**

OpenROAD is an integrated executable application that performs physical design for VLSI. This guide details the installation procedure.

-----

### **Section 1: Install Prerequisites and Compiler**

The OpenROAD build process is sensitive to the compiler version. These steps will install essential build tools and the required GCC-9 compiler.

1.  **Update repositories and install build-essential packages:**

    ```bash
    sudo apt update
    sudo apt install build-essential git cmake swig python3-dev -y
    ```

2.  **Add the toolchain repository and install g++-9:**

    ```bash
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
    sudo apt update
    sudo apt install g++-9 -y
    ```

### **Section 2: Build and Install OR-Tools Dependency**

OR-Tools is a critical dependency that must be built and installed to a system path (e.g., `/usr/local`) for OpenROAD to locate it.

1.  **Clone the OR-Tools repository:**

    ```bash
    git clone https://github.com/google/or-tools.git
    cd or-tools
    ```

2.  **Create a build directory, configure, compile, and install:**

    ```bash
    mkdir build && cd build
    cmake -DBUILD_DEPS=ON -DCMAKE_BUILD_TYPE=Release ..
    make -j$(nproc)
    sudo make install
    ```

3.  **Return to the original directory:**

    ```bash
    cd ../..
    ```

### **Section 3: Clone and Patch OpenROAD Source**

This section involves cloning the main OpenROAD repository and applying manual patches to resolve dependency issues related to `spdlog`.

1.  **Clone the OpenROAD repository in home directory:**

    ```bash
    git clone https://github.com/The-OpenROAD-Project/OpenROAD.git
    cd OpenROAD
    ```

2.  **Manually clone the `spdlog` dependency into the `third-party` folder:**

    ```bash
    cd third-party
    git clone https://github.com/gabime/spdlog.git
    cd ../.. 
    ```

3.  **Patch `src/CMakeLists.txt`:**
    Open the file (`nano src/CMakeLists.txt`) and comment out the `find_package` line for `spdlog` (approx. line 235) to prevent it from searching for a system-wide version.

    **Change this:**

    ```cmake
    find_package(spdlog REQUIRED)
    ```

    **To this:**

    ```cmake
    # find_package(spdlog REQUIRED)
    ```

4.  **Patch the root `CMakeLists.txt`:**
    Open the root `CMakeLists.txt` file (`nano CMakeLists.txt`) and add a line to build the `spdlog` library from the subdirectory we just cloned. Add it immediately after `add_subdirectory(third-party)`.

    ```cmake
    add_subdirectory(third-party)
    # Add this line
    add_subdirectory(third-party/spdlog)
    add_subdirectory(src)
    ```

### **Section 4: Configure the OpenROAD Build**

With the source patched, you can now configure the project using CMake, specifying the correct compiler and dependency paths.

1.  **Create a clean build directory:**

    ```bash
    rm -rf build
    mkdir build
    cd build
    ```

2.  **Run CMake with the required flags:**
    This command points CMake to the g++-9 compiler and the location of the OR-Tools installation.

    ```bash
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CXX_FLAGS="-Wno-error" \
    -DCMAKE_PREFIX_PATH="/usr/local" \
    -DCMAKE_CXX_COMPILER=/usr/bin/g++-9
    ```

### **Section 5: Compile and Install OpenROAD**

The final step is to compile the source code and install the binary.

1.  **Compile the project (this may take a significant amount of time):**

    ```bash
    make -j$(nproc)
    ```

2.  **Install the application to the system:**

    ```bash
    sudo make install
    ```

-----

Upon successful completion of these steps, you can launch the application by running the `openroad` command in your terminal.


## **Summary of Floorplanning and Placement Stages using OpenROAD**

### **1. Floorplanning**

Floorplanning is the initial stage of physical design, responsible for defining the chip's physical boundaries, allocating area for logic, and establishing the power infrastructure.

#### **1.1. Core and Die Initialization (`flow_floorplan.tcl`)**

The first step involved defining the chip's dimensions and the usable logic area.

* **Process:** The script initialized the **Die Area** (the total chip boundary) and the **Core Area** (the inner region for cell placement).
* **Observation:** The layout view successfully rendered the die and core boundaries. Within the core, horizontal **Standard Cell Rows** were created, establishing the foundational grid for subsequent logic placement.

#### **1.2. Power Distribution Network (PDN) Generation (`flow_pdn.tcl`)**

With the core area defined, the next critical step was to create the power grid.

* **Process:** The script generated the Power Distribution Network (PDN), creating a grid of metal straps for power (VDD, shown as red lines) and ground (GND, shown as pink lines).
* **Observation:** The PDN was successfully overlaid onto the core area. This ensures that the standard cell rows have access to the necessary power and ground connections, making the design structurally ready for cell placement.

---

### **2. Placement**

Placement involves arranging all standard cells within the core rows to meet key design objectives: minimizing wire length, reducing congestion, and satisfying timing constraints. This is executed in two phases.

#### **2.1. Global Placement (`flow_global_placement.tcl`)**

Global placement determines the optimal, approximate location for each standard cell to minimize overall wire length.

* **Process:** The tool placed all standard cells (visible as small red blocks) onto the rows based on their connectivity. I/O pins were also placed along the chip periphery.
* **Observation:** At this stage, cells were clustered based on logical connections, but legality was not enforced. This resulted in significant cell overlapping, which is the expected outcome of global placement.

#### **2.2. Detailed Placement (`flow_detalied_placement.tcl`)**

Detailed placement takes the output of the global phase and legalizes it, ensuring no cells overlap and all design rules are met.

* **Process:** The tool adjusted cell positions, snapping them to the placement grid defined by the standard cell rows.
* **Observation:** The resulting layout shows all standard cells (red blocks) are now neatly aligned within the horizontal rows with no overlaps. This represents a legal, routable placement, ready for the next stage (Clock Tree Synthesis).

---

### **3. Post-Placement Timing Analysis**

Following the completion of detailed placement, an initial timing analysis was performed to assess the design's adherence to timing constraints.

* **Worst Negative Slack (WNS) / Setup Timing:** The analysis reported a **WNS of +0.106 ns**. A positive WNS indicates that the design currently meets all setup timing requirements, meaning the longest signal paths are faster than the clock cycle allows.
* **Total Negative Slack (TNS) / Hold Timing:** The analysis reported a **TNS of -0.009 ns**. This small negative value indicates the presence of minor hold time violations. It is standard for minor hold violations to exist at this stage; they are typically resolved during or after Clock Tree Synthesis (CTS) and subsequent optimization steps.
