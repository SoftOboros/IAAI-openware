# **Standards, Metrology, and the Regulatory Landscape for Analog AI Weight Encoding**

## **1\. Introduction: The Deterministic Past and the Probabilistic Future**

The semiconductor industry currently stands at a pivotal juncture, transitioning from the rigid determinism of von Neumann architectures to the probabilistic efficiency of Analog In-Memory Computing (AIMC). For over five decades, the industry has operated on a foundation of binary certainty. A bit stored in a Dynamic Random Access Memory (DRAM) cell or a Static Random Access Memory (SRAM) flip-flop is, by definition and design, a deterministic entity. It is either a logic '0' or a logic '1'. The entire ecosystem of standards, metrology, and regulation—from JEDEC specifications to ISO safety certifications—has been constructed around the axiom that hardware errors are anomalies to be eliminated, not intrinsic features to be managed.  
However, the rise of Analog AI challenges this fundamental axiom. In this paradigm, the synaptic "weight" of a neural network is not encoded as a string of binary digits but as a physical state variable—the conductance of a filament in a Resistive RAM (ReRAM) cell, the amorphous-crystalline ratio in Phase Change Memory (PCM), or the magnetic orientation in a Magnetic Tunnel Junction (MTJ).1 These physical states are inherently analog and subject to the stochastic laws of thermodynamics and quantum mechanics. A weight encoded in a memristor is not a static value; it is a distribution, susceptible to thermal drift, random telegraph noise (RTN), and device-to-device variability.  
This shift necessitates a comprehensive re-evaluation of the industrial framework governing computing hardware. The metrology required to characterize a stochastic analog weight differs fundamentally from the dimensional metrology used for digital logic. The standards required to ensure interoperability between analog chiplets and digital hosts must account for signal noise and calibration protocols that have no equivalent in the DDR (Double Data Rate) world. Furthermore, the regulatory landscape is shifting rapidly with the introduction of legislation such as the European Union’s Artificial Intelligence Act (EU AI Act), which mandates "robustness" and "accuracy" for high-risk AI systems.3 For Analog AI to achieve commercial viability in safety-critical sectors like automotive and healthcare, the industry must reconcile the probabilistic nature of the hardware with the legal and safety requirements of the human world.  
This report provides an exhaustive analysis of the standards, metrology, and regulatory landscape for Analog AI weight encoding as of 2024-2025. It synthesizes data from the International Roadmap for Devices and Systems (IRDS), the National Institute of Standards and Technology (NIST), ISO/IEC standardization committees, and cutting-edge industrial research to provide a roadmap for navigating this complex transition.

## ---

**2\. The Metrology of Analog Weights: From Geometry to Function**

Metrology, the science of measurement, is the bedrock of semiconductor manufacturing. In the era of Moore’s Law, metrology was primarily concerned with geometry: measuring Critical Dimensions (CD), overlay accuracy, and line edge roughness. As the industry moves "Beyond CMOS" into the realm of neuromorphic and analog computing, the focus of metrology is expanding from measuring *structure* to measuring *function* and *state*.

### **2.1. The Physics of Uncertainty and the IRDS Roadmap**

The 2024 edition of the International Roadmap for Devices and Systems (IRDS) Metrology Chapter identifies a critical inflection point. The roadmap highlights that for emerging devices, "stochastic variations... can now create small variations in pattern fidelity that have substantial effects on device performance".4 In digital logic, these variations are defects. In Analog AI, they are often the defining characteristic of the computational substrate.  
The core measurand in Analog AI is conductance ($G$). Unlike a digital bit, which has expansive noise margins, an analog weight occupies a continuum. The metrological challenge lies in characterizing the Probability Density Function (PDF) of this conductance over time and temperature.  
**Key Metrological Challenges Identified by IRDS 2024:**

* **Stochasticity in Nanoscale Patterning:** The roadmap notes that Extreme Ultraviolet (EUV) lithography induces stochastic variations in resist films.5 For a ReRAM device where the filament diameter might be only a few nanometers, these stochastic variations in the defining geometry translate directly into variations in the programming voltage ($V\_{set}/V\_{reset}$) and the resulting conductance state. Metrology tools must therefore evolve from providing single-point measurements to providing statistical distributions of device parameters across a wafer.4  
* **3D Metrology for Vertical Structures:** To achieve the density required for large language models (LLMs), Analog AI is moving towards 3D vertical crossbar arrays (VRRAM). The IRDS roadmap emphasizes the difficulty of measuring critical dimensions and material properties in these "fully functional tiers" without destructive analysis. High-energy X-ray and tomographic techniques are identified as potential solutions to image devices buried deep within a 3D stack.4  
* **Material Characterization:** The integration of novel materials is a cornerstone of Analog AI. The IRDS predicts that 2D materials like molybdenum disulfide ($MoS\_2$) will be inserted into high-volume manufacturing within the next decade.7 However, the roadmap warns that "a one-size-fits-all metrology solution does not exist yet" for these materials.8 Standardized spectroscopic methods are required to measure the stoichiometry, strain, and defect density of these atomic-layer films, as these factors directly influence the linearity and symmetry of the weight update rules in analog learning.

### **2.2. NIST and the Quest for "Synthetic Cortical Function"**

The National Institute of Standards and Technology (NIST) has taken a proactive role in defining the metrology for neuromorphic systems. Their research extends beyond characterizing individual devices to measuring the behavior of large-scale, interconnected spiking networks. The ultimate goal, as described by the NIST Magnetic Imaging Group, is to develop metrology analogous to "functional MRI" (fMRI) for chips—techniques that can visualize the flow of information across a neuromorphic processor in real-time.9  
Magnetic Josephson Junctions (MJJs) as Metrological Standards:  
NIST researchers are developing Magnetic Josephson Junctions (MJJs) to serve as adaptive synapses in superconducting neural networks. These devices operate at speeds a billion times faster than biological neurons and with switching energies as low as 1 aJ ($10^{-18}$ J).9

