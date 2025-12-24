# **Advanced Architectures for Analog AI Weight Encoding: Programming, Precision, and Reliability**

## **1\. Introduction: The Analog Imperative and the Weight Encoding Challenge**

The trajectory of modern artificial intelligence, particularly the exponential scaling of Deep Neural Networks (DNNs) and Large Language Models (LLMs), has precipitated a fundamental collision with the physical constraints of digital computing architectures. The conventional Von Neumann architecture, predicated on the physical separation of processing units and memory, faces an insurmountable "memory wall." In state-of-the-art digital accelerators, the energy and latency costs associated with shuttling synaptic weights and activation data between Dynamic Random Access Memory (DRAM) and arithmetic logic units (ALUs) dominate the system's power budget, often consuming orders of magnitude more energy than the computation itself.1 As models swell to trillions of parameters, this data movement bottleneck threatens to stifle further advancements in AI deployment, particularly in energy-constrained edge environments.4  
Analog In-Memory Computing (AIMC) has emerged as the definitive architectural response to this crisis. By embedding computation directly within the memory arrays, AIMC eliminates the need to move weight data, thereby circumventing the Von Neumann bottleneck. This paradigm leverages the physical properties of non-volatile memory (NVM) devices—specifically Phase-Change Memory (PCM), Resistive Random Access Memory (RRAM), and Flash—to execute Matrix-Vector Multiplication (MVM), the foundational kernel of all deep learning, in the analog domain.1 In this scheme, weights are not stored as abstract binary codes but are instantiated as physical conductance values ($G$) within the memory cells. By applying input activations as voltages ($V$) along the rows of a crossbar array, the system exploits Ohm’s Law ($I \= V \\cdot G$) to perform multiplication and Kirchhoff’s Current Law to sum the resulting currents ($I\_{total} \= \\Sigma I$) along the columns, performing a massively parallel MAC operation in a single time step.1  
However, the transition from the deterministic precision of digital logic to the stochastic reality of analog physics introduces a profound set of challenges. Analog weights are not static integers; they are dynamic physical states subject to thermal noise, structural relaxation, cycle-to-cycle variability, and non-linear programming characteristics. "Section 7" of the foundational roadmap reports on Analog AI is widely recognized in the field as the definitive technical treatise on these challenges. It delineates the sophisticated engineering required to encode mathematical abstractions into imperfect physical devices. This report provides an exhaustive, expert-level analysis of these advanced weight encoding methodologies, programming algorithms, and reliability compensation techniques. We explore how differential pair architectures, significance encoding, and novel algorithms like Gradient Descent Programming (GDP) and Tiki-Taka are revolutionizing the fidelity of analog computation, enabling it to rival digital precision while delivering superior energy efficiency.8

## **2\. The Physics of Analog Weight Storage**

To understand the complexity of weight encoding, one must first appreciate the physical substrates that store these values. Unlike digital memory, which abstracts material physics into discrete binary states ("0" or "1"), analog AI exposes the raw material properties of the device to the computational process. The "weight" is a continuous variable defined by the atomic configuration of the memory cell.

### **2.1 Phase-Change Memory (PCM): The Kinetic Sculptor**

Phase-Change Memory (PCM) stands as a frontrunner for high-performance analog inference and training accelerators, particularly in designs pioneered by IBM Research.2 Its operation relies on the reversible phase transition of chalcogenide glass, typically Germanium-Antimony-Tellurium ($Ge\_2Sb\_2Te\_5$ or GST), which exhibits a dramatic contrast in electrical resistivity between its ordered crystalline phase and its disordered amorphous phase.  
Mechanism of Analog Storage:  
The encoding of an analog weight in PCM is achieved through a process of "partial RESET." The device begins in a low-resistance crystalline state (SET). To encode a specific weight (conductance), a high-amplitude, short-duration current pulse is applied to the device. This pulse generates Joule heat, melting a specific volume of the chalcogenide material directly above the heater electrode. When the pulse is abruptly cut off, the molten material quenches rapidly, freezing into a highly resistive amorphous plug.6  
The size of this amorphous plug determines the effective conductance of the device. By carefully modulating the amplitude and trailing edge of the programming pulse, the system can control the volume of the amorphous region, thereby creating a continuum of resistance states intermediate between fully crystalline and fully amorphous. This "volume fraction" modulation allows a single PCM cell to store multiple bits of information in an analog format.2  
Thermodynamic Instability:  
The fundamental challenge with PCM lies in the thermodynamics of the amorphous state. This disordered phase is metastable; the atoms possess higher free energy than in the crystalline lattice. Over time, the atomic structure undergoes "structural relaxation," locally rearranging to minimize Gibbs free energy. This relaxation increases the bandgap and the trap density of the material, leading to a spontaneous, monotonic increase in resistance over time. This phenomenon, known as conductance drift, is the primary source of error in PCM-based AI systems.10 The conductance $G(t)$ typically evolves according to a power law, necessitating the complex compensation schemes that will be detailed in Section 5\.

