# **Analog AI Weight Encoding and Reliability: A Comprehensive Analysis of Advanced Implementation Methodologies**

## **1\. Introduction: The Analog Encoding Paradigm Shift**

The fundamental premise of Analog In-Memory Computing (AIMC) is the physical realization of mathematical operations. Unlike digital architectures, where a weight is a symbolic representation stored as a binary string and processed by a distinct arithmetic logic unit (ALU), AIMC embeds the weight directly into the physical properties of the memory device itself—specifically, its conductance ($G$). This shift from symbolic processing to physical processing promises to dismantle the Von Neumann bottleneck, offering orders-of-magnitude improvements in energy efficiency and compute density.1 However, this transition necessitates a radical reimagining of how neural network parameters are encoded, stored, and maintained. The digital abstraction of a deterministic, immutable "weight" must be replaced with a nuanced understanding of analog states that are inherently stochastic, time-variant, and subject to device-specific non-idealities.  
The challenge of weight encoding in AIMC is not merely a matter of Digital-to-Analog conversion; it is a complex discipline involving material science, circuit theory, and algorithmic compensation. The storage medium—whether Phase Change Memory (PCM), Resistive Random Access Memory (ReRAM), or Flash—introduces noise sources such as random telegraph noise (RTN), conductance drift, and cycle-to-cycle variability that fundamentally degrade the precision of the computation.3 Consequently, the successful deployment of analog AI accelerators relies less on the raw physics of the memory device and more on the architectural "wrapper" of encoding schemes, bit-slicing topologies, and write-verification protocols that envelop these noisy components to extract reliable computational fidelity.  
This report provides an exhaustive analysis of these encoding methodologies. It dissects the transition from simple differential pairs to complex multi-level cell (MLC) architectures, explores the necessity of bit-slicing for high-precision inference, and details the advanced variation-resilient schemes such as VECOM and SWIPE designed to mitigate hardware faults. Furthermore, it examines the critical peripheral circuitry—specifically the evolution of write-termination circuits and readout interfaces—that enables these encoding strategies to function at scale.

## **2\. Physics of the Substrate: Material Constraints on Encoding**

To understand the encoding architectures, one must first characterize the physical substrate upon which these weights are inscribed. The encoding strategy is dictated by the specific non-idealities of the underlying Non-Volatile Memory (NVM) technology.

### **2.1 The Stochastic Nature of Conductance**

In an ideal analog accelerator, a weight $W$ would map linearly to a conductance $G$ such that $G \= G\_{min} \+ \\alpha W$. However, the physical reality is governed by the equation:

$$G\_{device}(t, V, T) \= G\_{target} \+ \\Delta G\_{write} \+ \\Delta G\_{drift}(t) \+ \\Delta G\_{noise}(V, T)$$  
Each term in this equation represents a distinct barrier to precision that the encoding scheme must overcome:

* **Write Noise ($\\Delta G\_{write}$):** This is the deviation of the programmed conductance from the target value immediately after the write operation. In ReRAM, this arises from the stochastic nature of oxygen vacancy generation and filament formation.5 In PCM, it stems from the variability in the volume of the amorphous plug created during the melt-quench process.7 This noise forces encoding schemes to adopt iterative programming or "write-verify" loops to achieve target accuracy.  
* **Conductance Drift ($\\Delta G\_{drift}(t)$):** Predominantly an issue in PCM, this represents the temporal evolution of the programmed state. As the amorphous chalcogenide material undergoes structural relaxation, its resistance increases (conductance decreases) following a power law, causing the stored weight to "fade" over time.8 Encoding schemes for PCM must therefore include dynamic compensation mechanisms, such as Reference Cell Conductance Tracking (RCCT).10  
* **Read Noise ($\\Delta G\_{noise}$):** Also known as Random Telegraph Noise (RTN) or thermal noise, this affects the sensing of the weight during inference. In ReRAM, current fluctuations through narrow conductive filaments can be significant, limiting the effective number of distinguishable levels (bits) per device.4

### **2.2 Device-Specific Encoding Profiles**

The choice of device dictates the optimal encoding strategy.  
**Phase Change Memory (PCM):** PCM devices, such as those based on Germanium-Antimony-Tellurium (GST), offer a large dynamic range and high density. However, due to conductance drift, absolute encoding (where a specific conductance value maps to a specific weight) is perilous. Differential encoding and drift-resilient schemes are mandatory.7 IBM’s demonstration of a Transformer model on PCM hardware highlights that despite these imperfections, near software-equivalent accuracy is achievable through rigorous calibration and noise-aware training.1  
**Resistive RAM (ReRAM/OxRAM):** ReRAM operates by forming conductive filaments through an oxide layer. It offers the highest switching speeds (sub-10ns), making it suitable for online training.11 However, the stochasticity of filament rupture leads to "stuck-at" faults and high variability. Encoding schemes for ReRAM often favor binary or ternary representations with aggressive bit-slicing to mitigate the risk of relying on unstable intermediate states.13  
**Flash Memory (Floating Gate):** Flash memory, specifically NOR Flash, utilizes the charge stored on a floating gate to modulate the threshold voltage ($V\_{th}$) of a transistor. By operating in the subthreshold regime, Flash can achieve extremely high precision (up to 8 bits equivalent) with very low read currents.15 The encoding challenge here is less about noise and more about the non-linearity of the I-V characteristics and the high voltages required for programming, which complicates the peripheral circuitry.17