* **The Measurement Challenge:** Measuring signals at the attajoule level requires fundamentally new metrological instruments. Traditional oscilloscopes and probes introduce too much thermal noise. NIST is developing superconducting single-photon detectors and advanced magnetic imaging techniques to detect the minute electromagnetic signatures of these synaptic events.10  
* **Stochastic Metrology:** For devices operating near the thermal limit, such as superparamagnetic tunnel junctions used in probabilistic computing, the output is inherently random. NIST is developing statistical verification protocols. Instead of checking if a device switched from '0' to '1', the metrology verifies that the *distribution* of switching events matches the expected Boltzmann distribution dictated by the energy barrier.10 This "stochastic metrology" is essential for validating the hardware used in Bayesian inference and generative AI models.

### **2.3. Characterizing Conductance Drift and Noise**

The commercial viability of Analog AI depends on the ability to retain a weight value over time. In Phase Change Memory (PCM), the physical mechanism of resistance drift is caused by the structural relaxation of the amorphous chalcogenide material. This drift follows a power law:

$$G(t) \= G\_0 \\left(\\frac{t}{t\_0}\\right)^{-\\nu}$$

where $G(t)$ is the conductance at time $t$, $G\_0$ is the initial conductance at time $t\_0$, and $\\nu$ is the drift coefficient.2  
Standardizing the Drift Coefficient ($\\nu$):  
Research indicates that the drift coefficient $\\nu$ is not a constant; it varies from state to state and from device to device. It also exhibits a dependency on the target conductance state $G\_T$. To enable a standardized "Analog Memory" market, metrology protocols must be established to characterize $\\nu$.

* **Protocol:** A standard drift measurement protocol might involve programming a statistical sample of devices to a target state, baking them at elevated temperature to accelerate aging (following Arrhenius principles), and measuring the conductance evolution over logarithmic time steps.  
* **Drift Compensation Metrics:** The effectiveness of drift compensation algorithms depends on the predictability of $\\nu$. Metrology must quantify not just the mean drift, but the variance ($\\sigma\_{\\nu}$). If the variance is too high, global compensation techniques (which assume a uniform drift across the array) will fail, leading to significant accuracy degradation.11

Random Telegraph Noise (RTN) and 1/f Noise:  
In addition to systematic drift, analog weights suffer from random noise. RTN, caused by the trapping and detrapping of charge carriers in oxide defects, causes discrete jumps in conductance. This creates a "read noise" floor that limits the effective bit-precision of the device. Metrology standards must define the integration time and bandwidth for measuring this noise to provide a comparable "Signal-to-Noise Ratio" (SNR) for analog memories.1

## ---

**3\. The Standardization Ecosystem: Filling the Analog Gap**

While the physics of Analog AI is being characterized by NIST and academic researchers, the industrial standardization of these technologies is currently fragmented. The existing infrastructure of semiconductor standards, dominated by JEDEC and IEEE, was built for the digital era. A new ecosystem is emerging to address the unique requirements of analog compute.

### **3.1. JEDEC: The Shift from Data Storage to Weight Storage**

JEDEC is the global leader in developing open standards for the microelectronics industry. Its standards for commodity memory (DDR, LPDDR, HBM) are ubiquitous. However, JEDEC's current standards for emerging non-volatile memories (NVM) like ReRAM and MRAM are focused on their use as *storage*, not as *computational weights*.12  
The JC-42.4 Subcommittee:  
This subcommittee handles standards for non-volatile memory. Currently, standards like JESD21-C treat ReRAM as a digital block device. The reliability metrics—Endurance and Retention—are defined in binary terms (Pass/Fail). A "failure" is defined as a single bit flip that cannot be corrected by on-die ECC.12  
The "Analog Weight" Gap:  
For Analog AI, a bit flip (or rather, a conductance shift) is not necessarily a failure. It contributes to error, but the neural network may be robust to it. The industry lacks a JEDEC standard that defines:

1. **Analog Endurance:** For training accelerators, devices must endure $10^9$ to $10^{12}$ updates. Current JEDEC standards for storage typically target $10^4$ to $10^5$ cycles. A new "Training Grade" endurance standard is needed.14  
2. **Analog Retention:** Instead of "Zero Bit Errors for 10 Years," an analog standard might define retention as "Maximum Conductance Drift $\< 10\\%$ over 10 Years at $85^\\circ C$."  
3. **Program/Verify Interfaces:** Standardizing the interface for iterative programming. Currently, every analog chip uses a proprietary algorithm to converge the weight to the target value. A standardized command set (e.g., SET\_TARGET\_G, VERIFY\_LOOP\_MAX) would allow for interoperable controller IP.

### **3.2. ISO/IEC JTC 1/SC 42: The Architect of AI Standards**

At the international level, the ISO/IEC Joint Technical Committee 1, Subcommittee 42 (JTC 1/SC 42\) is the system integration entity for AI standardization. This committee is developing the foundational standards that will likely govern the certification of Analog AI hardware in global markets.15  
**Key Standards for Hardware Developers:**

