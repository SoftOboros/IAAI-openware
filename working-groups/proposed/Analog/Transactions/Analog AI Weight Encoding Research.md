# **The Analog Manifold: Physical Encoding of Neural Parametrics in Next-Generation Neuromorphic Architectures**

## **1\. Introduction: The Physics of the Analog Manifold**

The trajectory of modern computing is currently negotiating a fundamental inflection point. For decades, the dominant paradigm has been the von Neumann architecture, a system predicated on the rigid separation of processing units and memory storage. While this digital abstraction has fueled the rise of general-purpose computing, it faces an existential limit when applied to the massively parallel, data-intensive workloads of artificial intelligence (AI) and neural network processing. The energy and latency penalties incurred by shuttling data between memory and the processor—the so-called "von Neumann bottleneck"—have spurred a resurgence in analog computing techniques.1  
This report investigates the "Analog Manifold," a conceptual and physical framework where neural weights are not stored as discrete binary abstractions but are encoded directly into the continuous physical properties of the computing substrate. In this regime, the mathematical parameters of a neural network—its synaptic weights—are instantiated as analog signals such as electrical conductance, magnetic polarization, optical phase, or charge density.3 The vector space of the neural network thus becomes a physical manifold, where computation is the natural evolution of the system's physical state according to Kirchhoff’s, Ohm’s, and Maxwell’s laws.1  
The transition to this analog paradigm requires a deep understanding of the device physics governing these weight-encoding mechanisms. Whether utilizing the stochastic migration of oxygen vacancies in resistive RAM (ReRAM), the crystallization kinetics of phase-change memory (PCM), or the spin-transfer torque in magnetic tunnel junctions (MTJs), the objective remains constant: to create a "neurotrophic" structure where hardware mimics the efficiency, plasticity, and density of biological neural systems.6 This analysis further explores the critical role of emerging electrical standards—specifically IEEE P2415 and P2851—and metrology initiatives by NIST and the IRDS in formalizing these technologies for commercial and industrial scale.7

### **1.1 The Imperative of In-Memory Computing**

The primary motivation for encoding weights into analog signals is to enable In-Memory Computing (IMC). In deep neural networks (DNNs), the dominant computational operation is Vector-Matrix Multiplication (VMM). In a digital system, VMM requires fetching weight data from memory, moving it to an Arithmetic Logic Unit (ALU), performing the multiplication, and writing the result back. This movement consumes orders of magnitude more energy than the computation itself.1  
Analog IMC circumvents this by utilizing the fundamental physics of the memory device to perform computation *in situ*.

* **Multiplication via Conductance:** If an input activation is represented as a voltage ($V$) applied to a word-line, and the synaptic weight is encoded as the conductance ($G$) of a memory cell, the resulting current ($I$) flowing through the device is the product $I \= V \\cdot G$, in accordance with Ohm’s Law.1  
* **Accumulation via Current Summation:** When multiple such devices are connected to a common bit-line, the total current is the algebraic sum of the individual currents ($I\_{total} \= \\Sigma (V\_i \\cdot G\_i)$), in accordance with Kirchhoff’s Current Law.1

This approach allows an entire array of weights to perform a matrix multiplication in a single clock cycle, offering potential speedups of 280× and energy efficiency improvements of 100× over state-of-the-art digital GPUs.1 However, the realization of this potential is contingent upon the ability to precisely encode and maintain analog values within the noisy, stochastic environment of nanoscale physics.

## ---

**2\. Resistive Encoding: The Memristive Substrate**

The most extensively researched class of analog weight encoding relies on modulating the electrical resistance of a two-terminal device. These devices, broadly termed "memristors," encode the synaptic weight as a non-volatile resistance state. The two primary technologies in this category are Resistive Random Access Memory (ReRAM) and Conductive Bridging RAM (CBRAM).

### **2.1 Physics of Filamentary Weight Modulation**

ReRAM devices typically consist of a metal-oxide dielectric layer (e.g., $HfO\_2$, $Ta\_2O\_5$, $TiO\_x$) sandwiched between two metal electrodes. The mechanism of weight encoding is the formation and rupture of conductive filaments (CFs) within this oxide layer.5

#### **2.1.1 Oxygen Vacancy Migration**

In valence change memory (VCM) ReRAM, the conductive filament is composed of oxygen vacancies ($V\_O$).

* **Potentiation (SET):** When a positive programming voltage is applied, oxygen ions are driven out of the lattice and towards the anode (often a reactive metal like Ti or Hf), leaving behind oxygen vacancies. These vacancies act as charge traps or dopants, creating a conductive path through the insulating dielectric. As more vacancies accumulate, the filament thickens or becomes more conductive, decreasing the device resistance. This analog increase in conductance represents the strengthening of a synaptic weight.5  
* **Depression (RESET):** Conversely, applying a negative voltage drives oxygen ions from the reservoir back into the filament, where they recombine with the vacancies. This ruptures the continuous conductive path or thins the filament, increasing the resistance and effectively "depressing" the synaptic weight.5

