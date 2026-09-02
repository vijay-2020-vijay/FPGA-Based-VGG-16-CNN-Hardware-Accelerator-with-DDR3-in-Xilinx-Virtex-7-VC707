# FPGA-Based Hardware Acceleration of VGG-16 Deep Convolutional Neural Network Using DDR3 for Real-Time AI Inference

A complete, silicon-ready hardware accelerator for the full VGG-16 deep neural network, designed from scratch in Verilog HDL and implemented on a Xilinx Virtex-7 VC707 FPGA. This project delivers a custom 4-engine parallel CNN accelerator with a DDR3-backed memory hierarchy, achieving **107.2 GOP/s** throughput at **4.62 W** — matching ASIC-class efficiency while retaining FPGA flexibility.

**Presented by:** Rabisankar Maity
**Under the supervision of:** Prof. Roy Paily Palathinkal
**Department of Electronics and Electrical Engineering, Indian Institute of Technology Guwahati**

---

##  Full Project Download

All RTL source files, simulation results, the full project presentation, and reference documentation are packaged together.

**[📁 Download Full Project (Google Drive)](https://drive.google.com/file/d/1-XOkdn0KZGbk76jt35BEpLkjuwYOc8G-/view?usp=drive_link)**
**[📁 Experimental Images & Screenshots (Google Drive Folder)](https://drive.google.com/drive/folders/1DM7Ek4enavNAiZPGLPzdEr_0tPWFCERx?usp=sharing)**



##  The Problem

VGG-16 is a 16-layer convolutional neural network requiring:
- **138 million parameters**
- **~15.5 billion multiply-accumulate (MAC) operations** per image
- **292.67 MB** of total memory for weights, biases, and feature maps

A CPU takes *seconds* per inference. Meanwhile, a typical Virtex-7 FPGA has only **4 MB of on-chip BRAM** — a **73× memory gap** against what VGG-16 actually needs. Existing FPGA/ASIC solutions in literature solve this by either burning excessive power (10–35 W), consuming enormous DSP/LUT resources, or dropping to low-bit precision with severe latency penalties.

**Goal of this project:** Match ASIC-level performance and power efficiency, solve the memory wall, and retain full FPGA reconfigurability — all three simultaneously, without compromise.

---

##  Performance Comparison with State-of-the-Art

| Work | Year | Type | Platform / Tech. | DSP | BRAM | LUT | FF | Freq. (MHz) | Latency (ms) | Throughput (GOP/s) | Power (W) | Precision (bit) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **FPGA Implementations** | | | | | | | | | | | | |
| Mei et al. | 2017 | FPGA | Xilinx XC7VX690T | 1,728 | 196.5 | 210,992 | 219,538 | 200 | 151.91 | 202.42 | 10.81 | 16-float |
| Kala et al. | 2010 | FPGA | Zynq XC7Z045 | 780 | 486 | 182,000 | 127,700 | 150 | 224.60 | 136.97 | 9.63 | 16 |
| Li et al. | 2020 | PPGA | Stratix V GXA7 | 256 | 1,377 | — | — | 200 | 46.30 | 669.10 | — | 16 |
| Li et al. | 2020 | PPGA | Virtex-7 VX900t | 2,160 | 1,220 | — | — | 150 | 106.60 | 290.00 | 35.00 | 16 |
| Li et al. | 2020 | PPGA | Intel Arria 10 | 1,518 | 2,232 | — | — | 200 | 43.20 | 715.90 | — | 16 |
| Yuan et al. | 2021 | PPGA | Zynq ZC706 | 780 | 486 | 182,616 | 127,653 | — | 112.4† | — | — | — |
| Yuan et al. | 2021 | PPGA | Virtex-7 VC707 | 2,296 | — | 215,556 | 66,792 | — | 29.6† | — | — | 16 |
| Yuan et al. | 2021 | PPGA | Virtex-7 VC709 | 2,877 | 882.5 | 337,152 | 606,307 | — | 22.0† | — | — | — |
| Zhang & Zhang | 2024 | FPGA | Zynq-7020 | 138 | — | — | — | 130 | 182.30 | 41.20 | — | 8 |
| **ASIC Implementations** | | | | | | | | | | | | |
| Eyeriss | 2017 | ASIC | 65 nm CMOS | — | — | — | — | 200 | 4309.50 | 21.40 | 0.236 | 16 |
| FID | 2020 | ASIC | 65 nm CMOS | — | — | — | — | 200 | 453.30 | 67.70 | 0.260 | 16 |
| ZASCAD | 2020 | ASIC | 65 nm CMOS | — | — | — | — | 200 | 421.80 | 72.50 | 0.301 | 16 |
| CARLA | 2020 | ASIC | 65 nm CMOS | — | — | — | — | 200 | 92.70 | 75.40 | 0.247 | 16 |
| **This Work** | **2026** | **FPGA** | **Virtex-7 VC707** | **300** | **388** | **98,735** | **77,375** | **200** | **369.97** | **107.2** | **4.62** | **16** |

**The headline result:** This work sustains **107.2 GOP/s at just 4.62 W** using only **300 DSPs** — a fraction of the DSP count used by comparable high-throughput FPGA designs (up to **9.6× fewer DSPs** than Yuan et al.'s VC709 implementation), while consuming **up to 7.5× less power** than other high-throughput FPGA accelerators in the same class. Against ASIC implementations, this design achieves **comparable or better throughput at similar power efficiency**, without sacrificing the reconfigurability that FPGAs uniquely offer.



---

## 🏗️ Architecture Overview

The system is built around **4 parallel CNN engines** (Engine A–D), each with its own dual-BRAM pair, orchestrated by a centralized VGG-16 control block that manages DDR3 traffic, AXI interconnects, and layer-wise scheduling.

```
CNN_A → BRAM_1A → BRAM_2A
CNN_B → BRAM_1B → BRAM_2B
CNN_C → BRAM_1C → BRAM_2C
CNN_D → BRAM_1D → BRAM_2D
```

- **72 parallel MAC units per engine** (8 × MAC-9 modules) → **288 MACs/cycle** across all 4 engines
- **200 MHz** clock, all four engines fully pipelined
- **512-bit AXI4** fabric connecting the CNN engines, DDR3 (via Xilinx MIG), AXI SmartConnect, AXI DataMover, and AXI BRAM Controller
- **Standalone MicroSD weight loading** via AXI Quad SPI — zero host/software dependency
- Hierarchical multi-FSM control: **Main FSM, MAC Control FSM, Accumulation FSM, and Max-Pool FSM**, all synchronized across the 200 MHz domain

### Core Engineering Insight: Minimizing DDR3 Traffic

The central design principle of this accelerator is **not** maximizing raw parallelism — it's minimizing off-chip DDR3 access, since DDR3 bandwidth (not compute) is the real bottleneck at this scale.

- From **Block 1 to Block 4**, feature map channels are streamed directly from DDR3 → BRAM, since keeping all channels on-chip simultaneously isn't possible at these resolutions.
- After **Block 4's max-pool**, output shrinks to 28×32×512 — small enough that its ~202 BRAM blocks worth of data can be cached fully on-chip, eliminating further DDR3 round-trips for Blocks 4–5. This single optimization is what allows the design to close the gap against much larger, power-hungry prior implementations.
- **DDR3 burst behavior is explicitly modeled in the datapath**: each burst delivers 4096 bytes with an inter-burst gap (2-clock minimum, designed conservatively for 4 clocks) — this timing was accounted for directly in the FSM design rather than treated as an afterthought.
- The core CNN engine is designed to accept **any row size**, with the constraint that **column size must be divisible by 8** — enabling the engine to handle every VGG-16 feature map size (224→112→56→28→14→7) through a single reusable, parameterized RTL block rather than per-layer custom hardware.

---

##  Verification Approach

- Full **post-synthesis simulation** across every convolution block (Conv1.1 through Conv5.3), with measured per-layer latency matching hand-calculated theoretical values (e.g., Conv1.1: 4.07 ms measured vs. 4.07 ms calculated).
- **Layer-wise output correctness validated** via Python (OpenCV) — pixel extraction and feature map reconstruction confirmed against expected values, including a full edge-detection test on a real 224×224×3 RGB image.
- **Post-implementation timing closure**: 0 failing endpoints across 71,424 timing paths at 200 MHz (WNS: 0.063 ns).
- DDR3 read/write burst behavior verified at the waveform level to confirm correct AXI protocol compliance under back-to-back access patterns.

---

## 📊 Post-Implementation Report

| Resource | Utilization | Available | % |
|---|---|---|---|
| LUT | 98,735 | 303,600 | 32.52% |
| LUTRAM | 5,409 | 130,800 | 4.14% |
| FF | 77,375 | 607,200 | 12.74% |
| BRAM | 388 | 1,030 | 37.67% |
| DSP | 300 | 2,800 | 10.71% |

**Timing:** WNS 0.063 ns, 0 failing endpoints across 71,424 paths — all constraints met.
**Power:** 4.932 W total on-chip, 4.625 W dynamic, junction temp 30.6°C.

---

## 📸 Architecture & Implementation Visuals

*(Paste your images into the `Experimental_Screenshot_images/` folder using these filenames, or update the paths below to match your actual filenames.)*

![VGG-16 Full Network Architecture](Experimental_Screenshot_images/vgg16_layer_architecture.png)
![Standard Convolution Operation Concept](Experimental_Screenshot_images/convolution_operation_concept.png)
![Internal Structure of Single MAC Unit](Experimental_Screenshot_images/mac_unit_internal_structure.png)
![Inline Buffer Streaming Architecture](Experimental_Screenshot_images/inline_buffer_streaming.png)
![BRAM2 Accumulation Process](Experimental_Screenshot_images/bram_accumulation_process.png)
![Max-Pooling Dataflow Architecture](Experimental_Screenshot_images/maxpool_dataflow.png)
![Full CNN Accelerator RTL Block (Vivado Schematic)](Experimental_Screenshot_images/cnn_accelerator_rtl_schematic.png)
![Full VGG-16 Dataflow with 4 CNN Engines](Experimental_Screenshot_images/vgg16_4engine_dataflow.png)
![Post-Implementation Utilization, Timing & Power Report](Experimental_Screenshot_images/post_implementation_report.png)
![Edge Detection Output Validation (224×224×3 RGB)](Experimental_Screenshot_images/edge_detection_validation.png)

## 📂 Repository Structure

| Folder / File | Description |
|---|---|
| [`MTP_Paper_VGG16_DDR3_IEEE_format.pdf`](https://github.com/vijay-2020-vijay/FPGA-Based-VGG-16-CNN-Hardware-Accelerator-with-DDR3-in-Xilinx-Virtex-7-VC707/blob/main/MTP_Paper_VGG16_DDR3_IEEE_formet.pdf) | Full project paper in IEEE format |
| [`VGG16_DDR3.pdf`](https://github.com/vijay-2020-vijay/FPGA-Based-VGG-16-CNN-Hardware-Accelerator-with-DDR3-in-Xilinx-Virtex-7-VC707/blob/main/VGG16_DDR3.pdf) | Full A-to-Z documentation report — complete design, implementation, and result details |
| [`VGG16_DDR3_PPT.pdf`](https://github.com/vijay-2020-vijay/FPGA-Based-VGG-16-CNN-Hardware-Accelerator-with-DDR3-in-Xilinx-Virtex-7-VC707/blob/main/VGG16_DDR3_PPT.pdf) | Full project presentation — architecture, results, and literature comparison |
| `Experimental_Screenshot_images/` | Architecture diagrams, RTL schematics, and implementation reports |
| [`README.pdf`](https://github.com/vijay-2020-vijay/FPGA-Based-VGG-16-CNN-Hardware-Accelerator-with-DDR3-in-Xilinx-Virtex-7-VC707/blob/main/README.pdf) | Platform requirements and licensing notes — Vivado 2017 (licensed FPGA platform required) |
| `README.md` | Project documentation |

> ⚠️ **Platform Note:** This project was built and verified using a **licensed Xilinx Vivado 2017 installation**. A licensed FPGA platform/toolchain matching this version is required to reproduce synthesis and implementation results — see `README.pdf` for full platform and licensing requirements.

##  Designed for Reuse Beyond VGG-16

The core CNN computation engine is **fully parameterized** — not hardcoded to VGG-16 — supporting both standard and depthwise convolution:

- **Direct extensibility** to MobileNet-V2, YOLOv3, and YOLOv3-tiny through register-level configuration alone (no RTL changes required).
- **ResNet-18/50, GoogLeNet, and Inception-V4** support requires only extending the core engine to add 5×5/7×7 kernel support — the DDR3 interface, memory hierarchy, and FSM control fabric remain completely unchanged.

---

## 🔮 Future Work

| Direction | Expected Impact |
|---|---|
| **1024-bit AXI bus** (double current width) | Enables 8 parallel channels simultaneously; projected 184.5 ms latency, 167 GOP/s |
| **INT8 Quantization** | Same throughput gains as above, with saturating accumulators to manage overflow risk |
| **Line-buffer streaming** | Fetch multiple pixels/cycle to feed more engines in parallel; reduces LUT usage |
| **On-chip memory optimization** | Maximize on-chip caching to further reduce DDR3 dependency (FPGA); current ASIC baseline is already highly competitive |

---

## 📚 References

1. Y.-H. Chen, T. Krishna, J. Emer, V. Sze, "Eyeriss: An Energy-Efficient Reconfigurable Accelerator for Deep Convolutional Neural Networks," *IEEE JSSC*, vol. 52, no. 1, pp. 127–138, Jan. 2017.
2. K. Simonyan, A. Zisserman, "Very Deep Convolutional Networks for Large-Scale Image Recognition," *ICLR*, 2015.
3. K. He, X. Zhang, S. Ren, J. Sun, "Deep Residual Learning for Image Recognition," *IEEE CVPR*, pp. 770–778, 2016.
4. A. Howard et al., "Searching for MobileNetV3," *IEEE/CVF ICCV*, pp. 1314–1324, 2019.
5. M. Tan, Q. V. Le, "EfficientNet: Rethinking Model Scaling for CNNs," *ICML*, pp. 6105–6114, 2019.
6. Xilinx Inc., "Virtex-7 FPGA Family Data Sheet DS183." [Link](https://www.xilinx.com/support/documentation/data_sheets/ds183_Virtex_7_Data_Sheet.pdf)

*(Full reference list with all literature comparison entries available in the project presentation.)*

---

## 👤 Author

**Rabisankar Maity**
VLSI M.Tech Graduate, IIT Guwahati — AIR 318 in GATE EC
Feel free to connect, raise issues, or fork this project for your own experiments.
