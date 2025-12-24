### Proposal for Unified Signal and Metrology Standards for Analog AI Hardware

#### 1.0 Introduction: The Analog Imperative and the Standardization Bottleneck

As artificial intelligence models scale exponentially, the semiconductor industry is confronting a fundamental barrier to progress: the von Neumann "memory wall." In conventional digital architectures, the energy and latency costs of moving synaptic weights and activation data between memory and processing units now dominate the system's power budget, consuming orders of magnitude more energy than the computation itself. This data movement bottleneck is becoming the primary inhibitor of deploying large-scale AI, especially in energy-constrained environments.Analog In-Memory Computing (AIMC) has emerged as the definitive architectural solution to this crisis. By embedding computation directly within memory arrays, AIMC circumvents the data movement bottleneck. 

This paradigm leverages the physical laws of Ohm and Kirchhoff to perform massively parallel matrix-vector multiplication—the core of all neural network operations—in a single time step, delivering orders-of-magnitude gains in energy efficiency.While AIMC technology is maturing rapidly across a diverse range of physical substrates, its path to broad commercial adoption is obstructed. The lack of industry-wide signal, metrology, and interface standards is now the single greatest barrier to ecosystem development and interoperability. Without a common language to define, measure, and control analog hardware, the industry risks fragmentation into a collection of proprietary, incompatible solutions.

This proposal outlines a comprehensive, cross-technology standardization framework for bodies like JEDEC and IEEE, aimed at ensuring the reliable, scalable, and certifiable deployment of analog AI. To build this framework, we must first establish the common principles that make such an effort feasible.

#### 2.0 The Foundation for Standardization: Universal Principles Across Diverse Technologies

While analog memory technologies such as Phase-Change Memory (PCM), Resistive RAM (RRAM), Ferroelectric FETs (FeFET), and Flash have distinct underlying physics, they converge on a common set of functional challenges and architectural requirements. This "common ground" of shared principles and problems makes a unified standardization effort not only possible but essential for the health of the emerging ecosystem.

##### 2.1 The Shared Computational Paradigm

All analog AI hardware shares a universal computational model that is fundamentally different from digital logic. This model is built upon two physical laws:

* **Ohm's Law for Multiplication:**  I \= V × G. The synaptic weight, encoded as a physical conductance (G), is multiplied by the input activation, encoded as a voltage (V), to produce an output current (I).  
* **Kirchhoff's Current Law for Accumulation:**  ΣI. Currents from multiple devices on a shared bitline sum naturally, performing the accumulation step of a matrix-vector multiplication in the analog domain.This shared physics-based computation is the source of AIMC's massive parallelism and efficiency.

##### 2.2 Universal State Variables

Despite their different materials and mechanisms, all leading analog memory technologies encode weights as continuous physical state variables. This convergence on a common functional abstraction provides a clear target for standardization.

| Technology | State Variable | Physical Mechanism |
| :---- | :---- | :---- |
| **ReRAM/RRAM** | Filament conductance | Oxygen vacancy migration |
| **PCM** | Crystalline-amorphous ratio | Chalcogenide phase transition |
| **FeFET** | Remnant polarization (P\_r) | Ferroelectric domain switching |
| **Flash/SONOS** | Trapped charge (Q) | Fowler-Nordheim tunneling |

##### 2.3 Universal Challenges

The transition from deterministic digital bits to probabilistic analog states introduces a common set of five fundamental challenges that every analog AI technology must overcome. These shared problems demand a common set of standards.

* **Stochasticity:**  All technologies exhibit inherent randomness in discrete physical events, such as individual ion migration in ReRAM or the nucleation of distinct crystalline domains in FeFETs.  
* **Temporal Drift:**  Programmed analog states are not perfectly static, with PCM exhibiting a characteristic power-law drift and FeFETs suffering from depolarization effects over time.  
* **Read/Write Asymmetry:**  Most technologies exhibit difficulty in making SET (conductance increase) and RESET (conductance decrease) operations symmetric in speed, energy, and linearity.  
* **Noise Floor:**  The effective precision of all analog devices is limited by noise sources, including Random Telegraph Noise (RTN) from single-electron effects and thermal noise.  
* **Digital Interface:**  All analog cores must ultimately interface with digital systems, requiring high-precision Digital-to-Analog (DAC) and Analog-to-Digital (ADC) converters.These shared challenges necessitate a common framework for specification, measurement, and operation, which is detailed in the following sections.