### **2.2 Resistive RAM (RRAM): The Filamentary Bridge**

Resistive Random Access Memory (RRAM) offers an alternative pathway, particularly favored for low-power edge applications due to its compatibility with standard CMOS back-end-of-line (BEOL) processes and its potential for 3D integration.12 Architectures like NeuRRAM leverage RRAM to achieve high energy efficiency.14  
Filamentary Switching Mechanism:  
RRAM devices typically consist of a metal-insulator-metal (MIM) structure, where the insulating layer is a transition metal oxide such as Hafnium Oxide ($HfO\_2$) or Tantalum Oxide ($TaO\_x$). The switching mechanism is driven by the formation and rupture of conductive filaments within this oxide layer. When a strong electric field is applied (forming or SET process), oxygen ions are displaced from the lattice, leaving behind oxygen vacancies. These vacancies, which are positively charged, migrate and cluster to form a conductive path (filament) bridging the top and bottom electrodes.15  
Analog Modulation via Geometry:  
Analog weight encoding in RRAM is achieved by modulating the geometry of this filament. A SET pulse can widen the filament or increase the density of vacancies within it, thereby increasing the conductance. Conversely, a RESET pulse promotes the recombination of vacancies with oxygen ions (often stored in an interface capping layer), effectively narrowing or rupturing the filament and decreasing conductance.13  
Stochasticity and Noise:  
Unlike the bulk volume switching of PCM, RRAM switching involves the movement of discrete ions and vacancies. This process is inherently stochastic, governed by hopping barriers and thermal fluctuations. Consequently, RRAM exhibits significant Random Telegraph Noise (RTN) and cycle-to-cycle variability. A device programmed to a specific state may spontaneously jump to a neighboring state due to the trapping or detrapping of a single electron or the diffusion of a vacancy. This stochasticity poses a rigorous challenge for precise weight encoding, requiring robust "program-and-verify" algorithms and differential encoding schemes to average out the noise.12

### **2.3 Flash Memory: The Tunable Transistor**

While typically associated with high-density digital storage, Flash memory has been successfully repurposed for analog compute-in-memory, most notably by companies like Mythic.7  
Subthreshold Analog Operation:  
In this architecture, the standard floating-gate transistor is operated in the subthreshold regime. In this region, the drain current ($I\_{ds}$) bears an exponential relationship to the gate voltage ($V\_{gs}$) and the threshold voltage ($V\_{th}$):

$$I\_{ds} \\propto e^{\\kappa(V\_{gs} \- V\_{th})}$$

The "weight" of the synapse is stored as the charge (number of electrons) on the floating gate, which directly modulates the threshold voltage $V\_{th}$. By precisely controlling this charge via Fowler-Nordheim tunneling or hot-carrier injection, the transistor acts as a high-precision voltage-controlled current source.7  
Precision vs. Endurance:  
Flash memory cells can achieve remarkable precision, with Mythic demonstrating the ability to tune a single cell to 256 distinct levels (equivalent to 8-bit precision).17 This high density is a significant advantage over PCM and RRAM, which typically struggle to achieve more than 4-5 bits of reliable storage per cell. Furthermore, Flash exhibits excellent retention and negligible drift compared to PCM. However, the high voltages required for programming degrade the tunnel oxide over time, limiting the endurance of Flash to approximately $10^4$ to $10^5$ cycles. This makes Flash ideal for inference-only applications where weights are programmed once and read many times, but less suitable for on-chip training workloads that require frequent weight updates.18

## **3\. Advanced Weight Encoding Architectures**

The simplistic mapping of one mathematical weight to one physical device is rarely sufficient for high-accuracy deep learning. Neural network weights are signed values (positive and negative) and often require high dynamic range. "Section 7" of the architectural roadmap focuses heavily on the aggregation strategies—combining multiple devices to form a robust, high-fidelity "synaptic unit."

### **3.1 Differential Pair Encoding: The Industry Standard**

Physical conductance is a strictly non-negative quantity ($G \\ge 0$). However, the synaptic weights in a neural network can be positive (excitatory) or negative (inhibitory). To resolve this fundamental mismatch, analog accelerators almost universally adopt a **differential pair** architecture.  
Operational Mechanism:  
In this scheme, each logical weight $W\_{ij}$ is represented by the difference between two physical conductance values, $G\_{ij}^+$ and $G\_{ij}^-$, stored in two separate memory cells or crossbar columns.1

$$W\_{ij} \\propto G\_{ij}^+ \- G\_{ij}^-$$

During the inference operation (MVM), the input activation voltage $V\_j$ is applied simultaneously to the word-lines of both devices.

