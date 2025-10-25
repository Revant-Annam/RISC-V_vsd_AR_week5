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
<img width="1920" height="1080" alt="Screenshot from 2025-10-24 16-11-12" src="https://github.com/user-attachments/assets/27b89cc8-f9be-4bf0-af34-659e9ab1eb08" />

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
<img width="1920" height="1080" alt="Screenshot from 2025-10-24 14-02-03" src="https://github.com/user-attachments/assets/d010aca9-fcd9-4385-9fc7-36ed39331497" />

2.  **Manually clone the `spdlog` dependency into the `third-party` folder:**

    ```bash
    cd third-party
    git clone https://github.com/gabime/spdlog.git
    cd ../.. 
    ```

3.  Build dependencies using the commaand:
   
   ```bash
    ./etc/DepenndencyInstaller.sh -common -local 
   ```
<img width="1920" height="1080" alt="Screenshot from 2025-10-24 22-47-02" src="https://github.com/user-attachments/assets/dd6fd602-5990-4382-ba4c-fa05fa184340" />

### **Section 4: Configure the OpenROAD Build**

We can now configure the project using CMake, specifying the correct compiler and dependency paths.

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
<img width="1920" height="1080" alt="Screenshot from 2025-10-25 18-50-55" src="https://github.com/user-attachments/assets/4f4c82ce-4024-4dd5-895a-5a2a9224ae2f" />

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
<img width="1920" height="1080" alt="Screenshot from 2025-10-25 00-51-29" src="https://github.com/user-attachments/assets/7b61b494-585b-4086-a097-cbd951927039" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-25 00-51-46" src="https://github.com/user-attachments/assets/5e0a8bf6-d2ec-4d82-828e-7543869fe1f2" />

-----

Upon successful completion of these steps, you can launch the application by running the `openroad` command in your terminal.


## **Summary of Floorplanning and Placement Stages using OpenROAD**

<img width="1920" height="1080" alt="Screenshot from 2025-10-25 16-28-48" src="https://github.com/user-attachments/assets/c4f3c51d-6774-4099-b1c6-87d5b3f04051" />

<img width="1920" height="1080" alt="Screenshot from 2025-10-25 16-31-20" src="https://github.com/user-attachments/assets/92dc9455-2a5b-49b6-a337-695e6b5af9fe" />

### **1. Floorplanning**

Floorplanning is the initial stage of physical design, responsible for defining the chip's physical boundaries, allocating area for logic, and establishing the power infrastructure.

#### **1.1. Core and Die Initialization (`flow_floorplan.tcl`)**

The first step involved defining the chip's dimensions and the usable logic area.

* **Process:** The script initialized the **Die Area** (the total chip boundary) and the **Core Area** (the inner region for cell placement).
* **Observation:** The layout view successfully rendered the die and core boundaries. Within the core, horizontal **Standard Cell Rows** were created, establishing the foundational grid for subsequent logic placement.

<img width="3972" height="2505" alt="image" src="https://github.com/user-attachments/assets/6161602e-6e2c-4455-9668-fec5f66c5092" />

<img width="3972" height="2505" alt="image" src="https://github.com/user-attachments/assets/5a262579-7205-49c3-a556-a51088576a94" />

#### **1.2. Power Distribution Network (PDN) Generation (`flow_pdn.tcl`)**

With the core area defined, the next critical step was to create the power grid.

* **Process:** The script generated the Power Distribution Network (PDN), creating a grid of metal straps for power (VDD, shown as red lines) and ground (GND, shown as pink lines).
* **Observation:** The PDN was successfully overlaid onto the core area. This ensures that the standard cell rows have access to the necessary power and ground connections, making the design structurally ready for cell placement.

<img width="3972" height="2505" alt="image" src="https://github.com/user-attachments/assets/7fd5144e-f59e-46bb-85f2-816aa7295cb5" />

---

### **2. Placement**

Placement involves arranging all standard cells within the core rows to meet key design objectives: minimizing wire length, reducing congestion, and satisfying timing constraints. This is executed in two phases.

#### **2.1. Global Placement (`flow_global_placement.tcl`)**

Global placement determines the optimal, approximate location for each standard cell to minimize overall wire length.

* **Process:** The tool placed all standard cells (visible as small red blocks) onto the rows based on their connectivity. I/O pins were also placed along the chip periphery.
* **Observation:** At this stage, cells were clustered based on logical connections, but legality was not enforced. This resulted in significant cell overlapping, which is the expected outcome of global placement.

<img width="3972" height="2505" alt="image" src="https://github.com/user-attachments/assets/8efabe6c-1a1f-44f6-991b-478dbda699a8" />

#### **2.2. Detailed Placement (`flow_detalied_placement.tcl`)**

Detailed placement takes the output of the global phase and legalizes it, ensuring no cells overlap and all design rules are met.

* **Process:** The tool adjusted cell positions, snapping them to the placement grid defined by the standard cell rows.
* **Observation:** The resulting layout shows all standard cells (red blocks) are now neatly aligned within the horizontal rows with no overlaps. This represents a legal, routable placement, ready for the next stage (Clock Tree Synthesis)

<img width="3972" height="2505" alt="image" src="https://github.com/user-attachments/assets/46034af5-6c54-4d61-921f-add07dc9e85f" />