#### 3.0 Proposed Framework for Core Signal and State Standards

This section details the core technical standards required to define and manage the state of an analog AI device. These standards form the bedrock for ensuring device interoperability, predictable performance, and a reliable supply chain.

##### 3.1 Standard for Weight State Specification

To move beyond proprietary definitions, the industry needs a universal standard for representing and specifying an analog weight.

###### *3.1.1 Conductance Representation*

The standard representation for an analog weight shall be  **normalized conductance** : G\_norm \= (G − G\_min) / (G\_max − G\_min). This standard must also define minimum requirements for Dynamic Range (the on/off ratio between G\_max and G\_min) and establish Precision Tiers to match application needs (e.g., "Inference Grade" for 4-6 effective bits, "Edge Training Grade" for 6-8 bits).

###### *3.1.2 Signed Weight Encoding*

To resolve the mismatch between strictly positive physical conductance and the signed (positive/negative) weights of neural networks, this framework mandates the use of  **differential pair encoding**  (W ∝ G⁺ − G⁻) as the industry standard. This approach uses two physical devices per weight and is justified by its critical benefits:

* It provides a native representation for both positive and negative weights.  
* It delivers powerful common-mode rejection, canceling out global variations from temperature fluctuations and power supply noise, thereby significantly improving signal integrity.This technique transforms the core liabilities of analog hardware—sensitivity to noise and temperature—into manageable common-mode signals, effectively designing resilience directly into the physical layer.

###### *3.1.3 Statistical State Specification*

The probabilistic nature of analog hardware requires a shift from deterministic to statistical specifications. A standard report for a programmed weight must include:

* **Target Conductance (G\_T):**  The intended mean value of the conductance distribution.  
* **State Variance (σ\_G):**  The acceptable standard deviation around the target.  
* **Confidence Interval:**  The probability that the actual device conductance falls within the specified bounds (e.g., 99.7% for a 3σ interval).

##### 3.2 Standard for Temporal Stability

To address the universal challenge of  **Temporal Drift** , which manifests as power-law relaxation in PCM and depolarization in FeFETs, a standardized approach to characterization and compensation is non-negotiable for long-term product reliability.

###### *3.2.1 Drift Characterization Protocol*

A standard protocol shall be established for measuring and reporting temporal drift. This includes specifying the  **Drift Coefficient (ν)**  based on the widely observed power-law equation: G(t) \= G(t₀) \* (t/t₀)⁻ν. To serve different market requirements, the standard should define tiered  **Retention Classes**  (e.g., Class A for enterprise, Class B for consumer) with specified maximum conductance changes over time at temperature.

###### *3.2.2 Drift Compensation Interface*

To enable interoperable hardware solutions for drift management, the framework proposes standards for:

* **Reference Column Protocol:**  A standard for the placement and calibration of fixed-reference conductance columns used for global drift tracking.  
* **Compensation Signal Format:**  A standard digital format for communicating time-dependent correction factors from tracking circuits to ADCs or digital post-processing units.

##### 3.3 Standard for Input/Output Encoding

Standardizing how analog cores receive inputs and transmit outputs is essential for system-level integration with digital host processors.| Input Modulation Scheme | Key Advantages | Areas Requiring Standardization || \------ | \------ | \------ || **Pulse Amplitude (PAM)** | Maximum throughput (single-cycle operation) | DAC linearity, voltage headroom, IR-drop compensation schemes || **Pulse Width (PWM)** | Inherent linearity, high noise robustness | Timing precision protocols, integration window definition |  
With these core state and signal definitions established, we can now specify a commensurate framework for metrology, ensuring that what one company specifies, another can reliably and repeatably measure.

#### 4.0 Proposed Framework for Metrology and Benchmarking

Conventional semiconductor metrology, focused on atomic-scale geometry, is fundamentally blind to the probabilistic behaviors that define analog AI performance. This obsolescence creates a metrology bottleneck that is as severe as the von Neumann memory wall, rendering our existing quality and reliability frameworks ineffective. As the International Roadmap for Devices and Systems (IRDS) Metrology Chapter notes, the industry's focus must expand from measuring physical  *structure*  to measuring statistical  *function* .

##### 4.1 Functional and Statistical Metrology Protocols