* The current through the positive device ($I^+ \= V\_j \\cdot G\_{ij}^+$) is collected on a "positive" bit-line.  
* The current through the negative device ($I^- \= V\_j \\cdot G\_{ij}^-$) is collected on a "negative" bit-line.  
* A differential readout circuit (such as a differential current sensing amplifier or a subtraction circuit in the digital periphery) computes the net current $I\_{out} \= I^+ \- I^-$, thereby realizing the signed multiplication operation.2

**Benefits of Differential Encoding:**

1. **Native Sign Representation:** It naturally accommodates both positive and negative weights without complex DC biasing schemes. A weight of zero, which is common in sparse networks, is robustly represented by setting $G^+ \\approx G^-$.  
2. **Common-Mode Rejection:** This is perhaps the most critical advantage. Global variations that affect the entire chip—such as fluctuations in ambient temperature, power supply voltage ripple, or substrate noise—tend to affect adjacent devices in the differential pair similarly. Because the output is the *difference* between the two currents, these common-mode errors cancel out. For instance, if temperature rise causes the conductance of both PCM devices to increase by 5%, the differential weight remains relatively stable, significantly improving the Signal-to-Noise Ratio (SNR).22  
3. **Symmetry Mitigation:** It helps mitigate the asymmetry in device programming. If increasing conductance (SET) is easier or more precise than decreasing it (RESET), the system can increase $G^-$ to effectively decrease the net weight $W$, avoiding the difficult RESET regime.8

### **3.2 Significance Encoding and Bit-Slicing**

The precision of a single analog memory device is fundamentally limited by noise (shot noise, thermal noise, RTN). Most RRAM and PCM devices can reliably store only 4 to 5 bits of information (16 to 32 distinct levels). To run modern quantized neural networks which often require 8-bit (INT8) or even higher precision, architectures employ **Significance Encoding**, also known as **Bit-Slicing**.  
Architectural Implementation:  
Rather than attempting to force a single noisy device to distinguish between 256 levels, the weight is decomposed into multiple "slices" of lower precision. For example, an 8-bit weight can be split into two 4-bit slices: a Most Significant Pair (MSP) and a Least Significant Pair (LSP).24

$$W\_{total} \= F \\cdot (G\_{MSP}^+ \- G\_{MSP}^-) \+ (G\_{LSP}^+ \- G\_{LSP}^-)$$

Here, $F$ is the significance factor (e.g., $F=16$ for a 4-bit split). The MSP devices encode the coarse value of the weight (the upper 4 bits), while the LSP devices encode the fine detail (the lower 4 bits).  
Readout and Recombination:  
During the MVM operation, the currents from the MSP and LSP columns are integrated separately. They are then digitized by Analog-to-Digital Converters (ADCs). The digital output of the MSP ADC is bit-shifted (multiplied by 16\) and added to the output of the LSP ADC in the digital accumulation unit.  
Trade-offs and Optimization:  
This approach effectively decouples the system-level precision from the device-level limitations. It allows an accelerator to achieve 8-bit or 16-bit accuracy using devices that are natively only 4-bit capable. However, this comes at the cost of area and energy: it requires multiple devices per weight (often 4 devices for a signed 8-bit weight) and additional ADCs. Advanced optimization strategies, such as Unbalanced Bit-Slicing, allocate more physical resources or larger dynamic ranges to the MSP to minimize errors in the most critical bits, as an error in the MSP is far more detrimental to accuracy than an error in the LSP.24

### **3.3 Case Study: Mythic’s Analog Flash Architecture**

Mythic’s architecture represents a divergent philosophy in weight encoding. Instead of using multiple low-precision devices, Mythic engineers the standard NOR Flash cell to act as a high-precision variable resistor.7  
8-Bit Single Cell Density:  
By leveraging the mature fabrication processes of Flash memory and operating in the subthreshold regime, Mythic claims to achieve 8-bit equivalent precision (256 levels) within a single memory cell. This is achieved through an iterative programming process that finely tunes the charge on the floating gate. This high density allows Mythic to pack up to 80 million weights on a single chip, eliminating the need for the area-intensive bit-slicing techniques required by PCM and RRAM.17  
Digital-Analog Hybrid:  
To maintain this precision, Mythic employs a hybrid approach. While the core matrix multiplication occurs in the analog domain, the input vectors are fed via high-precision Digital-to-Analog Converters (DACs), and the outputs are captured by high-speed ADCs. Digital SIMD (Single Instruction, Multiple Data) units and RISC-V processors are integrated on the same die to handle non-linear activation functions, pooling, and other operations that are difficult to implement accurately in analog. This architecture treats the analog flash array as a specialized, ultra-dense arithmetic co-processor.27

## **4\. The Write Path: Programming Paradigms and Algorithms**

Writing analog weights is fundamentally more complex than writing digital bits. In digital memory, a "1" is a wide bucket of charge; essentially, any charge above a certain threshold is a valid "1". In analog memory, a specific conductance value is a tiny target on a continuous sliding scale. "Section 7" delves into the sophisticated algorithms required to hit these targets despite the stochastic behavior of the devices.