The "analog manifold" in ReRAM is thus defined by the continuous variation in the geometry and composition of this filament. Unlike binary storage, which operates between a high-resistance state (HRS) and a low-resistance state (LRS), analog encoding utilizes the intermediate states (IRS) to represent continuous values.5

#### **2.1.2 Electrochemical Metallization (CBRAM)**

In conductive bridging RAM (CBRAM), the filament is formed by metal cations (e.g., $Ag^+$, $Cu^+$) rather than oxygen vacancies.

* **Mechanism:** An oxidizable anode (e.g., Ag) releases cations into the electrolyte under bias. These cations migrate to the cathode, are reduced, and deposit to form a metallic filament.11  
* **Dual-Filament Switching:** Research utilizing in-situ transmission electron microscopy (TEM) on $Ta\_2O\_5$-based devices has revealed complex switching behaviors, such as dual-filament switching, where interactions between silver filaments and the host lattice create multiple stable conductance states.11

### **2.2 Stochasticity and Noise in the Analog Manifold**

A defining characteristic of resistive encoding is its inherent stochasticity. The migration of ions is a thermally activated probabilistic process.

* **Cycle-to-Cycle Variability:** The exact shape and conductivity of the filament change with every programming pulse. Even with identical voltage pulses, the resulting change in conductance ($\\Delta G$) varies, following a distribution rather than a deterministic trajectory.1  
* **Telegraph Noise:** Read noise, or Random Telegraph Noise (RTN), arises from the trapping and de-trapping of carriers at defect sites near the filament. This causes the conductance to fluctuate even when no programming pulse is applied, introducing "jitter" into the encoded weight.1

While this stochasticity is detrimental for precise numerical computing, it is increasingly viewed as a feature for "neurotrophic" systems. The probabilistic nature of ion migration can be leveraged to implement stochastic learning rules or to act as entropy sources for probabilistic neural networks, mirroring the noisy synaptic transmission found in biological brains.5

### **2.3 Challenges in Linearity and Symmetry**

For analog weights to be trained effectively using standard algorithms like Backpropagation, the device's response to programming pulses should ideally be linear and symmetric. However, ReRAM devices often exhibit strong non-linearity.

* **Asymmetry:** The SET process (filament formation) is often abrupt, driven by a positive feedback loop of Joule heating and field enhancement, while the RESET process (filament rupture) is more gradual. This asymmetry leads to biased weight updates and degrades training accuracy.1  
* **Optimization Frameworks:** To mitigate these physical non-idealities, generalized computational frameworks have been developed. These frameworks use "Hardware-Aware Training" (HAT) to characterize the device's specific non-linearities and adjust the training algorithm to compensate, ensuring that the software-trained weights can be optimally translated into the analog hardware manifold.12

## ---

**3\. Phase-Change Dynamics and Conductance Drift**

Phase-Change Memory (PCM) offers an alternative resistive encoding mechanism based on the reversible phase transition of chalcogenide materials, most commonly Germanium-Antimony-Tellurium (GST) alloys.1

### **3.1 Thermodynamic Encoding of Weights**

In PCM, the synaptic weight is encoded in the ratio of the crystalline phase to the amorphous phase within the cell volume.

* **Crystalline State (Low Resistance):** The crystalline phase of GST is highly conductive. Programming the device to this state involves heating the material above its crystallization temperature ($T\_c$) but below its melting point ($T\_m$) for a sufficient duration to allow crystal nucleation and growth.  
* **Amorphous State (High Resistance):** The amorphous phase is highly resistive. Creating this state (RESET) requires a high-current pulse to melt the material ($T \> T\_m$), followed by a rapid quench that freezes the disordered atomic structure before it can recrystallize.5

### **3.2 Creating the Analog Continuum**

The analog manifold in PCM is accessed primarily through the "partial crystallization" of the active volume. By modulating the amplitude and duration of the SET pulse (current), one can control the size of the crystalline region growing within the amorphous background. This allows for a continuum of resistance levels, effectively storing an analog weight.5  
Unlike ReRAM, where the filamentary nature leads to abrupt changes, PCM crystallization can be more gradually controlled, offering better linearity for the potentiation (weight increase) curve. However, the amorphization (RESET) process remains inherently abrupt, as it involves melting the entire active volume. This creates a fundamental asymmetry in the update dynamics, often necessitating "reset-and-set" programming strategies where a weight is fully erased before being reprogrammed to a new value.5

### **3.3 Conductance Drift and Reliability**

A critical physical limitation of PCM for analog weight storage is "conductance drift." The amorphous state of GST is thermodynamically unstable; over time, the atomic structure relaxes toward a lower-energy, more resistive glass state.