## **3\. Fundamental Weight Mapping Architectures**

The most basic unit of analog storage is the mapping of a digital weight to an analog state. This mapping is rarely 1-to-1; it involves sophisticated arrangements of devices to handle signed arithmetic and common-mode noise.

### **3.1 Differential Pair Encoding**

Neural network weights can be positive or negative, but physical conductance is strictly positive. To resolve this, the industry standard is **Differential Pair Encoding**, where a logical weight $W$ is represented by the difference in conductance between two physical devices, $G^+$ and $G^-$.7

$$W \\propto G^+ \- G^-$$  
This topology provides several critical advantages for reliability:

1. **Zero-Point Stability:** A weight of zero is represented when $G^+ \\approx G^-$. This eliminates the need for a device to achieve a perfect "off" state ($G\_{min}$), which is often leaky in resistive technologies.19 The "offset current" from imperfect devices cancels out, provided the mismatch between the pair is minimal.  
2. **Common-Mode Rejection Ratio (CMRR):** Global variations that affect the entire array—such as temperature fluctuations or power supply ripples—tend to influence adjacent devices similarly. By reading the difference, these common-mode errors are subtracted, significantly improving the Signal-to-Noise Ratio (SNR).20  
3. **Sign Agility:** Weights can transition from positive to negative without a complex erase cycle; the controller simply increases the conductance of $G^-$ relative to $G^+$.5

However, differential encoding comes with a **50% density penalty** (two devices per weight). Advanced research suggests using a single "reference" column for an entire row of weights to save area, though this reintroduces vulnerability to spatial variability.10

### **3.2 Linear vs. Logarithmic Mapping**

The translation of the digital weight value to the analog target $G\_{target}$ typically follows one of two regimes:  
**Linear Mapping:** Weights are mapped to equidistant conductance levels. This is the standard approach for maximizing noise margins when the noise distribution is uniform across the dynamic range. It simplifies the digital-to-analog converters (DACs) used for input activations, as the multiplication $I \= V \\cdot G$ remains linear.6  
**Logarithmic/Non-Linear Mapping:** Certain devices, particularly Flash memory operating in the subthreshold region, exhibit an exponential relationship between the control parameter (gate voltage) and the output (current).17 Furthermore, the distribution of weights in deep neural networks often follows a bell curve (Gaussian or Laplacian), with many small weights and few large ones. Some advanced encoding schemes propose mapping weights to logarithmically spaced conductance levels. This provides higher precision for small weights (where sensitivity is high) and lower precision for large weights, aligning the hardware precision with the information density of the neural network.22

## **4\. Bit-Slicing: Decoupling Neural Precision from Device Fidelity**

As deep learning models increasingly demand 8-bit, 16-bit, or even floating-point precision, the limitations of single analog devices become apparent. The "Effective Number of Bits" (ENOB) for most ReRAM and PCM devices is limited to 3-5 bits due to noise and variability.1 To bridge this gap, **Bit-Slicing** is employed.

### **4.1 The Architecture of Bit-Slicing**

Bit-slicing decomposes a high-precision digital weight into multiple lower-precision "slices," each stored in a separate analog device. For example, an 8-bit weight might be distributed across four 2-bit cells (carrying bits \[0-1\], \[2-3\], \[4-5\], \[6-7\]) or eight 1-bit cells.24  
The analog currents from these devices are read out separately, converted to digital values by Analog-to-Digital Converters (ADCs), and then recombined in the digital domain using shift-and-add logic:

$$W\_{total} \= \\sum\_{k=0}^{K-1} 2^{k \\cdot M} \\cdot ADC(I\_{slice\\\_k})$$  
where $K$ is the number of slices and $M$ is the bits per cell. This architecture allows the system to achieve arbitrary precision regardless of the individual device limitations, trading off area and energy for accuracy.

### **4.2 Significance Error and MSB Vulnerability**

While bit-slicing solves the precision capacity problem, it introduces a new reliability challenge: **Significance Error**. In a bit-sliced architecture, not all devices are equal. A fault or noise spike in the device storing the Least Significant Bits (LSB) has a negligible impact on the total weight value. However, a similar error in the device storing the Most Significant Bits (MSB) can cause a catastrophic shift in the weight's value, potentially flipping the sign or maxing out the activation.20  
Research indicates that the SNR of the entire matrix-vector multiplication is dominated by the noise in the MSB cells. Therefore, uniform protection of all cells is inefficient; reliability resources must be concentrated on the MSBs.20