### **4.1 Iterative Program-and-Verify (Closed-Loop)**

The baseline standard for precision programming in analog AI is the **Iterative Program-and-Verify** loop. This method treats the programming of each device as a feedback control problem.  
**The Algorithm:**

1. **Pulse:** A programming pulse (SET or RESET) is applied to the device.  
2. **Verify (Read):** The conductance of the device is read to determine its current state.  
3. **Compare:** The measured conductance is compared to the target weight value.  
4. **Correction:** If the error exceeds a predefined tolerance ($\\epsilon$), a corrective pulse is applied. The amplitude or width of this pulse is often scaled based on the magnitude of the error (proportional control).  
5. **Repeat:** This cycle repeats until the device converges to the target state or a timeout is reached.8

Latency and Scalability Issues:  
While effective for achieving high precision, this method is excruciatingly slow. It effectively serializes the write operation, requiring multiple read-write cycles for every single device. For a neural network with millions of parameters, the total programming time can be substantial. This latency penalty is the primary reason why many analog AI chips are designed as "weight stationary" inference engines—the weights are programmed once and then used for millions of inference operations without modification.9 Furthermore, in PCM, the "verify" read must often be delayed to account for immediate conductance relaxation (short-term drift), further slowing the process.

### **4.2 Gradient Descent-Based Programming (GDP): The Blind Update**

To address the latency bottleneck and the hardware complexity of measuring single-device conductance, researchers have proposed **Gradient Descent-Based Programming (GDP)**, often referred to as "blind" or "direct minimization" updates.9  
Conceptual Shift:  
GDP reframes the programming challenge. Instead of attempting to tune each resistor individually to a theoretical target value (which requires high-precision, large-area ADCs for every column), GDP tunes the matrix operation itself. It optimizes the array to produce the correct output current, regardless of the individual device states.  
**The Procedure:**

1. **Synthetic Input:** A batch of random input vectors is fed into the crossbar array.  
2. **Forward Pass:** The MVM is performed on the hardware, generating an output vector $Y\_{chip}$.  
3. **Error Calculation:** This output is compared to the *ideal* mathematical output $Y\_{target}$ (calculated digitally) to determine the error $E \= Y\_{target} \- Y\_{chip}$.  
4. **Gradient Calculation:** The gradients of the error with respect to the weights ($\\partial E / \\partial W$) are calculated in the digital periphery.  
5. **Blind Update:** Programming pulses proportional to these gradients are applied to the entire array in parallel. Crucially, the system does **not** verify individual device conductances.

Advantages of GDP:  
This method offers profound benefits. First, it is significantly faster because it utilizes the parallel MVM capability of the array for the "read" step, rather than reading devices serially. Second, and perhaps more importantly, it inherently compensates for circuit non-idealities. If a specific column has a systematic offset due to wire resistance (IR drop) or ADC non-linearity, the gradient descent process will automatically adjust the weights in that column to compensate, effectively "learning" the hardware defects. Experimental results on PCM-based cores have shown that GDP can improve inference accuracy (e.g., \+1.26% on ResNet-9) compared to iterative programming while eliminating the need for complex single-device read circuitry.9

### **4.3 The Tiki-Taka Algorithm for On-Chip Training**

For applications requiring **on-chip training**, where weights must be updated continuously based on live data, the asymmetry of device switching poses a fatal problem. In real devices (PCM/RRAM), increasing conductance (SET) often follows a different curve and requires different energy than decreasing conductance (RESET). Standard Stochastic Gradient Descent (SGD) assumes symmetric updates; applying it to asymmetric devices leads to "weight saturation," where weights get stuck at the minimum or maximum values, stalling learning. The **Tiki-Taka** algorithm was developed to solve this.8  
Dual-Matrix Architecture:  
Tiki-Taka addresses asymmetry by decoupling the noisy gradient accumulation from the stable weight storage. It utilizes two coupled matrices:

1. **Matrix A (Accumulator):** This array is responsible for the fast, frequent updates from each training batch. It is allowed to be noisy and asymmetric. It acts as a short-term memory buffer.  
2. **Matrix C (Weights):** This array stores the long-term weights of the model. It is updated much less frequently.

The "Tiki-Taka" Update Rule:  
During training, gradients are accumulated in Matrix A. After a certain number of batches, a "transfer" operation is performed. The accumulated value in Matrix A is read, scaled, and used to update Matrix C via a single, carefully controlled pulse. Matrix A is then reset (or decays). This separation of timescales filters out the high-frequency noise and asymmetry of the raw gradient updates, allowing the "master" weights in Matrix C to evolve smoothly along the loss landscape.29 The advanced version, Tiki-Taka v2 (TTv2), incorporates digital filtering to further relax the hardware requirements, enabling successful training even on extremely noisy devices with limited states.31