* **Drift Physics:** The conductance $G(t)$ decays over time following a power law: $G(t) \= G\_0 (t/t\_0)^{-\\nu}$, where $\\nu$ is the drift coefficient.1  
* **Implication for Weights:** This means that an encoded synaptic weight is not static; it gradually decreases in magnitude over time. In a neural network, this global drift can degrade inference accuracy as the effective weights shift away from their trained values.1  
* **Mitigation:** Advanced encoding techniques, such as using "drift-insensitive" differential pairs or frequent re-calibration, are required to stabilize the analog manifold against this thermodynamic relaxation.

## ---

**4\. Ferroelectric and Field-Effect Weight Encoding**

Moving beyond two-terminal resistive devices, Ferroelectric Field-Effect Transistors (FeFETs) and Floating-Gate Transistors integrate the weight storage directly into the gate stack of a transistor, offering a three-terminal device with distinct advantages in control and efficiency.

### **4.1 Ferroelectric Polarization Dynamics (FeFET)**

FeFETs utilize the ferroelectric properties of doped Hafnium Oxide ($HfO\_2$) to store information.13 Unlike traditional ferroelectrics like PZT, doped $HfO\_2$ is compatible with advanced CMOS manufacturing, making FeFETs a highly scalable candidate for high-density analog memory.

#### **4.1.1 Polarization as the State Variable**

The analog weight in a FeFET is encoded in the remanent polarization ($P\_r$) of the ferroelectric gate insulator.

* **Hysteresis and Minor Loops:** The relationship between polarization ($P$) and the applied electric field ($E$) describes a hysteresis loop. By applying voltage pulses of varying amplitude or width, the domain structure of the ferroelectric film can be partially switched. This accesses "minor loops" within the major hysteresis curve, allowing the polarization to be set to arbitrary intermediate values.13  
* **Threshold Voltage Modulation:** The polarization charge in the gate stack exerts an electric field on the transistor channel, shifting the threshold voltage ($V\_T$). A "positive" polarization attracts electrons, lowering $V\_T$ (high conductance/weight), while "negative" polarization repels them, raising $V\_T$ (low conductance/weight). The synaptic weight is thus read out as the channel current $I\_{DS}$ for a fixed read gate voltage.13

#### **4.1.2 Phase Stabilization and Reliability**

The stability of the analog weight depends on the crystalline phase of the $HfO\_2$.

* **Dopant Engineering:** Elements like Silicon (Si), Zirconium (Zr), and Lanthanum (La) are doped into the HfO2 lattice to stabilize the ferroelectric orthorhombic phase. Dopant engineering is crucial for minimizing the monoclinic phase, which is non-ferroelectric, and optimizing the "wake-up" and "fatigue" characteristics of the device.13  
* **Defect Dynamics:** Reliability issues such as "wake-up" (where polarization increases with cycling) and "fatigue" (where it decreases) are driven by the redistribution of oxygen vacancies and the pinning of domain walls. These dynamics represent a time-variant component of the analog manifold that must be modeled for robust operation.13

### **4.2 Floating-Gate and Analog Flash**

While typically associated with digital storage, Flash memory (based on floating-gate transistors) represents one of the most mature and precise methods for analog weight encoding.14

#### **4.2.1 Charge-Based Precision**

In a floating-gate device, the weight is physically encoded as the number of electrons stored on an electrically isolated polysilicon or charge-trap gate.

* **Resolution:** Because the number of electrons can be finely controlled via Fowler-Nordheim tunneling or Hot Carrier Injection, Flash cells can achieve very high precision—up to 8 bits (256 levels) or more per cell.15  
* **Subthreshold Operation:** To utilize these devices as analog multipliers, they are often operated in the subthreshold (weak inversion) regime. Here, the drain current is exponentially related to the gate voltage ($I\_{DS} \\propto \\exp(V\_{GS})$). This exponential behavior is naturally suited for implementing log-domain arithmetic or neuromorphic models that require exponential transfer functions.14

#### **4.2.2 SONOS and Charge Trapping**

Recent research highlights Silicon-Oxide-Nitride-Oxide-Silicon (SONOS) devices for analog learning.

* **Mechanism:** Charge is trapped in distinct sites within the silicon nitride layer. This offers better reliability and lower programming voltages compared to traditional floating gates.  
* **Symmetry:** SONOS devices operating in the subthreshold regime have demonstrated excellent symmetric write non-linearities, allowing neural networks to be trained to "ideal accuracies" comparable to floating-point software implementations.14

## ---

**5\. Spintronic and Magnetic Encoding Mechanisms**

Spintronics (Spin Transport Electronics) introduces a fundamentally different physics for weight encoding, leveraging the intrinsic spin of electrons rather than their charge. This offers "non-volatile" operation with infinite endurance and intrinsic radiation hardness.17

### **5.1 Magnetic Tunnel Junctions (MTJs)**

The MTJ is the core component of spintronic memory (MRAM). It consists of two ferromagnetic layers separated by a thin insulating barrier (typically $MgO$). One layer has a fixed magnetic orientation, while the other ("free layer") can rotate.

#### **5.1.1 Probabilistic Encoding in the Stochastic Manifold**