### **4.3 Unbalanced Bit-Slicing (UBS)**

To address the MSB vulnerability, researchers at IBM and other institutions have proposed **Unbalanced Bit-Slicing (UBS)**.19 This technique abandons the uniform distribution of bits across cells in favor of a hierarchical reliability structure.  
The Mechanism of UBS:  
Instead of storing 2 bits in every cell, UBS configures the cells responsible for the MSBs to store fewer bits (e.g., 1 bit), while LSB cells continue to store 2 or 3 bits.

* **Sensing Margin Expansion:** By reducing the bit density of the MSB cell to 1 bit (Binary mode), the system utilizes only the High Conductance State (HCS) and Low Conductance State (LCS). This maximizes the "sensing margin"—the gap between states—making the MSB highly resilient to read noise, drift, and variability.27  
* **Integration with 2's Complement:** UBS is frequently implemented with 2's complement arithmetic. The MSB slice, often reduced to a single bit, serves as the sign bit. By ensuring this specific bit has the highest possible noise margin, the system effectively eliminates "sign reversal" errors, which are the most damaging to inference accuracy.19

Performance Impact:  
Simulation results on ReRAM crossbars demonstrate that UBS can achieve up to 8.8x higher accuracy compared to standard Balanced Bit-Slicing (BBS) and can tolerate significantly higher conductance variations.27 The energy overhead is minimal, as the reduction in MSB density is offset by the reduced need for error correction and re-transmission.

### **4.4 Significance-Based Weight Mapping**

Beyond slicing structure, the physical placement of bits can be optimized. **Significance-Based Mapping** involves placing MSB slices in the physical rows or columns of the crossbar that exhibit the best electrical characteristics (e.g., lowest IR drop, closest to the sense amplifiers).25  
Two specific algorithms have emerged to handle "Stuck-At-Faults" (SAF) in this context:

1. **Sign-Flip:** If a column has a high number of devices stuck at '1' (low resistance), and the target weights for that column are mostly '0', the controller can mathematically invert the sign of the weights ($\\mathbf{W} \\rightarrow \-\\mathbf{W}$) and the inputs, allowing the physical '1's to represent logical '0's (conceptually). This simple inversion can "hide" a significant percentage of hardware defects.25  
2. **Bit-Flip:** This offers finer granularity by selectively inverting individual bit slices. If an MSB slice has a fault, the system can invert the encoding for that specific slice to align the fault with the target value, drastically reducing the error magnitude.25

## **5\. Variation-Resilient Encoding Schemes**

The previous sections discussed static mapping. Advanced architectures employ dynamic encoding schemes that actively compensate for device variations during the mapping process.

### **5.1 VECOM: Variation-Resilient Encoding and Offset Compensation**

**VECOM** (Variation-Resilient Encoding and Offset Compensation) is a comprehensive framework designed specifically for ReRAM-based DNN accelerators.30 It addresses two critical reliability issues: conductance variation and the High Resistance State (HRS) offset current.  
The Offset Current Challenge:  
In an ideal crossbar, a cell in the "off" state (HRS) would have zero conductance and contribute zero current. In reality, ReRAM devices in HRS have a finite, non-zero conductance. When summing currents along a bitline with hundreds of cells, these small leakage currents accumulate, creating a massive "offset" that can saturate the ADC and reduce the dynamic range available for the actual signal (the "on" cells).30  
**The VECOM Solution:**

1. **Weight Pattern Analysis:** VECOM analyzes the trained weights of the DNN to identify distribution patterns.  
2. **Resilient Encoding:** It re-encodes weights to minimize the usage of states that are most prone to variation.  
3. **Offset Compensation:** The core innovation is a compensation scheme where the aggregate offset current is estimated or measured and then subtracted. This is often implemented by programming a "negative" offset conductance into a dedicated row or by subtracting a calibrated value in the digital domain after the ADC. Unlike simple fixed-bias subtraction, VECOM’s compensation is granular and accounts for the specific variation profile of the tile.32

Results:  
By effectively removing the offset floor, VECOM allows for larger crossbar arrays (more wordlines activated simultaneously) without saturation. This has been shown to improve throughput by up to 9.1x and reduce energy consumption by 50% by minimizing the need for segmented readouts.32

### **5.2 SWIPE: Single-Write In-Memory Program-Verify**

**SWIPE** is a programming-time encoding optimization.20 Standard programming treats every cell as independent. SWIPE recognizes that in a bit-sliced weight, the errors are correlated.  
Mechanism:  
SWIPE writes the bit slices sequentially, starting from the MSB.