## **5\. Reliability Engineering: Taming the Analog Chaos**

The physical reality of analog memory is hostile to precision computing. States drift over time, reading a value can disturb it, and thermal noise is ubiquitous. Without the mitigation strategies detailed in "Section 7," analog inference would degrade to uselessness within hours of programming.

### **5.1 Conductance Drift Compensation**

In Phase-Change Memory (PCM), conductance drift is the most insidious enemy. As the amorphous material relaxes into a lower-energy state, the resistance increases, causing the stored weight values to effectively "fade" toward zero over time.  
The Physics of Drift:  
The conductance $G(t)$ of a PCM cell evolves according to a power law relationship:

$$G(t) \= G(t\_0) \\cdot \\left(\\frac{t}{t\_0}\\right)^{-\\nu}$$

where $t\_0$ is the time of programming and $\\nu$ is the drift coefficient, which depends on the specific material composition and the volume of the amorphous plug.10 If uncorrected, this drift causes the distribution of weights to broaden and shift, destroying the accuracy of the neural network. In experimental setups, accuracy can drop by over 5% within a month.10  
Global Drift Compensation (GDC):  
Fortunately, drift is deterministic in its direction (resistance always increases) and relatively uniform across the array. This allows for a highly effective mitigation strategy known as Global Drift Compensation (GDC).

* **Mechanism:** The system periodically reads a subset of reference cells (or the average conductance of the array) to estimate the current magnitude of the drift.  
* **Correction:** It calculates a time-dependent gain factor $\\alpha(t) \= G\_{initial} / G\_{current}$. This factor is applied digitally to the output of the ADCs. Effectively, as the analog signal "fades," the digital "volume" is turned up to compensate.  
* **Efficacy:** This simple recalibration technique has been shown to recover inference accuracy to within 1% of the baseline even after 30 days of drift.10

Drift-Aware Training:  
More advanced strategies incorporate drift modeling directly into the training process. By artificially "drifting" the weights during the forward pass of the training simulation, the neural network is forced to learn a set of weights that are robust to magnitude scaling. This flattens the loss landscape, creating a solution that remains valid even as the physical weights decay.34

### **5.2 Read Disturb Mitigation**

In RRAM and Flash, the very act of reading the memory cell can alter its state, a phenomenon known as **Read Disturb**.  
RRAM Ion Migration:  
When a read voltage is applied across a filamentary RRAM device, it generates an electric field that can induce slight ion migration. Over millions of read cycles, this can gradually change the shape of the filament, effectively "soft programming" the device and altering the stored weight. This limits the inference lifetime of the accelerator.  
Bipolar Reading:  
To combat this, architectures employ Bipolar Reading. Instead of always applying a positive read voltage ($+V\_{read}$), the system alternates the polarity of the read pulse (e.g., $+V\_{read}$ for the first inference, $-V\_{read}$ for the second). This alternating field pushes and pulls the ions in opposite directions, canceling out the net migration and significantly extending the stability of the filament.35  
Flash Disturbance:  
In Flash memory, high pass-through voltages on unselected word-lines can cause weak tunneling of electrons, slowly shifting the threshold voltage of unselected cells. While digital SSDs handle this with Error Correction Codes (ECC), analog AI chips—which lack the overhead for heavy ECC—must mitigate this through circuit design (operating at lower voltages) and architectural partitioning (limiting the number of cells sharing a bit-line).36

### **5.3 Hardware-Aware (HWA) Training**

The single most potent defense against analog non-idealities is **Hardware-Aware Training (HWA)**. This methodology acknowledges that the hardware is imperfect and incorporates those imperfections into the training loop.1  
Noise Injection and Quantization:  
During the training phase (typically performed on a GPU), noise is injected into the weights and activation functions during the forward pass. This noise is statistically modeled to match the profile of the target hardware (e.g., Gaussian read noise, log-normal programming variability, drift noise).37 Furthermore, weights are quantized (clipped and rounded) to match the limited dynamic range and resolution of the analog devices.  
Robust Optimization:  
By training in the presence of this simulated noise, the optimization algorithm (SGD or Adam) is forced to find a "wide, flat minimum" in the loss landscape. In this region, small perturbations to the weights—whether caused by drift, thermal noise, or programming errors—do not result in a significant increase in error. HWA effectively "immunizes" the model against the specific flaws of the analog hardware.39

## **6\. Comparative Technology Analysis and Roadmap**

The choice of memory technology dictates the specific encoding strategies and limiting factors of the AI accelerator. Table 1 synthesizes the comparative data for the three dominant technologies as of 2024-2025.

### **6.1 Technology Comparison Table**