* **ISO/IEC 22989 (Artificial Intelligence Concepts and Terminology):** This standard establishes the vocabulary for AI. It is crucial that the definitions of "AI System" and "Model" are inclusive of analog hardware implementations, ensuring that regulations applying to "AI Models" also apply to the physical weights stored in ReRAM crossbars.15  
* **ISO/IEC 42001 (AI Management System):** This is a management system standard (similar to ISO 9001\) for organizations developing AI. For a semiconductor company, compliance with 42001 would involve establishing processes to manage the specific risks of analog hardware, such as drift-induced accuracy loss or batch-to-batch variability in manufacturing.17  
* **ISO/IEC 23894 (Guidance on Risk Management):** This standard provides a framework for managing AI-related risks. For Analog AI, relevant risks include "Hardware Faults," "Environmental Sensitivity," and "Model Theft." The standard provides a methodology for assessing these risks and implementing controls, which for analog hardware might include on-chip temperature compensation or differential weight encoding.19

### **3.3. Benchmarking as De Facto Standardization**

In the absence of rigid hardware specifications, benchmarks serve as the functional standard for performance and accuracy.  
The Limitations of MLPerf:  
MLCommons' MLPerf is the dominant benchmark for digital AI. However, its metrics—primarily time-to-solution for ISO-accuracy on standard datasets like ImageNet—disadvantage Analog AI. Analog chips often trade a small percentage of accuracy for orders-of-magnitude improvements in energy efficiency. MLPerf's rigid accuracy requirements effectively disqualify many analog prototypes that could otherwise be highly valuable for edge applications.20  
NeuroBench: A Benchmark for the Analog Era:  
To address this gap, a consortium of academic and industrial leaders has introduced NeuroBench, a benchmark framework specifically designed for neuromorphic and analog systems.23

* **Complexity Metrics:** NeuroBench introduces metrics such as "Footprint" (memory usage) and "Connection Sparsity" (fraction of zero weights). These metrics highlight the architectural advantages of analog systems that can implement sparse connectivity efficiently.26  
* **Correctness per Watt:** Rather than optimizing solely for top-1 accuracy, NeuroBench evaluates the trade-off between correctness and energy consumption. This allows an analog chip to demonstrate its superiority in "good enough" computing contexts.  
* **Event-Based Metrics:** For spiking neural networks (SNNs), NeuroBench measures performance in terms of synaptic operations (SynOps) and events per second, rather than FLOPS. This aligns with the physics of event-driven hardware like the processors from Innatera or SynSense.28

### **3.4. Interconnect Standards: UCIe and the Chiplet Revolution**

As Analog AI accelerators are increasingly integrated as chiplets alongside digital processors, the **Universal Chiplet Interconnect Express (UCIe)** standard has become a critical enabler. UCIe defines the physical and protocol layers for die-to-die communication.29  
**UCIe for Analog AI:**

* **Heterogeneous Integration:** UCIe allows an Analog AI chiplet (fabricated on a specialized legacy node like 40nm RRAM) to be packaged with a high-performance digital host (fabricated on 3nm FinFET). This decoupling of manufacturing processes is vital, as analog memory devices often cannot scale to the smallest process nodes due to voltage scaling limits.  
* **Standardized Interfaces:** UCIe supports protocols like CXL (Compute Express Link) and streaming raw modes. This allows the digital host to treat the analog accelerator as a standard memory-mapped device or a streaming processor, abstracting away the complex analog physics occurring within the chiplet.31 The UCIe 2.0 specification adds support for system management and debug, which is crucial for monitoring the health and calibration status of analog chiplets in the field.29

## ---

**4\. The Regulatory Landscape: The EU AI Act and Beyond**

The regulatory environment for AI is shifting from voluntary guidelines to mandatory law. The European Union's AI Act is the most significant piece of legislation in this domain, and it contains specific requirements that have profound implications for Analog AI hardware.

### **4.1. "High-Risk" AI and the Mandate for Robustness**

The EU AI Act classifies AI systems based on risk. "High-Risk" systems include medical devices, safety components in critical infrastructure (transport, water, energy), and biometric identification systems.3  
Article 15: Accuracy, Robustness, and Cybersecurity:  
Article 15 of the AI Act explicitly requires high-risk AI systems to "achieve an appropriate level of accuracy, robustness, and cybersecurity" and to perform consistently throughout their lifecycle.3

* **The Analog Challenge:** The inherent stochasticity of Analog AI poses a compliance challenge. If a medical diagnostic chip uses analog weights that drift over time, can it guarantee "consistent performance throughout its lifecycle"?  
* **The Hardware-Aware Training (HAT) Solution:** To comply with Article 15, manufacturers of Analog AI chips will likely need to employ rigorous Hardware-Aware Training (HAT). By simulating the device's noise and drift during the training phase, developers can produce models that are mathematically robust to the hardware's physical imperfections. This transforms HAT from an optimization technique into a *compliance tool*.2  
* **Documentation Requirements:** The Act also mandates detailed technical documentation (Article 11). For Analog AI, this implies that manufacturers must characterize and document the statistical properties of their hardware—drift rates, temperature dependence, and noise floors. "Black box" analog physics will not be acceptable in high-risk deployments.

### **4.2. Prohibited Practices and Hardware Limitations**

The AI Act prohibits certain AI practices deemed to carry unacceptable risk, such as real-time remote biometric identification in public spaces (with law enforcement exceptions) and emotion recognition in workplaces.35

* **Implications for "Always-On" Sensors:** Many Analog AI chips target the "always-on" market, performing tasks like face detection or voice activity detection at microwatt power levels.28 If a chip is hard-coded (via fixed analog weights) to perform a prohibited task (e.g., emotion detection), the hardware itself could be restricted.  
* **Programmability as Compliance:** To mitigate regulatory risk, Analog AI hardware should be programmable. The ability to update weights allows a device to be repurposed or patched to comply with evolving regulations. Fixed-function analog ASICs face a higher risk of obsolescence due to regulatory changes.

### **4.3. Conformity Assessments and Dual Certification**

Before a High-Risk AI system can be placed on the EU market, it must undergo a conformity assessment. For Analog AI hardware used in safety-critical systems (e.g., automotive ADAS), this creates a dual-certification requirement:

1. **Functional Safety Certification:** Compliance with ISO 26262 (Automotive) or IEC 61508 (Industrial) to ensure the hardware fails safely.  
2. **AI Act Conformity:** Compliance with the AI Act's requirements for accuracy, robustness, and fairness.

These two regimes are distinct but overlapping. A "safe" chip (ISO 26262\) might still be "biased" or "inaccurate" (AI Act). Conversely, a robust AI model might run on hardware that is not functionally safe. Manufacturers must navigate both pathways simultaneously.

## ---

**5\. Functional Safety in a Stochastic World: ISO 26262**

ISO 26262 is the functional safety standard for electrical and electronic systems in road vehicles. It is a rigorous, prescriptive standard designed to prevent unreasonable risk due to hazards caused by malfunctioning behavior.

### **5.1. The Conflict: Determinism vs. Probability**

ISO 26262 is rooted in the concept of determinism. It classifies faults into two categories:

1. **Systematic Faults:** Errors in the design or manufacturing process (bugs). These are deterministic and can be eliminated through rigorous process (the V-model).  
2. **Random Hardware Faults:** Failures that occur unpredictably during operation (e.g., a cosmic ray flipping a bit, electromigration causing an open circuit). These are managed probabilistically using safety mechanisms.36

The Analog Paradox:  
In Analog AI, a "fault" is not always a binary event. Thermal noise might cause a correctly functioning device to output a slightly different current value. Is this a random hardware fault? Or is it nominal behavior? ISO 26262 currently lacks a clear category for "intrinsic stochasticity."

* **Redefining Failure:** The industry is moving towards defining failure in terms of *functional degradation* rather than component deviation. A memristor drifting by 5% is only a "failure" if it causes the neural network's classification accuracy to drop below a safety threshold defined in the Safety Goal.

### **5.2. ASIL Decomposition and Safety Architectures**

Automotive Safety Integrity Levels (ASIL) range from A (lowest) to D (highest). Achieving ASIL D—which requires extremely low failure rates (10 FIT)—with a purely analog processor is currently infeasible due to the lack of robust ECC for analog values.  
ASIL Decomposition:  
To deploy Analog AI in safety-critical systems, architects employ ASIL decomposition. A high-performance Analog AI chip (rated ASIL B or QM) might be paired with a traditional digital safety checker (rated ASIL D).38

* **The Monitor Concept:** The analog chip performs the heavy lifting (e.g., pedestrian detection). The digital monitor performs a sanity check (e.g., checking if the detected object trajectory is physically possible) or runs a simplified, redundant model. If the analog chip's output deviates or becomes erratic due to drift, the safety monitor takes control, bringing the system to a Safe State.

Safety Element out of Context (SEooC):  
Since Analog AI chips are often developed as general-purpose components (IP) without knowing the final vehicle system, they are certified as Safety Elements out of Context (SEooC). The IP provider (e.g., Synopsys, Innatera) produces a "Safety Manual" that defines the assumptions of use.36

* **The Safety Manual:** For an analog chip, the Safety Manual is critical. It must specify the *Fault Tolerant Time Interval (FTTI)* and the required *Recalibration Frequency*. It puts the burden on the system integrator to ensure that the chip is reset or recalibrated often enough to prevent dangerous drift.40

### **5.3. Random Hardware Failure Metrics (PMHF)**

ISO 26262 requires the calculation of the **Probabilistic Metric for Random Hardware Failures (PMHF)**. This involves summing the failure rates ($\\lambda$) of all components in the safety path.

* **FMEDA for Analog:** Creating a Failure Modes, Effects, and Diagnostic Analysis (FMEDA) for an analog chip requires new data. What is the FIT rate of a ReRAM cell? While digital transistor failure rates are well-characterized (IEC 62380), analog memory reliability data is often proprietary to foundries.  
* **Diagnostic Coverage:** The PMHF calculation depends heavily on Diagnostic Coverage ($DC$). Safety mechanisms (like periodic self-tests) increase $DC$. For Analog AI, a key safety mechanism is the "Reference Column." By interleaving columns of fixed, known resistors within the memory array, the system can continuously monitor the read-out circuitry for drift and gain errors, achieving high diagnostic coverage.41

### **5.4. Case Studies in Automotive Analog AI**

Innatera:  
Innatera, a developer of spiking neural processors, positions its T1 processor for "always-on" sensing in automotive applications (e.g., in-cabin monitoring). To meet safety requirements, the chip combines a stochastic SNN engine with a deterministic RISC-V core. This heterogeneous architecture allows the RISC-V core to act as a safety manager, supervising the SNN and handling communication with the central ECU. While the SNN provides efficiency, the RISC-V provides the deterministic anchor required for ISO 26262 compliance.28  
NVIDIA Halos:  
NVIDIA's "Halos" safety system represents a holistic approach. It integrates hardware safety mechanisms with "algorithmic safety"—using formal methods and diverse datasets to ensure the AI model itself is robust. For Analog AI to compete, it must offer a similar "full-stack" safety case, proving that the hardware (ReRAM), the software (HAT), and the algorithm (SNN) work together to minimize risk.44

## ---

**6\. Hardware Security: Protecting the Weights**

In the world of AI, the "weights" of a trained model represent millions of dollars of R\&D investment and computational cost. Storing these weights on non-volatile analog memory at the edge exposes them to physical theft. Conversely, the variability of analog hardware offers unique opportunities for security.

### **6.1. The Threat of Model Theft**

A recent report by the RAND Corporation identifies "stealing a model's weights" as a primary vector for AI proliferation and misuse.45