1. The MSB slice is programmed.  
2. The system reads the *actual* conductance achieved by the MSB (which includes some error $\\epsilon$).  
3. Instead of spending time and energy to fix $\\epsilon$ in the MSB cell (which is difficult), SWIPE **adjusts the target value of the next slice (LSB)** to compensate. The error is "forwarded" to the lower-significance slices where it can be corrected or where its impact is naturally lower.

This "Forward Error Correction" during programming reduces the number of write pulses required to achieve a specific aggregate precision, offering **5x to 10x** improvements in write energy and latency compared to blind iterative programming.20

## **6\. Device-Specific Reliability: Drift and Fabrication**

### **6.1 Phase Change Memory (PCM): The Drift Problem**

PCM is the leading candidate for high-density analog inference due to its multi-level capability. However, it suffers from **Resistance Drift**. The amorphous state of the chalcogenide glass is thermodynamically unstable and relaxes over time, causing the resistance to increase and the conductance to drop.8 This drift follows the law:

$$R(t) \= R\_0 \\left( \\frac{t}{t\_0} \\right)^\\nu$$  
where $\\nu$ is the drift coefficient. If left uncompensated, the weights effectively change value over time, destroying accuracy within hours.  
Hardware Compensation: Reference Cell Tracking (RCCT)  
To combat this, architectures employ Reference Cell Conductance Tracking (RCCT).10

* **Concept:** A subset of cells (Reference Columns) is programmed to known conductance states at the same time as the weight array.  
* **Operation:** Since the reference cells and weight cells are on the same die and experience the same thermal history, they drift at the same rate. The readout circuitry uses the current from the reference cells to bias the sense amplifiers or ADCs.  
* **Result:** As the weight conductance drops, the reference conductance drops proportionally. The *ratio* $I\_{weight} / I\_{ref}$ remains constant. This analog division effectively cancels out the temporal drift term, allowing PCM chips to maintain inference accuracy for extended periods (months/years) without reprogramming.34

Software Compensation:  
Alternatively, Global Drift Compensation (GDC) can be applied in the digital domain. The system tracks the time $t$ elapsed since programming and multiplies the digital output by a scalar factor $(t/t\_0)^\\nu$ to normalize the magnitude. This is computationally cheap but less precise than RCCT as it assumes a uniform $\\nu$ across the entire array.8

### **6.2 Flash Memory: The Voltage Trade-off**

Flash memory offers mature, highly reliable analog storage. The challenge lies in the **High Voltage** required for Fowler-Nordheim tunneling (programming), which is often \>10V.12 This is incompatible with standard logic processes (typically 1V-3V).  
Implementation Strategy:  
Flash-based AIMC often utilizes "U-shape recessed channels" or specific floating gate geometries to enhance coupling ratios, allowing for lower voltage operation or faster programming speeds (50ns vs 1ms).15 Furthermore, Flash cells are often operated in the subthreshold regime for inference. In this regime, the drain current is exponentially related to the gate voltage ($I\_d \\propto e^{V\_{gs}}$). This allows for computing over a massive dynamic range but requires highly stable temperature control, as the subthreshold slope is temperature-dependent.17

## **7\. Write and Verification Algorithms: The Art of Tuning**

The precision of the encoded weight is established during the programming phase. "Section 6" emphasizes the transition from open-loop writing to closed-loop, predictive tuning.

### **7.1 Iterative vs. Predictive Programming**

Incremental Step Pulse Programming (ISPP):  
The traditional approach is iterative: Apply a pulse $\\rightarrow$ Read Conductance $\\rightarrow$ If too low, increase pulse amplitude $\\rightarrow$ Repeat. This "Program-and-Verify" loop is precise but extremely slow, creating a bottleneck for training.35  
Predictive/Slope-Based Tuning:  
Advanced controllers use Predictive Algorithms.37

* **Mechanism:** The controller applies one or two sensing pulses to measure the device's "sensitivity" (the slope of the conductance change $\\Delta G / \\Delta V$).  
* **Prediction:** Using this slope, it calculates the exact pulse amplitude required to bridge the gap to the target conductance in a single shot.  
* **Benefit:** This reduces the write sequence from \~10-20 pulses to \~2-3 pulses, dramatically improving write latency for training workloads.

### **7.2 Self-Terminating Write (STW) Circuits**

The ultimate efficiency is achieved by moving the verification into the analog domain. **Self-Terminating Write (STW)** circuits integrate a comparator within the programming driver.38  
Mechanism:  
During the application of the write voltage, a current mirror monitors the cell current. As soon as the current reaches the target threshold (indicating the correct conductance has been reached), a feedback loop triggers a cutoff transistor that disconnects the write voltage.