While commercial MRAM uses MTJs as binary switches (Parallel vs. Anti-Parallel), neuromorphic spintronics exploits the *instability* of these devices.

* **Superparamagnetism:** By reducing the volume of the free layer or tuning its magnetic anisotropy, the energy barrier between the P and AP states can be lowered to be comparable to thermal energy ($k\_B T$). The magnetization then fluctuates randomly between states.18  
* **Analog as Probability:** In this regime, the "analog weight" is not a static resistance but the *probability* ($P$) that the device is in the P state. This probability can be tuned continuously by a spin-polarized bias current. This turns the physical noise of the device into a computational resource, allowing for the direct hardware implementation of Boltzmann Machines and stochastic neural networks.18

### **5.2 Domain Wall and Skyrmion Weights**

To achieve deterministic analog weights in a single magnetic device, "spintronic memristors" utilize magnetic textures.

* **Domain Wall (DW) Racetracks:** A ferromagnetic wire (racetrack) contains a domain wall separating regions of opposite magnetization. The position of this domain wall can be moved continuously along the wire by applying current pulses (via Spin-Transfer Torque or Spin-Orbit Torque). The longitudinal position of the wall encodes the analog weight.20  
* **Skyrmions:** These are topologically protected magnetic quasiparticles—swirling spin textures that behave like particles. The presence or number of skyrmions in a given region can encode weight values. Skyrmion-based synapses are particularly promising due to their extremely low driving currents and stability.20

### **5.3 Spin-Torque Oscillators (STOs)**

Weights can also be encoded in the frequency domain. Spin-Torque Oscillators (STOs) generate continuous microwave oscillations when driven by DC current.

* **Mechanism:** The spin-transfer torque compensates for the magnetic damping, sustaining a steady-state precession of the magnetization.  
* **Coupled Dynamics:** Arrays of STOs can be coupled (magnetically or electrically) to synchronize their frequencies. This "synchronization map" creates an analog manifold defined by phase and frequency, mimicking the oscillatory binding observed in biological neural assemblies.21

## ---

**6\. Photonic and Optoelectronic Weight Encoding**

Photonic computing seeks to overcome the bandwidth and latency limitations of electronics by encoding weights into the properties of light—amplitude, phase, or wavelength.2

### **6.1 Mach-Zehnder Interferometers (MZIs)**

The workhorse of programmable photonics is the Mach-Zehnder Interferometer.

* **Encoding via Phase:** An input optical signal is split into two arms. A phase shifter (typically a thermo-optic or electro-optic heater) changes the refractive index of one arm, introducing a relative phase delay ($\\phi$). When the beams recombine, they interfere constructively or destructively.  
* **Matrix Operations:** The transmission $T$ is a function of $\\phi$, effectively multiplying the optical signal by a weight factor. Meshes of MZIs can be arranged to perform unitary matrix transformations at the speed of light.23

### **6.2 Microring Resonators (MRRs) and WDM**

To increase integration density, Microring Resonators are used in conjunction with Wavelength Division Multiplexing (WDM).

* **Broadcast and Weight:** A single waveguide carries multiple signals encoded on different wavelengths ($\\lambda\_1, \\lambda\_2, \\dots$). An MRR placed next to the waveguide acts as a weight for a specific wavelength.  
* **Resonance Tuning:** The weight is encoded by tuning the resonance frequency of the ring (via thermal or electrical modulation) relative to the signal wavelength. This determines how much light is coupled out of the bus and onto a summing detector.22  
* **Analog Manifold Challenges:** The "manifold" here is the spectral response of the ring. A major challenge is "thermal crosstalk"—heating one ring to change its weight can inadvertently shift the resonance of neighboring rings, requiring complex calibration and feedback control systems.22

### **6.3 Superconducting Optoelectronic Networks**

NIST is pioneering a hybrid approach that combines the speed of photonics with the energy efficiency of superconductors.

* **Mechanism:** Weights are stored in superconducting loops (flux quantization) or magnetic Josephson junctions. Single-photon detectors (like SNSPDs) receive optical spikes, which trigger Josephson junctions to perform threshold logic. This architecture allows for massive fan-out (thousands of connections) with femtojoule energy consumption per event, closer to biological efficiency than any other platform.25

## ---

**7\. CMOS-Native Analog Approaches**

Before the widespread adoption of exotic materials, standard CMOS technology has been engineered to support the analog manifold through circuit design techniques.

### **7.1 Switched Capacitor Networks**

Switched Capacitor (SC) circuits utilize the principle of charge conservation to perform analog arithmetic.27

* **Weight as Capacitance:** The synaptic weight is represented by the ratio of an input capacitor ($C\_{in}$) to an integration capacitor ($C\_{int}$). Since capacitors in modern CMOS processes (MIM caps) can be matched with high precision, SC circuits offer excellent linearity and accuracy (up to 10 bits).  
* **Discrete-Time Analog:** Computation occurs in discrete clock phases. Charge is sampled onto the input capacitor (Multiplication: $Q \= C \\cdot V$) and then transferred to the integrator (Summation). This avoids the static power consumption of resistive crossbars but requires larger area per synapse.29