| Metric | Phase-Change Memory (PCM) | Resistive RAM (RRAM) | Analog Flash (NOR) |
| :---- | :---- | :---- | :---- |
| **Physical Mechanism** | Amorphous/Crystalline Phase Transition (Volume Fraction) 6 | Filament Formation/Rupture (Geometry Modulation) 13 | Floating Gate Charge Storage (Threshold Voltage Shift) 7 |
| **Weight Encoding** | Analog Conductance (Partial RESET) | Analog Conductance (Filament Width) | Subthreshold $V\_{th}$ Modulation |
| **Primary Non-Ideality** | **Conductance Drift** (Structural Relaxation) 10 | **Stochasticity/RTN** (Ion Hopping) 15 | **Endurance** (Oxide Degradation) 18 |
| **Drift Coefficient ($\\nu$)** | High (\~0.01 \- 0.1) 10 | Low (Filament is thermodynamically stable) | Low (Excellent charge retention) 19 |
| **Write Latency** | Medium (Iterative required for precision) | Fast (\~10 \- 100 ns) 13 | Slow (High voltage tunneling, $\\mu$s-ms) |
| **Endurance** | High ($10^8 \- 10^9$ cycles) 40 | High ($10^6 \- 10^{12}$ cycles) 40 | Low ($10^4 \- 10^5$ cycles) 18 |
| **Linearity** | Poor (Asymmetric SET/RESET) | Medium (Material dependent, can be abrupt) | High (Incremental step pulse programming) |
| **Energy per MAC** | Low (In-memory accumulation) | Ultralow (Lower write voltages) 41 | Low (Subthreshold currents) |
| **Typical Architecture** | Differential Pair / 4-PCM per weight | Differential Pair / Voltage-Mode Sensing | Single Cell (8-bit equivalent) / Digital-Analog Hybrid |
| **Best Use Case** | Inference & Training (High endurance allows updates) | Low-Power Edge Inference (IoT, Mobile) | High-Density Inference (Data Center / Edge) |

### **6.2 Future Outlook**

The roadmap for Analog AI, as projected by "Section 7" developments, points toward distinct evolutionary paths.

* **Short Term:** We are seeing the commercialization of **Inference-Only** analog chips (e.g., Mythic, IBM’s Hermes prototype). These devices rely on "weight stationarity," where the model is programmed once (mitigating Flash's low endurance or PCM's programming latency) and drift is managed via periodic global calibration.  
* **Medium Term:** To compete with the relentless scaling of digital memory, Analog AI must go vertical. **3D Vertical RRAM (VRRAM)** and 3D PCM arrays are in development to rival the density of 3D NAND Flash. This density is a prerequisite for storing massive Transformer models (LLMs) entirely on-chip, which is the "killer app" for eliminating the memory bottleneck.12  
* **Long Term:** The holy grail is **On-Chip Training**. This requires solving the linearity and symmetry challenges of the write update. Research is shifting toward novel materials like **Electrochemical RAM (ECRAM)** and Li-Ion synaptic transistors, which offer naturally linear, symmetric, and low-energy updates by decoupling the read and write paths. Combined with robust algorithms like Tiki-Taka v2, these technologies could enable edge devices that learn continuously from their environment.31

In conclusion, weight encoding in Analog AI is not merely a static data format; it is a dynamic, active process of managing entropy. From the "blind" gradient updates that optimize the system over the device, to the drift compensation loops that fight the second law of thermodynamics, these technologies represent a sophisticated co-design of physics, circuit engineering, and algorithmic intelligence. The success of Analog AI depends less on discovering a "perfect" ideal memory device and more on the perfection of these compensatory architectures that allow imperfect devices to perform intelligent computation.

#### **Works cited**