* **Latency:** The write pulse is terminated instantly (nanosecond scale) upon success.  
* **Energy:** This prevents "over-programming" (wasting energy heating a cell that is already set) and protects the device from stress, enhancing endurance.  
* **Impact:** STW schemes have demonstrated **97% energy reduction** during the forming/SET process compared to fixed-width pulses.41

### **Table 1: Comparison of Write Mechanisms**

| Mechanism | Precision | Latency | Energy | Complexity | Best Application |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Open-Loop Pulse** | Low | Very Low | Low | Low | Binary/Ternary ReRAM |
| **Iterative (ISPP)** | Very High | High | High | Medium | Multi-level PCM Inference |
| **Predictive/Slope** | High | Medium | Medium | High (Logic) | Online Training |
| **Self-Terminating (STW)** | High | **Very Low** | **Very Low** | High (Circuit) | High-Efficiency Edge AI |

## **8\. Peripheral Circuitry: The ADC Bottleneck**

The data encoded in the array must eventually be read out. This interface—the Analog-to-Digital Converter (ADC)—is the primary bottleneck in AIMC, often consuming **60-80% of the total energy and area**.42

### **8.1 ADC Architectures**

The choice of ADC dictates the throughput and area efficiency of the encoding scheme.

* **Flash ADCs:** Use $2^N \- 1$ comparators. They are the fastest (1 clock cycle) but area scales exponentially. They are generally unsuitable for high precision (\>4 bits) in dense arrays.44  
* **SAR (Successive Approximation Register) ADCs:** Use a binary search algorithm with a single comparator and a DAC. They offer a good balance of area and power but are slower (N clock cycles per conversion). They are the dominant choice for medium precision (6-10 bits).45  
* **Shift-Add ADCs:** A novel architecture for bit-sliced arrays. Instead of converting the full sum, partial sums are digitized and shifted/added in the digital domain. This allows for lower precision ADCs to build up high precision results.47

### **8.2 ADC-Less Architectures**

To eliminate the ADC tax, "ADC-less" architectures are being developed.42

* **Hybrid Analog-Digital (HCiM):** These architectures split the computation. The heavy matrix multiplication happens in analog. The scaling factors and accumulations are handled by specialized digital logic that accepts low-precision (ternary) inputs directly from the array without a full ADC.42  
* **Charge-Domain Computing:** Some SRAM-based IMC designs use capacitors to accumulate charge. The result is read out via charge sharing, keeping the data in the analog domain for the next layer's input, effectively pipelining analog computations.49

## **9\. Security: Utilizing Encoding Noise**

Interestingly, the very flaws of analog encoding—process variation and stochasticity—are being repurposed for security. **Physical Unclonable Functions (PUFs)** utilize the unique, random breakdown patterns of ReRAM filaments or the random threshold voltage variations of Flash cells to create unique cryptographic keys.50  
RePACK Scheme:  
The RePACK scheme 50 uses these PUFs to encrypt the weights stored on the device. Since the encryption key is derived from the physical microstructure of the specific chip, the weights are chemically bound to the hardware. Cloning the digital content to another chip will fail because the physical "fingerprint" (the key) will not match. This turns the "bug" of D2D variability into a critical security "feature."

## **10\. Conclusion**

The analysis of "Section 6" reveals that **Analog AI Weight Encoding** is not a static process of writing values to memory, but a dynamic, multi-layered system of compensation and architectural co-design. The industry has moved beyond the naive utilization of raw memory devices. Current state-of-the-art implementations rely on **Unbalanced Bit-Slicing** to protect critical data, **Differential Encoding** to reject noise, and **Self-Terminating Write** circuits to manage energy and endurance.  
The future of this field lies in **Hardware-Software Co-Design**. Algorithms like VECOM and SWIPE demonstrate that the software stack must be aware of the underlying physics to correct errors that are impossible to fix at the device level. Furthermore, the integration of **Drift Compensation (RCCT)** in PCM and **Offset Compensation** in ReRAM has largely solved the retention and leakage issues that plagued early prototypes. As these encoding schemes mature, they enable AIMC to scale from experimental macros to commercial-grade accelerators capable of running Transformer models and large CNNs with digital-like accuracy and analog-like efficiency.

#### **Works cited**