### **7.2 Subthreshold Neuromorphic Circuits**

Pioneered by Carver Mead, this approach operates transistors in the subthreshold regime ($V\_{GS} \< V\_{th}$), where the transport physics is dominated by diffusion rather than drift.

* **Biological Isomorphism:** The current-voltage relationship is exponential, identical to the physics of ion channels in biological membranes. This allows silicon neurons to emulate the non-linear dynamics of Hodgkin-Huxley models with extremely low power (picoamperes).30  
* **Mismatch Challenges:** A major barrier is the high sensitivity to process variations (mismatch). Small variations in threshold voltage ($V\_{th}$) lead to exponential variations in current, making it difficult to define a consistent "analog manifold" across a large chip without extensive calibration or adaptive circuitry.31

## ---

**8\. The Neurotrophic Structure: Plasticity and Adaptation**

The term "neurotrophic" implies not just the structure of the brain (neuromorphic) but its ability to grow, adapt, and maintain itself. In hardware, this translates to systems that implement synaptic plasticity and structural learning.6

### **8.1 Implementing Spike-Timing-Dependent Plasticity (STDP)**

STDP is a fundamental biological learning rule where the synaptic weight changes based on the relative timing of pre-synaptic and post-synaptic spikes.

* **Analog Implementation:** In a ReRAM or PCM crossbar, STDP can be implemented by shaping the voltage waveforms of the spikes. If a pre-synaptic spike (row) and post-synaptic spike (column) overlap in time, the net voltage drop across the device exceeds the switching threshold, causing a conductance change ($\\Delta G$). The magnitude and direction (potentiation vs. depression) depend on the exact timing overlap.33  
* **Spintronic Plasticity:** In MTJs, the stochastic switching probability can be biased by the overlap of current pulses, allowing for a probabilistic implementation of STDP that is robust to noise and variation.17

### **8.2 Structural Plasticity and Self-Repair**

True neurotrophic systems may leverage the "analog manifold" for self-repair.

* **Red-Ox Front Propagation:** Research into polymer-based memristors (e.g., Polyaniline) suggests that the electrochemical "red-ox front" can propagate through the material, effectively "growing" conductive paths in response to activity. This mimics the physical growth of dendrites and axons, creating a dynamic hardware structure that evolves with the data it processes.4

## ---

**9\. Standards, Metrology, and the Regulatory Landscape**

For the "Analog Manifold" to transition from a research curiosity to a standardized industrial platform, a rigorous framework for metrology and interoperability is required. Bodies like the IEEE, NIST, and IRDS are actively establishing this infrastructure.

### **9.1 IEEE Standards for Neuromorphic Systems**

* **IEEE P2415 (Energy Proportional Systems):** This working group focuses on "Unified Hardware Abstraction and Layer for Energy Proportional Electronic Systems." In analog neuromorphic chips, energy consumption is highly data-dependent (e.g., high conductance weights draw more current). P2415 aims to define syntax and semantics to model and verify these unique energy profiles, enabling designers to optimize power management across the hardware-software stack.8  
* **IEEE P2851 (Functional Safety & Dependability):** Originally driven by automotive needs, IEEE P2851 is critical for deploying neuromorphic hardware in safety-critical systems (e.g., autonomous vehicles). It deals with the interoperability of safety data. A key challenge is modeling the reliability of analog weights—concepts like "conductance drift" in PCM or "retention loss" in ReRAM must be formalized as failure modes. The standard provides a format to exchange this dependability data, ensuring that a "forgetting" neural network is treated as a quantifiable risk.9

### **9.2 NIST Metrology for Synthetic Cortical Function**

The National Institute of Standards and Technology (NIST) recognizes that measuring the state of an "analog manifold" requires new tools.

* **The "Neuro-Scope":** Traditional logic analyzers cannot capture the collective dynamics of a million coupled oscillators. NIST is developing novel metrology, analogous to functional MRI (fMRI) for chips, to measure spatial and temporal correlations in high-density spiking systems. This includes advanced magnetic imaging to visualize the state of spintronic arrays in real-time.7  
* **Benchmarks:** NIST is also establishing benchmark datasets and task metrics that go beyond simple accuracy (e.g., MNIST) to measure energy-delay products, robustness to noise, and adaptability—metrics where analog systems theoretically excel.25

### **9.3 International Roadmap for Devices and Systems (IRDS)**

The IRDS (formerly ITRS) now explicitly includes Neuromorphic Computing in its "More-than-Moore" roadmap.