* **Attack Vectors:** Attackers can extract weights via **Side-Channel Attacks** (measuring power consumption or electromagnetic emissions during inference) or **Physical Probing** (delayering the chip to measure the resistance of individual cells).47  
* **Analog Vulnerability:** Analog compute, which relies on current summation, is potentially more vulnerable to simple and differential power analysis (SPA/DPA) than digital logic. The total current drawn by a crossbar is directly proportional to the vector-matrix product, leaking information about the weights.

ISO/IEC 27090:  
The upcoming standard ISO/IEC 27090 (Guidance for addressing security threats to AI systems) addresses these risks. It will likely recommend specific controls for protecting model confidentiality. For Analog AI, this implies the need for architectural defenses, such as:

* **Address Scrambling:** Randomizing the physical location of weights so that a physical readout yields unintelligible data.  
* **Obfuscation:** Using "locking" weights or inputs that require a specific digital key to produce the correct analog output.48

### **6.2. Physical Unclonable Functions (PUFs): Turning Bugs into Features**

The same stochastic variability that plagues analog metrology—the random variation in filament formation or threshold voltage—can be harnessed to create **Physical Unclonable Functions (PUFs)**. A PUF uses the device's unique physical "fingerprint" to generate a cryptographic key that exists only when the device is powered on and cannot be cloned.49  
ISO/IEC 20897 Compliance:  
ISO/IEC 20897 (Physically unclonable functions) is the international standard governing PUF security. It specifies requirements for Randomness, Uniqueness, Steadiness, and Tamper-resistance.51

