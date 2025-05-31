# **AnalogGym**
![License: BSD 3-Clause](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)

## ⚠️ Important Notice on `ngspice` Version

The latest version of `ngspice` currently available through Conda (`conda install -c conda-forge ngspice`) is **`ngspice-41`**. However, this version has known limitations and may yield inaccurate simulation results, especially in **DC sweeps involving temperature or current sources**.

> ⚠️ **We strongly recommend NOT using the Conda-provided `ngspice`.**

Instead, please **manually upgrade to `ngspice-42` or later** to ensure simulation correctness. These versions fix critical issues present in `ngspice-41`.

You can download newer releases from the official [ngspice SourceForge repository](https://sourceforge.net/projects/ng-spice-rework/files/ng-spice-rework/).

### 🔧 Manual Upgrade Instructions

```bash
# Navigate to the bin directory of your Conda environment (example for Windows)
cd /Anaconda/envs/analoggym-env/Library/bin
# Replace the existing ngspice executable with the newer version you downloaded
```



## **About AnalogGym**

This repository is the analog circuit synthesis testing suite, **AnalogGym**.

AnalogGym encompasses 30 circuit topologies in five key categories: sensing front ends, voltage references, AMPs, low dropout regulators (LDOs), and phase-locked loops (PLLs). 
Among these, the LDOs and AMPs support the open-source [Ngspice](https://ngspice.sourceforge.io/) simulator and the [SkyWater](https://github.com/google/skywater-pdk)  process design kit (PDK), allowing for greater accessibility and reproducibility. 

## **Table of Contents**

- [Getting Started](#Getting_Started)
- [AnalogGym Contents](#AnalogGym_Contents)
- [Usage](#Usage)
  - [Workflow in AnalogGym](#Workflow_in_AnalogGym)
  - [Testbench](#Testbench)
  - [Simulation](#Simulation)
  - [Performance Extraction](#Performance_Extraction)
  - [Additional Resources](#Additional_Resources)
- [Citation](#Citation)
- [Contact](#Contact)

<h2 id="Getting_Started">Getting Started</h2>

Examples of using the AnalogGym with the relational graph neural network and reinforcement learning algorithm[^1], referencing [this repository](https://github.com/ChrisZonghaoLi/sky130_ldo_rl). A [Docker version](https://github.com/CODA-Team/AnalogGym/tree/main/RGNN_RL_Docker) and a [downloadable code](https://github.com/CODA-Team/AnalogGym/tree/main/RGNN_RL) package that can be run locally are provided.

[^1]: Z. Li and A. C. Carusone, "Design and Optimization of Low-Dropout Voltage Regulator Using Relational Graph Neural Network and Reinforcement Learning in Open-Source SKY130 Process," 2023 IEEE/ACM International Conference on Computer-Aided Design (ICCAD), San Francisco, CA, USA, 2023, pp. 01-09, doi: 10.1109/ICCAD57390.2023.10323720.

<h2 id="AnalogGym_Contents">AnalogGym Contents</h2>

The test circuits provided in AnalogGym include:

- `netlist` folder: Contains pre-packaged circuit files that require no modification.
- `testbench` folder: Includes testbench files for running simulations with the simulator.
- `design variables` folder: Stores the input parameters for each circuit separately.
- `schematic` folder: Provides circuit diagrams for reference and visualization.

## **Usage**

Note that in the sky130 PDK, transistors have a drain-source breakdown voltage of 1.8V and a threshold voltage of 1V. Consequently, the supply voltage is maintained at 1.8V, rather than being reduced to 1.2V, to meet the required reliability and operational standards.

<h3 id="Workflow_in_AnalogGym">Workflow in AnalogGym</h3>

<img width="879" alt="AnalogGym_Flow" src="https://github.com/user-attachments/assets/2e06e4cc-7042-42c1-a395-9157f3677d56">

The design flow decouples circuit configuration from the optimization process, allowing for flexible parameter tuning. 
The circuit parameters are maintained in independent configuration files in the `design variables` folder.
Different netlists can be switched in the testbench, with each netlist representing an encapsulated circuit.

<h3 id="Testbench">Testbench</h3>

| Line | Ngspice Testbench Description |
|------|------------------------------------------------------------|
| 1    | `.include ./path_to_spice_netlist/circuit_name`  — *Include the SPICE netlist* |
| 2    | `.include ./path_to_decision_variable/circuit_name` — *Include the circuit parameters (decision variables)* |
| 3    | `.include ./mosfet_model/sky130_pdk/libs.tech/ngspice/corners/tt.spice` — *Include PDK, modify Process in PVT* |
| 4    | `.PARAM supply_voltage = 1.3` — *Specify supply voltage for PVT* |
| 5    | `.temp 27` — *Specify temperature for PVT* |
| 6    | `.PARAM PARAM_CLOAD = 10p` — *Specify load capacitance* |
| ...  | *Simulation commands; no modifications required.* |

<h3 id="Simulation">Simulation</h3>

For the simulation setup and execution, you can check the following scripts:

- For the **Amplifier** simulation, refer to [main_AMP.py](https://github.com/CODA-Team/AnalogGym/blob/main/RGNN_RL/main_AMP.py).
- For the **LDO** simulation, refer to [main_LDO.py](https://github.com/CODA-Team/AnalogGym/blob/main/RGNN_RL/main_LDO.py).

<h3 id="Performance_Extraction">Performance Extraction</h3>

When extracting performance metrics for the included AMP and LDO circuits, the following points should be noted:

- **Amplifier (AMP)**:
  - For AMPs, **Slew Rate (SR)** and **Settling Time** are not directly measurable from simulations and must be **derived** from transient response analysis.
  
- **Low-Dropout Regulator (LDO)**:
  - LDO performance varies under **light load** (5mA) and **heavy load** (55mA) conditions, as load current affects efficiency, stability, and response time. Light load (minload) may reduce power consumption but can slow response, while heavy load (maxload) demands a higher current supply while maintaining stable output voltage.

Two performance extraction scripts are provided for reference: [AMP](https://github.com/CODA-Team/AnalogGym/blob/main/AnalogGym/Amplifier/perf_extraction_amp.py) and [LDO](https://github.com/CODA-Team/AnalogGym/blob/main/AnalogGym/Low%20Dropout%20Regulator/perf_extraction_LDO.py).

<h3 id="Additional_Resources">Additional Resources</h3>

- For a detailed tutorial on using Ngspice, please refer to [this link](https://ngspice.sourceforge.io/tutorials.html).
- Detailed documentation can be found in [doc](https://coda-team.github.io/AnalogGym/), and our paper can be found [here](https://arxiv.org/abs/2409.08534).

### 📌 Update Note: Adjustment to the $\mathrm{FOM}_{\text{AMP}}$ Formula Structure

#### ✅ Original Formula

The originally defined amplifier $FOM_{AMP}$ is given as:

![FOM_AMP Formula](https://latex.codecogs.com/svg.image?%5Cdpi%7B150%7D%20%5Cbg_white%20%5Cmathrm%7BFOM%7D_%7B%5Ctext%7BAMP%7D%7D%20%3D%20%5Cleft(%20%5Cfrac%7B%5Cmathrm%7BPSRR%7D%7D%7B%5Cmathrm%7BPSRR%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BCMRR%7D%7D%7B%5Cmathrm%7BCMRR%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BGain%7D%7D%7B%5Cmathrm%7BGain%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BFOMS%7D%7D%7B%5Cmathrm%7BFOMS%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BFOML%7D%7D%7B%5Cmathrm%7BFOML%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Cright)%20%5Ccdot%20%5Cleft(%20%5Cfrac%7BT_s%7D%7BT_%7Bs%2C%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BArea%7D%7D%7B%5Cmathrm%7BArea%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Cright)%5E%7B-1%7D%20%5Ccdot%20%5Cleft(%20%5Cfrac%7Bv_n%7D%7Bv_%7Bn%2C%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cmathbb%7BI%7D(v_n%20%3E%20v_%7Bn%2C%5Ctext%7Bref%7D%7D)%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BTC%7D%7D%7B%5Cmathrm%7BTC%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cmathbb%7BI%7D(%5Cmathrm%7BTC%7D%20%3E%20%5Cmathrm%7BTC%7D_%7B%5Ctext%7Bref%7D%7D)%20%5Ccdot%20%5Cfrac%7Bv_%7B%5Cmathrm%7Bos%7D%7D%7D%7Bv_%7B%5Cmathrm%7Bos%7D%2C%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cmathbb%7BI%7D(v_%7B%5Cmathrm%7Bos%7D%7D%20%3E%20v_%7B%5Cmathrm%7Bos%7D%2C%5Ctext%7Bref%7D%7D)%20%5Cright)%5E%7B-1%7D) 


The last term represents a **penalty factor**, applied to suppress degradation in three **precision-related metrics**: output noise voltage ($v_n$), temperature coefficient (TC), and input offset voltage ($v_{\mathrm{os}}$). Since these metrics are "smaller is better", a penalty is applied when they exceed their respective reference values.

---

#### ❌ Problem with the Original Expression

While the original expression intends to penalize only when degradation occurs (i.e., when a metric exceeds its reference), it presents a critical issue in implementation:

- If any metric is better than its reference, the corresponding indicator function $\mathbb{I}(\cdot)$ returns 0. This causes the entire product to become 0, making the denominator undefined or divergent.
---

#### 🔁 Revised Expression

To address these issues, we reformulate the penalty term as:

![FOM_Penalty Formula](https://latex.codecogs.com/svg.image?%5Cdpi%7B120%7D%20%5Cmathrm%7BFOM%7D_%7B%5Ctext%7BPenalty%7D%7D%20%3D%20%5Cleft(%20%5Cmax%5Cleft(1,%20%5Cfrac%7Bv_n%7D%7Bv_%7Bn,%5Ctext%7Bref%7D%7D%7D%20%5Cright)%20%5Ccdot%20%5Cmax%5Cleft(1,%20%5Cfrac%7B%5Cmathrm%7BTC%7D%7D%7B%5Cmathrm%7BTC%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Cright)%20%5Ccdot%20%5Cmax%5Cleft(1,%20%5Cfrac%7Bv_%7B%5Cmathrm%7Bos%7D%7D%7D%7Bv_%7B%5Cmathrm%7Bos%7D,%5Ctext%7Bref%7D%7D%7D%20%5Cright)%20%5Cright)%5E%7B-1%7D) 


This structure is logically equivalent to the original one: it penalizes only when a parameter exceeds its reference. 

The updated $\mathrm{FOM}_{\text{AMP}}$ is:

![FOM_AMP Formula](https://latex.codecogs.com/svg.image?%5Cdpi%7B150%7D%20%5Cmathrm%7BFOM%7D_%7B%5Ctext%7BAMP%7D%7D%20%3D%20%5Cleft(%20%5Cfrac%7B%5Cmathrm%7BPSRR%7D%7D%7B%5Cmathrm%7BPSRR%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BCMRR%7D%7D%7B%5Cmathrm%7BCMRR%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BGain%7D%7D%7B%5Cmathrm%7BGain%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BFOM%7D_S%7D%7B%5Cmathrm%7BFOM%7D_%7BS%2C%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BFOM%7D_L%7D%7B%5Cmathrm%7BFOM%7D_%7BL%2C%5Ctext%7Bref%7D%7D%7D%20%5Cright)%20%5Ccdot%20%5Cleft(%20%5Cfrac%7BT_s%7D%7BT_%7Bs%2C%5Ctext%7Bref%7D%7D%7D%20%5Ccdot%20%5Cfrac%7B%5Cmathrm%7BArea%7D%7D%7B%5Cmathrm%7BArea%7D_%7B%5Ctext%7Bref%7D%7D%7D%20%5Cright)%5E%7B-1%7D%20%5Ccdot%20%5Cmathrm%7BFOM%7D_%7B%5Ctext%7BPenalty%7D%7D) 


## **Citation**

Please cite us if you find AnalogGym useful.

- AnalogGym: An Open and Practical Testing Suite for Analog Circuit Synthesis, Jintao Li, Haochang Zhi, Ruiyu Lyu, Wangzhen Li, Zhaori Bi<sup>\*</sup>, Keren Zhu<sup>\*</sup>, Yanhan Zhen, Weiwei Shan, Changhao Yan, Fan Yang, Yun Li<sup>\*</sup>, and Xuan Zeng<sup>\*</sup> IEEE/ACM International Conference on Computer-Aided Design (ICCAD '24), October 27--31, 2024, New York, NY, USA
```bash
@inproceedings{10.1145/3676536.3697117,
  author = {Li, Jintao and Zhi, Haochang and Lyu, Ruiyu and Li, Wangzhen and Bi, Zhaori and Zhu, Keren and Zeng, Yanhan and Shan, Weiwei and Yan, Changhao and Yang, Fan and Li, Yun and Zeng, Xuan},
  title = {AnalogGym: An Open and Practical Testing Suite for Analog Circuit Synthesis},
  year = {2025},
  isbn = {9798400710773},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3676536.3697117 },
  doi = {10.1145/3676536.3697117},
  booktitle = {Proceedings of the 43rd IEEE/ACM International Conference on Computer-Aided Design},
  articleno = {59},
  numpages = {9},
  keywords = {analog circuit optimization, electronic design automation},
  location = {Newark Liberty International Airport Marriott, New York, NY, USA},
  series = {ICCAD '24}
}
```

## **Contact**

If you have any questions, are seeking collaboration, or would like to contribute circuit designs, please get in touch with us at [j.t.li@i4ai.org](mailto:j.t.li@i4ai.org).

<img src="./docs/images/logos/4school.png" alt="school_logo" width="90%"/>