* **2024/2025 Outlook:** The roadmap identifies the need for "Intelligent Materials" and "Autonomous Machine Computing." It highlights the gap between current device capabilities (e.g., limited endurance of ReRAM) and the requirements of "Embodied AI," calling for materials that can support continuous learning without degradation.38  
* **Standardization of "Neurotrophic":** The roadmap encourages the community to standardize definitions—distinguishing between "Neuromorphic" (mimicking structure) and "Neurotrophic" (mimicking growth/repair)—to guide material science research towards self-healing and adaptive substrates.6

## ---

**10\. Comparative Analysis of Analog Encoding Techniques**

The following analysis synthesizes the physical mechanisms, advantages, and limitations of the discussed analog encoding approaches.

| Technology | Physical Encoding Mechanism | Analog State Variable | Precision / Dynamic Range | Key Reliability Challenge | Standards Context |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **ReRAM** | Oxygen Vacancy Filamentation | Conductance ($G$) | Low (2-4 bits), Stochastic | Asymmetric Non-Linearity, RTN | IEEE P2851 (Safety of drift/noise) |
| **PCM** | Amorphous/Crystalline Ratio | Phase / Resistance | Medium (3-5 bits) | Conductance Drift ($t^{-\\nu}$) | IRDS (Endurance for training) |
| **FeFET** | Ferroelectric Polarization ($P\_r$) | Threshold Voltage ($V\_T$) | High (4-6 bits) | Wake-up, Fatigue, Depolarization | IEEE P2415 (Energy of switching) |
| **Flash** | Charge Trapping (Electrons) | Threshold Voltage ($V\_T$) | Very High (8+ bits) | High Voltage Write, Endurance | Mature (JEDEC), migrating to AI |
| **MRAM** | Spin Orientation Probability | Probability ($P$) | Stochastic / Continuous | Low On/Off Ratio, Read Disturbance | NIST (Magnetic Metrology) |
| **Photonics** | Optical Interference / Resonance | Phase shift ($\\phi$) | High Bandwidth / Speed | Thermal Crosstalk, Footprint | IEEE P2415 (Laser power mgmt) |
| **Switched Cap** | Charge Conservation | Charge ($Q$) | High Linearity (8-10 bits) | Volatility, Area Scaling | Standard CMOS Design Flows |

## ---

**11\. Conclusion: Synthesizing the Analog Manifold**

The exploration of the Analog Manifold reveals a rich landscape where the boundaries between physics, computation, and biology blur. We are moving away from the era of "imposing" logic onto silicon—forcing transistors to act as binary switches—toward an era of "eliciting" computation from the native physics of materials.  
The resistance of a oxide filament, the spin of an electron, and the phase of a photon are no longer just physical artifacts to be managed; they are the fundamental alphabets of a new computational language. This language speaks in continuous variables, probabilities, and dynamic evolutions, much like the biological systems it aims to emulate.  
However, the path to widespread adoption is paved with challenges in reliability, variability, and standardization. The stochastic nature of the analog manifold, while useful for certain probabilistic models, remains a hurdle for precision tasks. The work of IEEE P2415 and P2851 in creating a grammar for energy and safety, and the metrological advances by NIST in visualizing these systems, are as critical as the material breakthroughs themselves.  
As we progress toward the 2025/2030 horizons outlined by the IRDS, the successful realization of neurotrophic structures will depend on a holistic approach that co-designs the material physics, the circuit architecture, and the algorithmic learning rules. Only through this unified effort can we fully leverage the analog manifold to break the von Neumann bottleneck and usher in the age of truly cognitive hardware.

#### **Works cited**