To capture the full behavior of analog devices, the following metrology protocols must be standardized:

* **Conductance PDF Measurement:**  Define standard procedures for measuring and reporting the full probability density functions (PDFs) of programmed conductance states across specified sample sizes and temperature conditions.  
* **Noise Power Spectral Density:**  Specify standard measurement parameters (e.g., bandwidth, integration time) for characterizing Random Telegraph Noise (RTN) and 1/f noise to quantify the device's noise floor.  
* **Analog Endurance:**  Propose a new functional definition for endurance. Instead of a simple binary pass/fail criterion, "Analog Endurance" shall be defined as the number of programming cycles before the state variance (σ\_G) exceeds a specified threshold for a given precision tier.

##### 4.2 Alignment with NIST-Aligned Standards

The proposed framework must align with the U.S. National Institute of Standards and Technology (NIST) neuromorphic measurement programs. This includes collaboration on developing standard metrics for attajoule-scale energy measurement and protocols for the statistical verification of probabilistic devices.

##### 4.3 Benchmarking Standards

Current industry benchmarks like MLPerf, which demand strict, deterministic accuracy thresholds, inherently disadvantage probabilistic analog hardware. This proposal advocates for the adoption of NeuroBench-style metrics that better reflect the operational value of analog AI. The following key metrics must be standardized:

* **Accuracy-per-Watt:**  The primary efficiency metric that captures the crucial trade-off between energy consumption and model performance.  
* **Synaptic Operations (SynOps):**  An event-based throughput metric essential for evaluating spiking neural networks and other dynamic workloads.  
* **Footprint:**  A measure of memory utilization and compute density, highlighting a key advantage of analog architectures.Once devices can be reliably specified and measured, they must be able to interface with the broader digital ecosystem.

#### 5.0 Proposed Framework for System Integration and Interfaces

Without standardized chiplet and software interfaces, analog AI will remain a collection of impressive but isolated lab curiosities, unable to form the robust, multi-vendor ecosystem necessary for commercial scale. To move from standalone components to a thriving ecosystem of interoperable hardware, analog AI accelerators require standardized digital interfaces for both chiplet-level integration and software-level control.

##### 5.1 Digital-Analog Chiplet Interconnect

This proposal recommends building upon the  **Universal Chiplet Interconnect Express (UCIe)**  standard to facilitate the integration of analog AI accelerators. Key extensions required for the UCIe specification include:

* Support for heterogeneous process nodes, allowing analog chiplets optimized for NVM to interface with leading-edge digital hosts.  
* Definition of CXL-compatible protocol modes for both memory-mapped access and streaming inference data.  
* A standard sideband channel dedicated to communicating calibration, health monitoring, and drift compensation data.

##### 5.2 Programming and Software Interface Standards

###### *5.2.1 Weight Programming Command Set*

A standard command set, analogous to JEDEC's commands for DDR memory, is needed to ensure software portability across different analog hardware. Essential commands to be standardized include:

* SET\_TARGET\_G(address, G\_target, tolerance)  
* VERIFY\_LOOP(max\_iterations, convergence\_threshold)  
* READ\_G\_DISTRIBUTION(address\_range)

###### *5.2.2 Hardware-Aware Training (HAT) Interface*

Hardware-Aware Training is the definitive algorithmic solution to the universal challenges of  **Stochasticity** ,  **Asymmetry** , and  **Noise Floor** . To operationalize HAT at an industry scale, we must standardize the bridge between hardware characterization and software frameworks. This requires:

* A  **Device Model Format**  for specifying hardware characteristics like noise, drift, and non-linearity. The model used by IBM's AIHWKit provides an excellent foundation.  
* A  **Noise Injection Protocol**  that defines how these hardware characteristics are translated into statistical perturbations within software training frameworks like PyTorch and TensorFlow.These interface standards are not only for performance and interoperability; they are also prerequisites for meeting the stringent requirements of regulatory and safety bodies.

#### 6.0 Alignment with Safety, Security, and Regulatory Frameworks

For analog AI to be deployed in high-stakes applications such as automotive, aerospace, and critical enterprise systems, its probabilistic nature must be reconciled with deterministic safety and regulatory standards.

##### 6.1 EU AI Act Compliance

Article 15 of the EU AI Act mandates verifiable "accuracy, robustness, and cybersecurity." The proposed standards provide a direct pathway to compliance by:

* Creating a framework for  **Robustness Certification** , where performance is demonstrated across the full statistical range of device behavior.  
* Establishing  **Hardware-Aware Training (HAT)**  as a recognized and standardized compliance tool for demonstrating mathematical robustness against hardware non-idealities.

##### 6.2 ISO 26262 Functional Safety (Automotive)

Applying the deterministic failure models of ISO 26262 to probabilistic hardware is a significant challenge. This framework proposes:

* Standardizing  **Functional Degradation Metrics** , which redefine "failure" not as a binary device fault but as an unacceptable drop in model accuracy below a safety-critical threshold.  
* Providing ASIL decomposition templates that pair analog inference cores (ASIL B/QM) with certified digital safety monitors (ASIL D) for redundancy and oversight.

##### 6.3 Security Standards (ISO/IEC 20897\)

The inherent device-to-device variability of analog hardware, traditionally a challenge, can be leveraged as a powerful security feature through  **Physical Unclonable Functions (PUFs)** . The framework must align with ISO/IEC 20897 by:

* Defining standard  **PUF Quality Metrics**  for uniqueness, steadiness, and randomness.  
* Establishing standards for validating analog noise as a certified source of entropy for true random number generation.These compliance frameworks are the final piece of the puzzle, providing the 'why' for standardization. The following roadmap outlines the 'how' and 'who'—a concrete plan for industry-wide execution.

#### 7.0 Standardization Roadmap and Call to Action

To prevent ecosystem fragmentation and unlock the immense commercial potential of analog AI, the industry must take urgent and coordinated action. This proposal outlines a clear roadmap for developing and implementing the necessary standards.

##### 7.1 Proposed Standards Body Alignment

Body,Scope,Priority  
JEDEC,"Analog memory specifications: endurance, retention, commands",HIGH  
IEEE P2415,Energy-proportional hardware abstraction for analog accelerators,HIGH  
IEEE P2851,Functional safety data exchange for analog reliability modeling,HIGH  
ISO/IEC JTC1/SC42,AI system vocabulary and risk management for hardware,MEDIUM  
UCIe Consortium,Chiplet interface extensions for analog accelerators,MEDIUM  
NeuroBench,Benchmarking metrics for neuromorphic and analog systems,MEDIUM

##### 7.2 Phased Implementation Timeline

* **Phase 1 (2025-2026): Foundation**  Establish the foundational standards for conductance state specification and drift characterization protocols.  
* **Phase 2 (2026-2027): Interface**  Define the programming command sets for JEDEC and the hardware-aware training (HAT) integration interfaces.  
* **Phase 3 (2027-2028): Compliance**  Develop and align standards with EU AI Act and ISO 26262 requirements to establish clear certification pathways.  
* **Phase 4 (2028+): Ecosystem**  Achieve full NeuroBench integration and extend standards to cover 3D architectures and emerging technologies like photonics.

##### 7.3 Key Recommendations and Call to Action

This proposal synthesizes a complex technical landscape into a set of actionable recommendations for the industry:

* **Adopt Statistical Specifications:**  Move from deterministic point values to probabilistic distributions (G\_T, σ\_G) to define analog weights.  
* **Standardize Drift Management:**  Establish universal drift characterization protocols and retention classes applicable across all analog technologies.  
* **Mandate Differential Encoding:**  Require G⁺/G⁻ differential pairs as the standard for representing signed weights to ensure inherent noise rejection.  
* **Establish HAT as a Compliance Tool:**  Position Hardware-Aware Training as the recognized bridge between hardware stochasticity and regulatory demands.  
* **Engage JEDEC for Analog Memory Standards:**  Initiate a new JEDEC workgroup to create standards for "Analog Weight Storage" devices.  
* **Adopt NeuroBench Metrics:**  Transition industry benchmarking from legacy accuracy-only metrics to energy-aware, event-based metrics suited for analog AI.The analog AI ecosystem is at a critical inflection point. Without coordinated standardization, we risk a future of incompatible proprietary interfaces and uncertain regulatory pathways. We urge semiconductor manufacturers, AI developers, and standards bodies to collaborate on developing these essential standards.**The future of energy-efficient AI computing depends on our ability to standardize the analog—transforming physics into reliable, interoperable, and certifiable intelligence.**