1. Demonstration of transformer-based ALBERT model on a 14nm analog AI inference chip, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12485056/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12485056/)  
2. New algorithms open possibilities for training AI models on analog chips \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/blog/analog-in-memory-training-algorithms](https://research.ibm.com/blog/analog-in-memory-training-algorithms)  
3. Understanding Analog AI, Currecnt Challenges, and Prospects | by Hadi Saghir | Medium, accessed December 24, 2025, [https://medium.com/@hadisaghir00/understanding-analog-ai-currecnt-challenges-and-prospects-8673670512d1](https://medium.com/@hadisaghir00/understanding-analog-ai-currecnt-challenges-and-prospects-8673670512d1)  
4. Accelerating AI Applications using Analog In-Memory Computing: Challenges and Opportunities, accessed December 24, 2025, [https://par.nsf.gov/servlets/purl/10262138](https://par.nsf.gov/servlets/purl/10262138)  
5. Signal and noise extraction from analog memory elements for neuromorphic computing, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC5974407/](https://pmc.ncbi.nlm.nih.gov/articles/PMC5974407/)  
6. Open-loop analog programmable electrochemical memory array \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10550916/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10550916/)  
7. Accurate deep neural network inference using computational phase-change memory \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC7235046/](https://pmc.ncbi.nlm.nih.gov/articles/PMC7235046/)  
8. Decoding Algorithms and HW Strategies to Mitigate Uncertainties in a PCM-Based Analog Encoder for Compressed Sensing \- Unibo, accessed December 24, 2025, [https://cris.unibo.it/retrieve/ba666038-ccd8-4f6f-bc12-9d189b8f2ace/jlpea-13-00017.pdf](https://cris.unibo.it/retrieve/ba666038-ccd8-4f6f-bc12-9d189b8f2ace/jlpea-13-00017.pdf)  
9. Impact of Phase-Change Memory Nonidealities on Analog In-Memory Computing Deep Learning Accuracy for MRS Fall Meeting 2023 \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/publications/impact-of-phase-change-memory-nonidealities-on-analog-in-memory-computing-deep-learning-accuracy](https://research.ibm.com/publications/impact-of-phase-change-memory-nonidealities-on-analog-in-memory-computing-deep-learning-accuracy)  
10. A Readout Scheme for PCM-Based Analog In-Memory Computing With Drift Compensation Through Reference Conductance Tracking \- Unibo, accessed December 24, 2025, [https://cris.unibo.it/bitstream/11585/976134/3/A\_Readout\_Scheme\_for\_PCM-Based\_Analog\_In-Memory\_Computing\_With\_Drift\_Compensation\_Through\_Reference\_Conductance\_Tracking.pdf](https://cris.unibo.it/bitstream/11585/976134/3/A_Readout_Scheme_for_PCM-Based_Analog_In-Memory_Computing_With_Drift_Compensation_Through_Reference_Conductance_Tracking.pdf)  
11. RRAM vs PCM: Which Delivers Higher Write Speed? \- Patsnap Eureka, accessed December 24, 2025, [https://eureka.patsnap.com/report-rram-vs-pcm-which-delivers-higher-write-speed](https://eureka.patsnap.com/report-rram-vs-pcm-which-delivers-higher-write-speed)  
12. Resistive Random Access Memory (RRAM): an Overview of Materials, Switching Mechanism, Performance, Multilevel Cell (mlc) Storage, Modeling, and Applications \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC7176808/](https://pmc.ncbi.nlm.nih.gov/articles/PMC7176808/)  
13. TReMo+: Modeling Ternary and Binary ReRAM-Based Memories With Flexible Write-Verification Mechanisms \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/nanotechnology/articles/10.3389/fnano.2021.765947/full](https://www.frontiersin.org/journals/nanotechnology/articles/10.3389/fnano.2021.765947/full)  
14. A Survey of ReRAM-Based Architectures for Processing-In-Memory and Neural Networks, accessed December 24, 2025, [https://www.researchgate.net/publication/324774821\_A\_Survey\_of\_ReRAM-Based\_Architectures\_for\_Processing-In-Memory\_and\_Neural\_Networks](https://www.researchgate.net/publication/324774821_A_Survey_of_ReRAM-Based_Architectures_for_Processing-In-Memory_and_Neural_Networks)  
15. A Floating Gate Memory with U-Shape Recessed Channel for Neuromorphic Computing and MCU Applications \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2072-666X/10/9/558](https://www.mdpi.com/2072-666X/10/9/558)  
16. Time Domain Analog Neuromorphic Engine Based on High-Density Non-Volatile Memory in Single-Poly CMOS \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/iel7/6287639/9668973/09767835.pdf](https://ieeexplore.ieee.org/iel7/6287639/9668973/09767835.pdf)  
17. Flash Memory for Synaptic Plasticity in Neuromorphic Computing: A Review \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2313-7673/10/2/121](https://www.mdpi.com/2313-7673/10/2/121)  
18. Analog weight updates with compliance current modulation of binary ReRAMs for on-chip learning \- ZORA, accessed December 24, 2025, [https://www.zora.uzh.ch/200400/1/2020\_ISCAS\_ICC\_%283%29.pdf](https://www.zora.uzh.ch/200400/1/2020_ISCAS_ICC_%283%29.pdf)  
19. Accurate and Energy-Efficient Bit-Slicing for RRAM-Based Neural Networks \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/publications/accurate-and-energy-efficient-bit-slicing-for-rram-based-neural-networks](https://research.ibm.com/publications/accurate-and-energy-efficient-bit-slicing-for-rram-based-neural-networks)  
20. SWIPE: Enhancing Robustness of ReRAM Crossbars for In-memory ..., accessed December 24, 2025, [http://shanbhag.ece.illinois.edu/publications/SWIPE-ICCAD-2020.pdf](http://shanbhag.ece.illinois.edu/publications/SWIPE-ICCAD-2020.pdf)  
21. Neuromorphic Computing Using NAND Flash Memory Architecture With Pulse Width Modulation Scheme \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2020.571292/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2020.571292/full)  
22. Dynamic Precision Analog Computing for Neural Networks \- Queen's University, accessed December 24, 2025, [https://www.queensu.ca/physics/shastrilab/sites/shastwww/files/uploaded\_files/publications/journals/88\_Garg\_JSTQE\_Dynamic-precision\_2023.pdf](https://www.queensu.ca/physics/shastrilab/sites/shastwww/files/uploaded_files/publications/journals/88_Garg_JSTQE_Dynamic-precision_2023.pdf)  
23. Precision of bit slicing with in-memory computing based on analog phase-change memory crossbars \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/publications/precision-of-bit-slicing-with-in-memory-computing-based-on-analog-phase-change-memory-crossbars](https://research.ibm.com/publications/precision-of-bit-slicing-with-in-memory-computing-based-on-analog-phase-change-memory-crossbars)  
24. Exploring Bit-Slice Sparsity in Deep Neural Networks for Efficient ReRAM-Based Deployment, accessed December 24, 2025, [https://www.emc2-ai.org/assets/docs/neurips-19/emc2-neurips19-paper-12.pdf](https://www.emc2-ai.org/assets/docs/neurips-19/emc2-neurips19-paper-12.pdf)  
25. Weight Transformations in Bit-Sliced Crossbar Arrays for Fault Tolerant Computing-in-Memory \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2512.18459](https://arxiv.org/pdf/2512.18459)  
26. Weight Transformations in Bit-Sliced Crossbar Arrays for Fault Tolerant Computing-in-Memory: Design Techniques and Evaluation Framework \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2512.18459v1](https://arxiv.org/html/2512.18459v1)  
27. Delft University of Technology Unbalanced Bit-slicing Scheme for ..., accessed December 24, 2025, [https://research.tudelft.nl/files/96033660/SSD\_AICAS21.pdf](https://research.tudelft.nl/files/96033660/SSD_AICAS21.pdf)  
28. Accurate and Energy-Efficient Bit-Slicing for RRAM-Based Neural Networks \- TU Delft Research Portal, accessed December 24, 2025, [https://research.tudelft.nl/en/publications/accurate-and-energy-efficient-bit-slicing-for-rram-based-neural-n/](https://research.tudelft.nl/en/publications/accurate-and-energy-efficient-bit-slicing-for-rram-based-neural-n/)  
29. Accurate and Energy-Efficient Bit-Slicing for RRAM-Based Neural Networks \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/362283726\_Accurate\_and\_Energy-Efficient\_Bit-Slicing\_for\_RRAM-Based\_Neural\_Networks](https://www.researchgate.net/publication/362283726_Accurate_and_Energy-Efficient_Bit-Slicing_for_RRAM-Based_Neural_Networks)  
30. VECOM: Variation-Resilient Encoding and Offset Compensation Schemes for Reliable ReRAM-Based DNN Accelerator \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2312.11042v1](https://arxiv.org/html/2312.11042v1)  
31. Thai-Hoang Nguyen, accessed December 24, 2025, [https://thnguyen.netlify.app/](https://thnguyen.netlify.app/)  
32. VECOM: Variation-Resilient Encoding and Offset Compensation Schemes for Reliable ReRAM-Based DNN Accelerator \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/iel7/10323590/10323543/10323803.pdf](https://ieeexplore.ieee.org/iel7/10323590/10323543/10323803.pdf)  
33. Programming Schemes To Reduce Conductance Drift In Analog PCM Arrays, accessed December 24, 2025, [https://eureka.patsnap.com/report-programming-schemes-to-reduce-conductance-drift-in-analog-pcm-arrays](https://eureka.patsnap.com/report-programming-schemes-to-reduce-conductance-drift-in-analog-pcm-arrays)  
34. Phase Change Memory Drift Compensation in Spiking Neural Networks Using a Non-Linear Current Scaling Strategy \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9268/14/4/50](https://www.mdpi.com/2079-9268/14/4/50)  
35. Characterization and Programming Algorithm of Phase Change Memory Cells for Analog In-Memory Computing \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC8037667/](https://pmc.ncbi.nlm.nih.gov/articles/PMC8037667/)  
36. Accurate program/verify schemes of resistive switching memory (RRAM) for in-memory neural network circuits \- IRIS Re.Public@polimi.it, accessed December 24, 2025, [https://re.public.polimi.it/retrieve/e0c31c11-b93a-4599-e053-1705fe0aef77/TED\_2021\_rev\_v3\_nomarks.pdf](https://re.public.polimi.it/retrieve/e0c31c11-b93a-4599-e053-1705fe0aef77/TED_2021_rev_v3_nomarks.pdf)  
37. Programming Protocol Optimization for Analog Weight Tuning in Resistive Memories, accessed December 24, 2025, [https://asu.elsevierpure.com/en/publications/programming-protocol-optimization-for-analog-weight-tuning-in-res/](https://asu.elsevierpure.com/en/publications/programming-protocol-optimization-for-analog-weight-tuning-in-res/)  
38. Self-Terminating Write of Multi-Level Cell ReRAM for Efficient Neuromorphic Computing \- Fangxin Liu's Homepage, accessed December 24, 2025, [https://mxhx7199.github.io/files/%5BDATE-22%5DSTW\_preprint.pdf](https://mxhx7199.github.io/files/%5BDATE-22%5DSTW_preprint.pdf)  
39. (PDF) Write Termination Circuits for RRAM: A Holistic Approach From Technology to Application Considerations \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/342023589\_Write\_Termination\_Circuits\_for\_RRAM\_A\_Holistic\_Approach\_From\_Technology\_to\_Application\_Considerations](https://www.researchgate.net/publication/342023589_Write_Termination_Circuits_for_RRAM_A_Holistic_Approach_From_Technology_to_Application_Considerations)  
40. Density Enhancement of RRAMs using a RESET Write Termination for MLC Operation \- DATE 2019, accessed December 24, 2025, [https://past.date-conference.com/proceedings-archive/2021/pdf/1702.pdf](https://past.date-conference.com/proceedings-archive/2021/pdf/1702.pdf)  
41. Switching event detection and self-termination programming circuit for energy efficient ReRAM memory arrays \- Infoscience \- EPFL, accessed December 24, 2025, [https://infoscience.epfl.ch/bitstreams/b4929e98-cc2e-485c-9db3-cb37c63b7891/download](https://infoscience.epfl.ch/bitstreams/b4929e98-cc2e-485c-9db3-cb37c63b7891/download)  
42. HCiM: ADC-Less Hybrid Analog-Digital Compute in Memory Accelerator for Deep Learning Workloads \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2403.13577v1](https://arxiv.org/html/2403.13577v1)  
43. Pruning for Improved ADC Efficiency in Crossbar-based Analog In-memory Accelerators, accessed December 24, 2025, [https://arxiv.org/html/2403.13082v1](https://arxiv.org/html/2403.13082v1)  
44. Types Of ADCs \- Monolithic Power Systems, accessed December 24, 2025, [https://www.monolithicpower.com/en/learning/mpscholar/analog-to-digital-converters/introduction-to-adcs/types-of-adcs](https://www.monolithicpower.com/en/learning/mpscholar/analog-to-digital-converters/introduction-to-adcs/types-of-adcs)  
45. Review of Analog-To-Digital Conversion Characteristics and Design Considerations for the Creation of Power-Efficient Hybrid Data Converters \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9268/8/2/12](https://www.mdpi.com/2079-9268/8/2/12)  
46. Understanding SAR ADCs: Their Architecture and Comparison with Other ADCs, accessed December 24, 2025, [https://www.analog.com/en/resources/technical-articles/successive-approximation-registers-sar-and-flash-adcs.html](https://www.analog.com/en/resources/technical-articles/successive-approximation-registers-sar-and-flash-adcs.html)  
47. Analog-to-Digital Converter Design Exploration for Compute-in-Memory Accelerators \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/ielaam/6221038/9722756/9319225-aam.pdf](https://ieeexplore.ieee.org/ielaam/6221038/9722756/9319225-aam.pdf)  
48. ADC-Less Reprogrammable RRAM Array Architecture for In-Memory Computing, accessed December 24, 2025, [http://ieeexplore.ieee.org/document/10289643/](http://ieeexplore.ieee.org/document/10289643/)  
49. Charge-Domain Static Random Access Memory-Based In-Memory Computing with Low-Cost Multiply-and-Accumulate Operation and Energy-Efficient 7-Bit Hybrid Analog-to-Digital Converter \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9292/13/3/666](https://www.mdpi.com/2079-9292/13/3/666)  
50. Physical unclonable in-memory computing for simultaneous protecting private data and deep learning models \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11762733/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11762733/)  
51. Tamper-sensitive pre-formed ReRAM-based PUFs: Methods and experimental validation, accessed December 24, 2025, [https://www.frontiersin.org/journals/nanotechnology/articles/10.3389/fnano.2022.1055545/full](https://www.frontiersin.org/journals/nanotechnology/articles/10.3389/fnano.2022.1055545/full)