1. Analog AI — IBM Analog Hardware Acceleration Kit 1.0.0 documentation, accessed December 24, 2025, [https://aihwkit.readthedocs.io/en/latest/analog\_ai.html](https://aihwkit.readthedocs.io/en/latest/analog_ai.html)  
2. The hardware behind analog AI \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/blog/the-hardware-behind-analog-ai](https://research.ibm.com/blog/the-hardware-behind-analog-ai)  
3. The Promise of Analog AI: Could In-Memory Computing Revolutionize Edge Devices?, accessed December 24, 2025, [https://runtimerec.com/the-promise-of-analog-ai-could-in-memory-computing-revolutionize-edge-devices/](https://runtimerec.com/the-promise-of-analog-ai-could-in-memory-computing-revolutionize-edge-devices/)  
4. Roadmap to Neuromorphic Computing with Emerging Technologies \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2407.02353v1](https://arxiv.org/html/2407.02353v1)  
5. Edge Artificial Intelligence: A Systematic Review of Evolution, Taxonomic Frameworks, and Future Horizons \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2510.01439v1](https://arxiv.org/html/2510.01439v1)  
6. An energy-efficient analog chip for AI inference \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/blog/analog-ai-chip-inference](https://research.ibm.com/blog/analog-ai-chip-inference)  
7. Analog Compute-in-Memory at Mythic, accessed December 24, 2025, [https://knowen-production.s3.amazonaws.com/uploads/attachment/file/5192/Analog%2BCompute%2Bat%2BMythic.pdf](https://knowen-production.s3.amazonaws.com/uploads/attachment/file/5192/Analog%2BCompute%2Bat%2BMythic.pdf)  
8. All-in-One Analog AI Accelerator: On-Chip Training and Inference with Conductive-Metal-Oxide/HfOx ReRAM Devices \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2502.04524v1](https://arxiv.org/html/2502.04524v1)  
9. Gradient descent-based programming of analog in-memory ... \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2305.16647](https://arxiv.org/pdf/2305.16647)  
10. Demonstration of transformer-based ALBERT model on a 14nm analog AI inference chip, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12485056/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12485056/)  
11. Programming Schemes To Reduce Conductance Drift In Analog PCM Arrays, accessed December 24, 2025, [https://eureka.patsnap.com/report-programming-schemes-to-reduce-conductance-drift-in-analog-pcm-arrays](https://eureka.patsnap.com/report-programming-schemes-to-reduce-conductance-drift-in-analog-pcm-arrays)  
12. Resistive Switching Random-Access Memory (RRAM): Applications and Requirements for Memory and Computing | Chemical Reviews \- ACS Publications, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acs.chemrev.4c00845](https://pubs.acs.org/doi/10.1021/acs.chemrev.4c00845)  
13. RRAM vs PCM: Which Delivers Higher Write Speed? \- Patsnap Eureka, accessed December 24, 2025, [https://eureka.patsnap.com/report-rram-vs-pcm-which-delivers-higher-write-speed](https://eureka.patsnap.com/report-rram-vs-pcm-which-delivers-higher-write-speed)  
14. A compute-in-memory chip based on resistive random-access memory \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC9385482/](https://pmc.ncbi.nlm.nih.gov/articles/PMC9385482/)  
15. RRAM-Based Analog Matrix Computing for Massive MIMO Signal Processing \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2512.04365](https://arxiv.org/pdf/2512.04365)  
16. Analog AI Startup Mythic To Compute And Scale In Flash \- WikiChip Fuse, accessed December 24, 2025, [https://fuse.wikichip.org/news/2755/analog-ai-startup-mythic-to-compute-and-scale-in-flash/](https://fuse.wikichip.org/news/2755/analog-ai-startup-mythic-to-compute-and-scale-in-flash/)  
17. M1076 Analog Matrix Processor \- Mythic AI, accessed December 24, 2025, [https://mythic.ai/products/m1076-analog-matrix-processor/](https://mythic.ai/products/m1076-analog-matrix-processor/)  
18. Types of NAND Flash: Everything You Need to Know \- Flexxon, accessed December 24, 2025, [https://www.flexxon.com/types-of-nand-flash-everything-you-need-to-know/](https://www.flexxon.com/types-of-nand-flash-everything-you-need-to-know/)  
19. NOR Or NAND Flash Memory. Which Is Better? \- Aesonlabs Data Recovery Systems, accessed December 24, 2025, [https://aesonlabs.ca/blogs/nor-or-nand-flash-memory-which-is-better/](https://aesonlabs.ca/blogs/nor-or-nand-flash-memory-which-is-better/)  
20. Toward Software-Equivalent Accuracy on Transformer-Based Deep Neural Networks With Analog Memory Devices \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/computational-neuroscience/articles/10.3389/fncom.2021.675741/full](https://www.frontiersin.org/journals/computational-neuroscience/articles/10.3389/fncom.2021.675741/full)  
21. An analog-AI chip for energy-efficient speech recognition and transcription \- PMC, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10447234/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10447234/)  
22. Rules For Handling Differential Signals \- EE Times, accessed December 24, 2025, [https://www.eetimes.com/rules-for-handling-differential-signals/](https://www.eetimes.com/rules-for-handling-differential-signals/)  
23. The Why and How of Differential Signaling \- Technical Articles \- All About Circuits, accessed December 24, 2025, [https://www.allaboutcircuits.com/technical-articles/the-why-and-how-of-differential-signaling/](https://www.allaboutcircuits.com/technical-articles/the-why-and-how-of-differential-signaling/)  
24. Unbalanced Bit-slicing Scheme for Accurate Memristor-based Neural Network Architecture \- TU Delft Research Portal, accessed December 24, 2025, [https://research.tudelft.nl/files/96033660/SSD\_AICAS21.pdf](https://research.tudelft.nl/files/96033660/SSD_AICAS21.pdf)  
25. Precision of bit slicing with in-memory computing based on analog phase-change memory crossbars \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/358184090\_Precision\_of\_bit\_slicing\_with\_in-memory\_computing\_based\_on\_analog\_phase-change\_memory\_crossbars](https://www.researchgate.net/publication/358184090_Precision_of_bit_slicing_with_in-memory_computing_based_on_analog_phase-change_memory_crossbars)  
26. Optimised weight programming for analogue memory-based deep neural networks, accessed December 24, 2025, [https://www.researchgate.net/publication/361640231\_Optimised\_weight\_programming\_for\_analogue\_memory-based\_deep\_neural\_networks](https://www.researchgate.net/publication/361640231_Optimised_weight_programming_for_analogue_memory-based_deep_neural_networks)  
27. Mythic's analog matrix processor is launched to the market, injecting a big heart into AI, accessed December 24, 2025, [https://en.eeworld.com.cn/news/qrs/eic517659.html](https://en.eeworld.com.cn/news/qrs/eic517659.html)  
28. Programming Weights to Analog In-Memory Computing Cores by Direct Minimization of the Matrix-Vector Multiplication Error \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/publications/programming-weights-to-analog-in-memory-computing-cores-by-direct-minimization-of-the-matrix-vector-multiplication-error](https://research.ibm.com/publications/programming-weights-to-analog-in-memory-computing-cores-by-direct-minimization-of-the-matrix-vector-multiplication-error)  
29. Algorithm for Training Neural Networks on Resistive Device Arrays \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/1909.07908](https://arxiv.org/pdf/1909.07908)  
30. Analog Resistive Switching Devices for Training Deep Neural Networks with the Novel Tiki-Taka Algorithm \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10811689/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10811689/)  
31. Enabling Training of Neural Networks on Noisy Hardware \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2021.699148/full](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2021.699148/full)  
32. Optimized Weight Programming for Analogue Memory-based Deep Neural Networks \- Semantic Scholar, accessed December 24, 2025, [https://pdfs.semanticscholar.org/2c1c/18961f934a07e7b5c9778199d28964089658.pdf](https://pdfs.semanticscholar.org/2c1c/18961f934a07e7b5c9778199d28964089658.pdf)  
33. Phase Change Memory Drift Compensation in Spiking Neural Networks Using a Non-Linear Current Scaling Strategy \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9268/14/4/50](https://www.mdpi.com/2079-9268/14/4/50)  
34. Paper 6121 Detail Information \- Epapers, accessed December 24, 2025, [https://epapers2.org/aicas2025/ESR/paper\_details.php?paper\_id=6121](https://epapers2.org/aicas2025/ESR/paper_details.php?paper_id=6121)  
35. Investigation of Read Disturb and Bipolar Read Scheme on Multilevel RRAM based Deep Learning Inference Engine \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/ielaam/16/9098120/9069908-aam.pdf](https://ieeexplore.ieee.org/ielaam/16/9098120/9069908-aam.pdf)  
36. Read Disturb Errors in MLC NAND Flash Memory: Characterization, Mitigation, and Recovery \- Carnegie Mellon University, accessed December 24, 2025, [https://users.ece.cmu.edu/\~omutlu/pub/flash-read-disturb-errors\_dsn15.pdf](https://users.ece.cmu.edu/~omutlu/pub/flash-read-disturb-errors_dsn15.pdf)  
37. Analog Hardware-aware Training, accessed December 24, 2025, [https://aihwkit.readthedocs.io/en/latest/hwa\_training.html](https://aihwkit.readthedocs.io/en/latest/hwa_training.html)  
38. The marriage of training and inference for scaled deep learning analog hardware | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/339262495\_The\_marriage\_of\_training\_and\_inference\_for\_scaled\_deep\_learning\_analog\_hardware](https://www.researchgate.net/publication/339262495_The_marriage_of_training_and_inference_for_scaled_deep_learning_analog_hardware)  
39. Closing the accuracy gap between analog and digital AI \- Research Communities, accessed December 24, 2025, [https://communities.springernature.com/posts/closing-the-accuracy-gap-between-analog-and-digital-ai](https://communities.springernature.com/posts/closing-the-accuracy-gap-between-analog-and-digital-ai)  
40. Current Opinions on Memristor-Accelerated Machine Learning Hardware \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2501.12644v1](https://arxiv.org/html/2501.12644v1)  
41. RRAM vs Traditional Memory: Efficiency and Energy Use \- Patsnap Eureka, accessed December 24, 2025, [https://eureka.patsnap.com/report-rram-vs-traditional-memory-efficiency-and-energy-use](https://eureka.patsnap.com/report-rram-vs-traditional-memory-efficiency-and-energy-use)  
42. Resistive Memory-based Neural Differential Equation Solver for Score-based Diffusion Model \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2404.05648v1](https://arxiv.org/html/2404.05648v1)  
43. Resistive memory-based analog synapse: The pursuit for linear and symmetric weight update \- Arizona State University, accessed December 24, 2025, [https://asu.elsevierpure.com/en/publications/resistive-memory-based-analog-synapse-the-pursuit-for-linear-and-/](https://asu.elsevierpure.com/en/publications/resistive-memory-based-analog-synapse-the-pursuit-for-linear-and-/)