1. Optimised weight programming for analogue memory-based deep ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC9247051/](https://pmc.ncbi.nlm.nih.gov/articles/PMC9247051/)  
2. Photonic neural networks and optics-informed deep learning fundamentals \- AIP Publishing, accessed December 24, 2025, [https://pubs.aip.org/aip/app/article/9/1/011102/3161086/Photonic-neural-networks-and-optics-informed-deep](https://pubs.aip.org/aip/app/article/9/1/011102/3161086/Photonic-neural-networks-and-optics-informed-deep)  
3. 2007 MY OBD System Operation Summary for Gasoline Engines \- Injector Dynamics, accessed December 24, 2025, [http://injectordynamics.com/wp-content/uploads/2013/12/2007-OBDII.pdf](http://injectordynamics.com/wp-content/uploads/2013/12/2007-OBDII.pdf)  
4. (PDF) Red-Ox Front Propagation in Polyaniline-Polymer Electrolyte System as a Basis for Spiking and Rate-Based Neural Networks and Multibit ReRAM \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/370665844\_Red-Ox\_Front\_Propagation\_in\_Polyaniline-Polymer\_Electrolyte\_System\_as\_a\_Basis\_for\_Spiking\_and\_Rate-Based\_Neural\_Networks\_and\_Multibit\_ReRAM](https://www.researchgate.net/publication/370665844_Red-Ox_Front_Propagation_in_Polyaniline-Polymer_Electrolyte_System_as_a_Basis_for_Spiking_and_Rate-Based_Neural_Networks_and_Multibit_ReRAM)  
5. Signal and noise extraction from analog memory elements for neuromorphic computing, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC5974407/](https://pmc.ncbi.nlm.nih.gov/articles/PMC5974407/)  
6. Video Transcript \- Cantley's ENDS Lab Tour \- Boise State University, accessed December 24, 2025, [https://www.boisestate.edu/coen-crg/video-transcript-cantleys-ends-lab-tour/](https://www.boisestate.edu/coen-crg/video-transcript-cantleys-ends-lab-tour/)  
7. Neuromorphic systems | NIST \- National Institute of Standards and Technology, accessed December 24, 2025, [https://www.nist.gov/programs-projects/neuromorphic-systems](https://www.nist.gov/programs-projects/neuromorphic-systems)  
8. AGGIOS, Inc. October 25, 2016 09:00 ET AGGIOS DONATES ITS UNIFIED HARDWARE ABSTRACTION FORMAT TO IEEE STANDARDS PROJECT, accessed December 24, 2025, [https://www.aggios.com/news/AGGIOS\_IEEE\_Donation\_Press\_Release\_2016\_10\_26.pdf](https://www.aggios.com/news/AGGIOS_IEEE_Donation_Press_Release_2016_10_26.pdf)  
9. P2851™/D1.0 \- IEEE Standards \- draft standard template, accessed December 24, 2025, [https://ieeexplore.ieee.org/ielD/10093820/10093821/10093822.pdf](https://ieeexplore.ieee.org/ielD/10093820/10093821/10093822.pdf)  
10. A Survey of ReRAM-Based Architectures for Processing-In-Memory and Neural Networks, accessed December 24, 2025, [https://knowen-production.s3.amazonaws.com/uploads/attachment/file/5193/make-01-00005.pdf](https://knowen-production.s3.amazonaws.com/uploads/attachment/file/5193/make-01-00005.pdf)  
11. Direct Observation of Dual-Filament Switching Behaviors in Ta 2 O 5 \-Based Memristors, accessed December 24, 2025, [https://www.researchgate.net/publication/313414640\_Direct\_Observation\_of\_Dual-Filament\_Switching\_Behaviors\_in\_Ta\_2\_O\_5\_-Based\_Memristors](https://www.researchgate.net/publication/313414640_Direct_Observation_of_Dual-Filament_Switching_Behaviors_in_Ta_2_O_5_-Based_Memristors)  
12. Optimised weight programming for analogue memory-based deep neural networks, accessed December 24, 2025, [https://pure.psu.edu/en/publications/optimised-weight-programming-for-analogue-memory-based-deep-neura/](https://pure.psu.edu/en/publications/optimised-weight-programming-for-analogue-memory-based-deep-neura/)  
13. Hafnium oxide-based ferroelectric field effect transistors: From ..., accessed December 24, 2025, [https://pubs.aip.org/aip/jap/article/138/1/010701/3351745/Hafnium-oxide-based-ferroelectric-field-effect](https://pubs.aip.org/aip/jap/article/138/1/010701/3351745/Hafnium-oxide-based-ferroelectric-field-effect)  
14. Using Floating Gate Memory to Train Ideal Accuracy Neural Networks \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/331623576\_Using\_Floating\_Gate\_Memory\_to\_Train\_Ideal\_Accuracy\_Neural\_Networks](https://www.researchgate.net/publication/331623576_Using_Floating_Gate_Memory_to_Train_Ideal_Accuracy_Neural_Networks)  
15. 8-bit states in 2D floating-gate memories using gate-injection mode for large-scale convolutional neural networks \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11920423/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11920423/)  
16. Flash Memory for Synaptic Plasticity in Neuromorphic Computing: A Review \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2313-7673/10/2/121](https://www.mdpi.com/2313-7673/10/2/121)  
17. Neuromorphic Spintronics \- PMC, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC7754689/](https://pmc.ncbi.nlm.nih.gov/articles/PMC7754689/)  
18. Spintronics for Neuromorphic Computing | NIST, accessed December 24, 2025, [https://www.nist.gov/programs-projects/spintronics-neuromorphic-computing](https://www.nist.gov/programs-projects/spintronics-neuromorphic-computing)  
19. Spintronic Technologies. a) Magnetic tunnel junction. b) Spin–orbit... | Download Scientific Diagram \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/figure/Spintronic-Technologies-a-Magnetic-tunnel-junction-b-Spin-orbit-torque-c-Magnetic\_fig2\_352765631](https://www.researchgate.net/figure/Spintronic-Technologies-a-Magnetic-tunnel-junction-b-Spin-orbit-torque-c-Magnetic_fig2_352765631)  
20. Research – INC Lab, accessed December 24, 2025, [https://utinclab.com/research/](https://utinclab.com/research/)  
21. Neuromorphic Computing | NIST \- National Institute of Standards and Technology, accessed December 24, 2025, [https://www.nist.gov/programs-projects/neuromorphic-computing](https://www.nist.gov/programs-projects/neuromorphic-computing)  
22. Design automation of photonic resonator weights \- PMC, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11501581/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11501581/)  
23. Si Microring Resonator Crossbar Array for On-Chip Inference and Training of the Optical Neural Network | ACS Photonics \- ACS Publications, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acsphotonics.1c01777](https://pubs.acs.org/doi/10.1021/acsphotonics.1c01777)  
24. Microring-based programmable coherent optical neural networks \- Optica Publishing Group, accessed December 24, 2025, [https://opg.optica.org/abstract.cfm?uri=oe-31-12-18871](https://opg.optica.org/abstract.cfm?uri=oe-31-12-18871)  
25. NIST's Superconducting Hardware Could Scale Up Brain-Inspired Computing, accessed December 24, 2025, [https://www.nist.gov/news-events/news/2022/10/nists-superconducting-hardware-could-scale-brain-inspired-computing](https://www.nist.gov/news-events/news/2022/10/nists-superconducting-hardware-could-scale-brain-inspired-computing)  
26. Physics and Hardware for Intelligence | NIST, accessed December 24, 2025, [https://www.nist.gov/programs-projects/physics-and-hardware-intelligence](https://www.nist.gov/programs-projects/physics-and-hardware-intelligence)  
27. Design, Implementation, and Analysis of an Integrated Switched ..., accessed December 24, 2025, [https://re.public.polimi.it/retrieve/ae932748-d1f1-4038-b5d5-886db9911c82/Design\_Implementation\_and\_Analysis\_of\_an\_Integrated\_Switched\_Capacitor\_Analog\_Neuron\_for\_Edge\_Computing\_AI\_Accelerators\_2025.pdf](https://re.public.polimi.it/retrieve/ae932748-d1f1-4038-b5d5-886db9911c82/Design_Implementation_and_Analysis_of_an_Integrated_Switched_Capacitor_Analog_Neuron_for_Edge_Computing_AI_Accelerators_2025.pdf)  
28. Programmable switched-capacitor neural network for MVDR beamforming \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/document/485203/](https://ieeexplore.ieee.org/document/485203/)  
29. MINIMALIST: switched-capacitor circuits for efficient in-memory computation of gated recurrent units \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2505.08599](https://arxiv.org/pdf/2505.08599)  
30. A CMOS+X Spiking Neuron With On-Chip Machine Learning \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2512.03966v2](https://arxiv.org/html/2512.03966v2)  
31. Switched-capacitor realization of presynaptic short-term-plasticity and stop-learning synapses in 28 nm CMOS \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2015.00010/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2015.00010/full)  
32. An adaptable neuromorphic model of orientation selectivity based on floating gate dynamics, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2014.00054/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2014.00054/full)  
33. Memristive and CMOS Devices for Neuromorphic Computing \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC6981548/](https://pmc.ncbi.nlm.nih.gov/articles/PMC6981548/)  
34. IEEE P2415: Unified HW Abstraction & Layer for Energy Proportional Electronic Systems, accessed December 24, 2025, [https://semiengineering.com/knowledge\_centers/standards-laws/standards/ieee-p2415/](https://semiengineering.com/knowledge_centers/standards-laws/standards/ieee-p2415/)  
35. Functional Safety Standards: IEEE P2851 Road Map, accessed December 24, 2025, [https://ieeexplore.ieee.org/iel8/2/10937984/10938016.pdf](https://ieeexplore.ieee.org/iel8/2/10937984/10938016.pdf)  
36. IEEE 2851 STANDARDS \- UNECE, accessed December 24, 2025, [https://unece.org/sites/default/files/2024-05/GRVA-19-43e.pdf](https://unece.org/sites/default/files/2024-05/GRVA-19-43e.pdf)  
37. Neuromorphic Device Measurements | NIST, accessed December 24, 2025, [https://www.nist.gov/programs-projects/neuromorphic-device-measurements](https://www.nist.gov/programs-projects/neuromorphic-device-measurements)  
38. Neuromorphic Computing Roadmap 2025 \- Topsector ICT, accessed December 24, 2025, [https://digital-holland.nl/assets/images/default/Roadmap-Neuromorphic-Computing-2025-Short-1.0.pdf](https://digital-holland.nl/assets/images/default/Roadmap-Neuromorphic-Computing-2025-Short-1.0.pdf)  
39. Long Version | Neuromorphic Computing Roadmap \- Topsector ICT, accessed December 24, 2025, [https://digital-holland.nl/assets/images/default/Roadmap-Neuromorphic-Computing-2025-Long-1.0.pdf](https://digital-holland.nl/assets/images/default/Roadmap-Neuromorphic-Computing-2025-Long-1.0.pdf)  
40. 2024 EDITION \- IRDS \- IEEE, accessed December 24, 2025, [https://irds.ieee.org/images/files/pdf/2024/2024IRDS\_WP\_AMC.pdf](https://irds.ieee.org/images/files/pdf/2024/2024IRDS_WP_AMC.pdf)