* **ReRAM PUFs:** ReRAM arrays are excellent candidates for PUFs. The cycle-to-cycle variation and device-to-device mismatch can be used to generate high-entropy keys. Companies like CrossBar are commercializing ReRAM PUF technology for secure root-of-trust applications.50  
* **Enrollment and Reconstruction:** The standard defines the "Enrollment" phase (where the device's fingerprint is characterized and "helper data" is generated) and the "Reconstruction" phase (where the key is regenerated). For analog PUFs, the reconstruction phase often requires Error Correction Codes (ECC) to stabilize the noisy analog reading into a deterministic digital key. The implementation of these "Fuzzy Extractors" is a critical part of the security architecture.53

NIST SP 800-90B (Entropy Sources):  
If the analog noise is used to generate random numbers (TRNG), the entropy source must be validated according to NIST SP 800-90B. This involves rigorous statistical testing (e.g., Repetition Count Test, Adaptive Proportion Test) to ensure the noise source is truly random and not subject to predictable patterns or manipulation.54

## ---

**7\. The Crucial Role of Hardware-Aware Training (HAT)**

As discussed, the physical realities of analog hardware (noise, drift, variability) conflict with the regulatory demands for accuracy and robustness. The bridge between these two worlds is **Hardware-Aware Training (HAT)**.

### **7.1. Software-Defined Metrology and Calibration**

HAT involves incorporating a simulation of the hardware's non-idealities into the forward pass of the neural network training loop. This allows the network to "learn" weights that are resilient to the specific noise profile of the chip.  
The IBM AIHWKit:  
IBM's open-source Analog Hardware Acceleration Kit (AIHWKit) is a pioneering tool in this domain. It provides a standardized framework for simulating various memory technologies (PCM, ReRAM) and their associated noise models.1

* **Drift Compensation:** The toolkit simulates conductance drift over time. By training with this simulation, developers can verify if their model will meet accuracy requirements after 1 day, 1 month, or 1 year of operation. This simulation data can be used to generate the "Technical Documentation" required by the EU AI Act, providing evidence of robustness.  
* **Noise Injection:** HAT injects Gaussian noise into the weights and activations during training. This acts as a regularizer, forcing the network to find wide, flat minima in the loss landscape, which are intrinsically more robust to analog perturbations.57

### **7.2. Standardizing the Error Model**

For the ecosystem to scale, the industry needs standardized "Noise Models" for memory technologies. Just as BSIM models standardize transistor behavior for circuit simulation, a **"JEDEC ReRAM Noise Model"** would allow AI developers to train models that are guaranteed to work on any compliant analog memory chip.

* **Current State:** Currently, error models are proprietary to each foundry (TSMC, GlobalFoundries) or integrated into specific tools like AIHWKit.  
* **Future Need:** A unified standard would define parameters for drift ($\\nu$), read noise (PSD), and write non-linearity, allowing for "write once, run anywhere" (within noise limits) portability for Analog AI models.

## ---

**8\. Summary of Recommendations for Industry Players**

Based on this deep research into the intersection of physics, standards, and law, the following strategic actions are recommended for stakeholders in the Analog AI ecosystem:

| Stakeholder | Strategic Recommendations |
| :---- | :---- |
| **Chip Designers** | **1\. Implement HAT as Standard Practice:** Treat Hardware-Aware Training not as an optimization, but as a compliance requirement for the EU AI Act. **2\. Adopt SEooC Safety Cases:** Develop detailed Safety Manuals specifying recalibration intervals and FTTI to support ISO 26262 integration. **3\. Design for Test:** Integrate "Reference Columns" and on-chip metrology to enable continuous calibration. |
| **Standards Bodies (JEDEC/ISO)** | **1\. Create Analog Compute Standards:** JEDEC must move beyond "storage" standards to define "Analog Weight Interfaces" (AWI) with standardized conductance levels and training-grade endurance. **2\. Harmonize AI & Safety:** ISO/IEC SC 42 and TC 22 (Automotive) must collaborate to define how "probabilistic robustness" maps to "functional safety." |
| **Regulators** | **1\. Accept Statistical Verification:** Conformity assessment bodies must adapt to accept statistical confidence intervals (e.g., "99.9% robust") rather than deterministic bit-exactness. **2\. Clarify "Robustness":** Provide clear guidance on acceptable drift limits for High-Risk AI systems under the EU AI Act. |
| **Metrology Labs (NIST/IRDS)** | **1\. Standardize Stochastic Metrology:** Develop protocols for characterizing the *distribution* of analog states, not just their mean values. **2\. Reference Artifacts:** Create standard "Analog Weight" artifacts to calibrate foundry tools. |

Conclusion:  
The commercialization of Analog AI is no longer just a physics problem; it is a metrology and regulatory challenge. The technology promises a revolution in edge AI efficiency, but it will only succeed if the industry can build a robust framework of standards that translates its probabilistic nature into the reliable, safe, and legal language of the modern world. The convergence of NIST's advanced metrology, JEDEC's emerging standards, and ISO's safety frameworks provides the necessary path forward. The winners in this space will be those who can not only build the hardware but also mathematically prove its reliability to the satisfaction of both the engineer and the regulator.

#### **Works cited**

1. Optimised weight programming for analogue memory-based deep ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC9247051/](https://pmc.ncbi.nlm.nih.gov/articles/PMC9247051/)  
2. Optimized Weight Programming for Analogue Memory-based Deep Neural Networks \- Semantic Scholar, accessed December 24, 2025, [https://pdfs.semanticscholar.org/2c1c/18961f934a07e7b5c9778199d28964089658.pdf](https://pdfs.semanticscholar.org/2c1c/18961f934a07e7b5c9778199d28964089658.pdf)  
3. EU AI Act: first regulation on artificial intelligence | Topics \- European Parliament, accessed December 24, 2025, [https://www.europarl.europa.eu/topics/en/article/20230601STO93804/eu-ai-act-first-regulation-on-artificial-intelligence](https://www.europarl.europa.eu/topics/en/article/20230601STO93804/eu-ai-act-first-regulation-on-artificial-intelligence)  
4. (PDF) Metrology for the next generation of semiconductor devices \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/328251292\_Metrology\_for\_the\_next\_generation\_of\_semiconductor\_devices](https://www.researchgate.net/publication/328251292_Metrology_for_the_next_generation_of_semiconductor_devices)  
5. accessed December 24, 2025, [https://tsapps.nist.gov/publication/get\_pdf.cfm?pub\_id=956664](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=956664)  
6. INTERNATIONAL ROADMAP FOR DEVICES AND SYSTEMS TM 2024 METROLOGY | NIST, accessed December 24, 2025, [https://www.nist.gov/publications/international-roadmap-devices-and-systems-tm-2024-metrology](https://www.nist.gov/publications/international-roadmap-devices-and-systems-tm-2024-metrology)  
7. Metrology for 2D materials: a perspective review from the ... \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11059534/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11059534/)  
8. Metrology for 2D materials: a perspective review from the international roadmap for devices and systems \- National Institute of Standards and Technology, accessed December 24, 2025, [https://www.nist.gov/publications/metrology-2d-materials-perspective-review-international-roadmap-devices-and-systems](https://www.nist.gov/publications/metrology-2d-materials-perspective-review-international-roadmap-devices-and-systems)  
9. Neuromorphic systems | NIST, accessed December 24, 2025, [https://www.nist.gov/programs-projects/neuromorphic-systems](https://www.nist.gov/programs-projects/neuromorphic-systems)  
10. Neuromorphic Computing | NIST \- National Institute of Standards and Technology, accessed December 24, 2025, [https://www.nist.gov/programs-projects/neuromorphic-computing](https://www.nist.gov/programs-projects/neuromorphic-computing)  
11. Inference and PCM statistical model \- IBM Analog Hardware Acceleration Kit's documentation\!, accessed December 24, 2025, [https://aihwkit.readthedocs.io/en/v0.3.0/pcm\_inference.html](https://aihwkit.readthedocs.io/en/v0.3.0/pcm_inference.html)  
12. JEDEC memory standards \- Wikipedia, accessed December 24, 2025, [https://en.wikipedia.org/wiki/JEDEC\_memory\_standards](https://en.wikipedia.org/wiki/JEDEC_memory_standards)  
13. Examining Resistive RAM Standards and Compliance, accessed December 24, 2025, [https://eureka.patsnap.com/report-examining-resistive-ram-standards-and-compliance](https://eureka.patsnap.com/report-examining-resistive-ram-standards-and-compliance)  
14. ReRAM: Emerging as the New Embedded NVM Standard \- Flash Memory Summit, accessed December 24, 2025, [https://files.futurememorystorage.com/proceedings/2025/20250806\_OMEM-202-1\_Regev.pdf](https://files.futurememorystorage.com/proceedings/2025/20250806_OMEM-202-1_Regev.pdf)  
15. SC 42 \- JTC 1, accessed December 24, 2025, [https://jtc1info.org/sd-2-history/jtc1-subcommittees/sc-42/](https://jtc1info.org/sd-2-history/jtc1-subcommittees/sc-42/)  
16. SC 42 – Artificial Intelligence \- ITU, accessed December 24, 2025, [https://www.itu.int/en/ITU-T/extcoop/ai-data-commons/Documents/ISO\_IEC%20JTC1%20SC%2042%20Keynote\_Wael%20Diab.pdf](https://www.itu.int/en/ITU-T/extcoop/ai-data-commons/Documents/ISO_IEC%20JTC1%20SC%2042%20Keynote_Wael%20Diab.pdf)  
17. How the ISO and IEC are developing international standards for the responsible adoption of AI | Global AI Ethics and Governance Observatory \- UNESCO, accessed December 24, 2025, [https://www.unesco.org/ethics-ai/en/articles/how-iso-and-iec-are-developing-international-standards-responsible-adoption-ai](https://www.unesco.org/ethics-ai/en/articles/how-iso-and-iec-are-developing-international-standards-responsible-adoption-ai)  
18. AI and Compliance for the Mid-Market | CSA \- Cloud Security Alliance, accessed December 24, 2025, [https://cloudsecurityalliance.org/articles/ai-and-compliance-for-the-mid-market](https://cloudsecurityalliance.org/articles/ai-and-compliance-for-the-mid-market)  
19. A Guide to the Responsible Adoption of AI: Building a Secure Foundation with ISO \- CWSI, accessed December 24, 2025, [https://cwsisecurity.com/a-guide-to-the-responsible-adoption-of-ai/](https://cwsisecurity.com/a-guide-to-the-responsible-adoption-of-ai/)  
20. MLPerf Client Benchmark \- MLCommons, accessed December 24, 2025, [https://mlcommons.org/benchmarks/client/](https://mlcommons.org/benchmarks/client/)  
21. MLCommons Releases MLPerf Training v5.1 Results, accessed December 24, 2025, [https://mlcommons.org/2025/11/training-v5-1-results/](https://mlcommons.org/2025/11/training-v5-1-results/)  
22. MLPerf Client v1.5 Advances AI PC Benchmarking with Windows ML Integration, accessed December 24, 2025, [https://mlcommons.org/2025/11/mlperf-client-1-5-release/](https://mlcommons.org/2025/11/mlperf-client-1-5-release/)  
23. (PDF) The neurobench framework for benchmarking neuromorphic computing algorithms and systems \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/388915673\_The\_neurobench\_framework\_for\_benchmarking\_neuromorphic\_computing\_algorithms\_and\_systems](https://www.researchgate.net/publication/388915673_The_neurobench_framework_for_benchmarking_neuromorphic_computing_algorithms_and_systems)  
24. The neurobench framework for benchmarking neuromorphic computing algorithms and systems \- Simple search \- DiVA portal, accessed December 24, 2025, [http://ri.diva-portal.org/smash/record.jsf?pid=diva2:1999847](http://ri.diva-portal.org/smash/record.jsf?pid=diva2:1999847)  
25. The neurobench framework for benchmarking neuromorphic computing algorithms and systems \- PubMed, accessed December 24, 2025, [https://pubmed.ncbi.nlm.nih.gov/39934126/](https://pubmed.ncbi.nlm.nih.gov/39934126/)  
26. neurobench/leaderboard.rst at main \- GitHub, accessed December 24, 2025, [https://github.com/NeuroBench/neurobench/blob/main/leaderboard.rst](https://github.com/NeuroBench/neurobench/blob/main/leaderboard.rst)  
27. Energy Aware Development of Neuromorphic Implantables: From Metrics to Action \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2506.09599v1](https://arxiv.org/html/2506.09599v1)  
28. Products | Innatera, accessed December 24, 2025, [https://innatera.com/products](https://innatera.com/products)  
29. Specifications \- UCIe Consortium, accessed December 24, 2025, [https://www.uciexpress.org/specifications](https://www.uciexpress.org/specifications)  
30. The UCIe Chiplet Interconnect Standard \- Alphawave Semi, accessed December 24, 2025, [https://awavesemi.com/the-ucie-chiplet-interconnect-standard/](https://awavesemi.com/the-ucie-chiplet-interconnect-standard/)  
31. The Future of Chip Connectivity: UCIe and Optical I/O FAQs Explained \- Ayar Labs, accessed December 24, 2025, [https://ayarlabs.com/blog/the-future-of-chip-connectivity-ucie-and-optical-i-o-faqs-explained/](https://ayarlabs.com/blog/the-future-of-chip-connectivity-ucie-and-optical-i-o-faqs-explained/)  
32. Enabling Industry NoC Interconnects with UCIe IP \- Synopsys, accessed December 24, 2025, [https://www.synopsys.com/articles/noc-interconnects-ucie-ip.html](https://www.synopsys.com/articles/noc-interconnects-ucie-ip.html)  
33. High-level summary of the AI Act | EU Artificial Intelligence Act, accessed December 24, 2025, [https://artificialintelligenceact.eu/high-level-summary/](https://artificialintelligenceact.eu/high-level-summary/)  
34. AI Act: EU's Bid to Set the Global Standard for AI Regulation \- McDermott Will & Schulte, accessed December 24, 2025, [https://www.mwe.com/insights/the-ai-act-the-eus-bid-to-set-the-global-standard-for-ai-regulation/](https://www.mwe.com/insights/the-ai-act-the-eus-bid-to-set-the-global-standard-for-ai-regulation/)  
35. Article 5: Prohibited AI Practices | EU Artificial Intelligence Act, accessed December 24, 2025, [https://artificialintelligenceact.eu/article/5/](https://artificialintelligenceact.eu/article/5/)  
36. ISO 26262 Functional Safety for Automotive \- Renesas, accessed December 24, 2025, [https://www.renesas.com/en/applications/automotive/automotive-functional-safety](https://www.renesas.com/en/applications/automotive/automotive-functional-safety)  
37. What Is ISO 26262? ISO 26262 Functional Safety Overview \+ ASIL | Perforce Software, accessed December 24, 2025, [https://www.perforce.com/blog/qac/what-is-iso-26262](https://www.perforce.com/blog/qac/what-is-iso-26262)  
38. Safety-Oriented System Hardware Architecture Exploration in Compliance with ISO 26262, accessed December 24, 2025, [https://www.mdpi.com/2076-3417/12/11/5456](https://www.mdpi.com/2076-3417/12/11/5456)  
39. Stochastic Constituents for the Probabilistic Metric of Random Hardware Failures in ISO 26262 \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/369847442\_Stochastic\_Constituents\_for\_the\_Probabilistic\_Metric\_of\_Random\_Hardware\_Failures\_in\_ISO\_26262](https://www.researchgate.net/publication/369847442_Stochastic_Constituents_for_the_Probabilistic_Metric_of_Random_Hardware_Failures_in_ISO_26262)  
40. Safety manual \- NXP Semiconductors, accessed December 24, 2025, [https://www.nxp.com/docs/en/user-guide/UM11163.pdf](https://www.nxp.com/docs/en/user-guide/UM11163.pdf)  
41. Generalized Formula for the Calculation of a Probabilistic Metric for Random Hardware Failures in Redundant Systems \- In Compliance Magazine, accessed December 24, 2025, [https://incompliancemag.com/random-hardware-failures-in-redundant-systems/](https://incompliancemag.com/random-hardware-failures-in-redundant-systems/)  
42. ISO26262 ams deploys unique technology to meet every new safety requirement, accessed December 24, 2025, [https://ams-osram.com/documents/20143/80162/TA\_ams\_ISO2626\_new+safety+requirements.pdf/0bce4ef6-23c9-bc85-bf98-b3edf7b075f5](https://ams-osram.com/documents/20143/80162/TA_ams_ISO2626_new+safety+requirements.pdf/0bce4ef6-23c9-bc85-bf98-b3edf7b075f5)  
43. Spiking Neural Processor T1 | Innatera, accessed December 24, 2025, [https://innatera.com/products/spiking-neural-processor-t1](https://innatera.com/products/spiking-neural-processor-t1)  
44. NVIDIA Launches NVIDIA Halos, a Full-Stack, Comprehensive Safety System for Autonomous Vehicles, accessed December 24, 2025, [https://blogs.nvidia.com/blog/halos-safety-system-autonomous-vehicles/](https://blogs.nvidia.com/blog/halos-safety-system-autonomous-vehicles/)  
45. A Playbook for Securing AI Model Weights \- RAND, accessed December 24, 2025, [https://www.rand.org/pubs/research\_briefs/RBA2849-1.html](https://www.rand.org/pubs/research_briefs/RBA2849-1.html)  
46. Securing AI Model Weights: Preventing Theft and Misuse of Frontier Models | RAND, accessed December 24, 2025, [https://www.rand.org/pubs/research\_reports/RRA2849-1.html](https://www.rand.org/pubs/research_reports/RRA2849-1.html)  
47. Neuromorphic Device Measurements | NIST, accessed December 24, 2025, [https://www.nist.gov/programs-projects/neuromorphic-device-measurements](https://www.nist.gov/programs-projects/neuromorphic-device-measurements)  
48. A Lightweight PUF-Based Weights Obfuscation Technique for Secure In-Memory AI Inference | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/390401229\_A\_Lightweight\_PUF-Based\_Weights\_Obfuscation\_Technique\_for\_Secure\_In-Memory\_AI\_Inference](https://www.researchgate.net/publication/390401229_A_Lightweight_PUF-Based_Weights_Obfuscation_Technique_for_Secure_In-Memory_AI_Inference)  
49. Hardware Security Primitives \- Dr. Domenic Forte, accessed December 24, 2025, [https://faculty.eng.ufl.edu/dforte/research/hardware-security-primitives/](https://faculty.eng.ufl.edu/dforte/research/hardware-security-primitives/)  
50. ReThink Secure Computing With ReRAM PUF Keys \- CrossBar Inc., accessed December 24, 2025, [https://www.crossbar-inc.com/technology/applications/secure-computing/](https://www.crossbar-inc.com/technology/applications/secure-computing/)  
51. Synopsys PUF IP: ISO/IEC 20897 Compliant, accessed December 24, 2025, [https://www.synopsys.com/designware-ip/security-ip/cryptography-ip/puf/iso-iec-20897.html](https://www.synopsys.com/designware-ip/security-ip/cryptography-ip/puf/iso-iec-20897.html)  
52. BS ISO/IEC 20897-1:2020 | 31 Dec 2020 \- BSI Knowledge, accessed December 24, 2025, [https://knowledge.bsigroup.com/products/information-security-cybersecurity-and-privacy-protection-physically-unclonable-functions-security-requirements](https://knowledge.bsigroup.com/products/information-security-cybersecurity-and-privacy-protection-physically-unclonable-functions-security-requirements)  
53. Hardware-Backed Identity: Implementing Physically Unclonable Functions (PUFs) in Software | by Lance Harvie | Oct, 2025 | Medium, accessed December 24, 2025, [https://medium.com/@lanceharvieruntime/hardware-backed-identity-implementing-physically-unclonable-functions-pufs-in-software-2216eb97574a](https://medium.com/@lanceharvieruntime/hardware-backed-identity-implementing-physically-unclonable-functions-pufs-in-software-2216eb97574a)  
54. Entropy Source Assessment \- atsec – INFORMATION SECURITY, accessed December 24, 2025, [https://www.atsec.com/services/cryptographic-testing/entropy-source-assessment/](https://www.atsec.com/services/cryptographic-testing/entropy-source-assessment/)  
55. SP 800-90B, Recommendation for the Entropy Sources Used for Random Bit Generation, accessed December 24, 2025, [https://csrc.nist.gov/pubs/sp/800/90/b/final](https://csrc.nist.gov/pubs/sp/800/90/b/final)  
56. Analog Hardware-aware Training, accessed December 24, 2025, [https://aihwkit.readthedocs.io/en/v0.9.0/hwa\_training.html](https://aihwkit.readthedocs.io/en/v0.9.0/hwa_training.html)  
57. Hardware-aware training for large-scale and diverse deep learning inference workloads using in-memory computing-based accelerators \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10469175/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10469175/)