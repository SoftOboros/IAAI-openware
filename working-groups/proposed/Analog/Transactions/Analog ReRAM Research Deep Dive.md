# **Resistive Encoding for AI Weights: Physics, Analog Manifold, and Neurotrophic Structures**

## **1\. Introduction: The Physical Basis of Neural Computation**

The trajectory of modern artificial intelligence has been defined by the exponential scaling of parameter counts, moving from simple multi-layer perceptrons to deep neural networks (DNNs) and transformers comprising hundreds of billions of weights. However, this growth has collided with a fundamental barrier in computing architecture: the von Neumann bottleneck. In conventional systems, the physical separation of processing units (CPUs, GPUs) from memory storage necessitates the constant shuttling of data, a process that consumes orders of magnitude more energy than the computation itself. The concept of In-Memory Computing (IMC) seeks to dismantle this bottleneck by physically colocating storage and processing. At the forefront of this revolution is resistive encoding—the use of non-volatile memory devices, specifically resistive random-access memory (ReRAM or RRAM) and memristors, to represent synaptic weights directly as physical conductance values.  
This report provides an exhaustive analysis of the resistive encoding landscape. We explore the underlying physics of switching mechanisms, dissect the complex "analog manifold" that governs weight representation, and evaluate the neurotrophic circuit structures designed to harness these devices for high-precision computing. By synthesizing insights from material science, device physics, and circuit theory, we aim to provide a definitive reference on the state of resistive AI hardware.

## **2\. Fundamental Physics of Resistive Switching**

The core functional unit of a neuromorphic accelerator is the artificial synapse. Unlike a binary transistor, which operates as a switch, a synaptic device must provide a continuum of stable conductance states to emulate the synaptic weights of a neural network. The physics governing this variable conductance is rich and varied, depending heavily on the material system and the thermodynamic forces at play.

### **2.1. Valence Change Mechanism (VCM)**

The Valence Change Mechanism (VCM) represents the most widely studied and industrially mature mode of resistive switching, particularly in transition metal oxides (TMOs) such as hafnium oxide ($HfO\_x$), tantalum oxide ($TaO\_x$), and titanium oxide ($TiO\_x$). The mechanism is fundamentally anionic, driven by the migration of oxygen vacancies ($V\_O^{\\bullet\\bullet}$) within the oxide lattice.1

#### **2.1.1. Thermodynamic Origins and Filament Formation**

In a pristine TMO film, the lattice is generally insulating. The initialization of the device, known as "electroforming," involves the application of a high electric field that triggers a soft breakdown. This field lowers the formation energy of oxygen vacancies, effectively ripping oxygen ions from the lattice and driving them toward the anode, where they are often stored in a reactive capping layer (e.g., Ti or Hf). The remaining trail of localized oxygen vacancies forms a conductive filament (CF) that bridges the top and bottom electrodes.  
The physics of this filament are governed by the interplay of electric field-driven drift and thermal diffusion. The Soret effect (thermodiffusion) plays a critical role; as the filament conducts current, Joule heating creates steep temperature gradients that can oppose or assist the ionic drift depending on the sign of the Soret coefficient.

#### **2.1.2. The Dynamics of Bipolar Switching**

VCM devices typically exhibit bipolar switching behavior, meaning the SET (transition to Low Resistance State, LRS) and RESET (transition to High Resistance State, HRS) operations occur at opposite voltage polarities.

* **SET Process:** A positive voltage applied to the active electrode repels oxygen vacancies from the reservoir interface back into the bulk oxide, or alternatively, drives oxygen ions away from the filament tip. This re-establishes the percolation path of defects. The conductivity is often metallic-like or governed by hopping transport through defect states.  
* **RESET Process:** A negative voltage drives oxygen ions (stored in the capping layer or interface) back toward the filament. These ions recombine with the vacancies, effectively "oxidizing" the filament and creating a tunneling gap. This gap determines the resistance of the HRS. The relationship between the gap distance ($d$) and resistance ($R$) is exponential ($R \\propto \\exp(d)$), which explains the vast dynamic range (On/Off ratios of $10^3$ to $10^7$) but also poses challenges for linear analog control.1

#### **2.1.3. Conduction Mechanisms in the Manifold**

The "analog" nature of VCM arises from the modulation of the filament's geometry or the defect density within it.

* **LRS Regime:** Conduction is often Ohmic ($I \\propto V$), dominated by the continuous metallic sub-oxide phase (e.g., Magnéli phases in $TiO\_{2-x}$).  
* **HRS Regime:** As the filament ruptures, conduction shifts to trap-assisted tunneling (TAT) or Poole-Frenkel emission, where electrons hop between discrete defect sites. This transition leads to highly non-linear I-V characteristics in the high-resistance states, complicating weight readout at low voltages.

### **2.2. Electrochemical Metallization (ECM)**

Electrochemical Metallization (ECM) memories, also known as Conductive Bridge RAM (CBRAM), operate on a fundamentally different principle: the migration of active metal cations rather than oxygen anions. These systems typically pair an electrochemically active metal anode (Silver \- Ag, or Copper \- Cu) with an inert cathode (Platinum \- Pt, Tungsten \- W, or Gold \- Au), separated by a solid electrolyte (e.g., $GeS$, $SiO\_2$, $Al\_2O\_3$).1

#### **2.2.1. Cationic Nucleation and Growth**

The switching process begins with the anodic oxidation of the active metal ($Ag \\to Ag^+ \+ e^-$). The cations dissolve into the electrolyte and drift under the electric field toward the cathode. Upon reaching the cathode, they are reduced ($Ag^+ \+ e^- \\to Ag$) and nucleate to form a metallic protrusion. This protrusion grows back toward the anode—a process often described as "dendritic" growth—until it bridges the gap, switching the device to the LRS.  
Recent *in situ* Transmission Electron Microscopy (TEM) studies have provided atomic-scale visualization of this process. For instance, in $Cu-GeS$ systems, the formation of nanocrystalline Cu filaments is observed. In $Pt/Ta\_2O\_5/Ru$ structures, Ruthenium (Ru) ions migrate to form discontinuous Ru-rich nanoclusters.3

#### **2.2.2. Quantum Conductance and Analog Behavior**

ECM devices are often characterized by extremely abrupt switching due to the high mobility of Ag/Cu ions and the positive feedback loop of filament growth (as the gap narrows, the field intensifies, accelerating growth). However, in the regime of atomic-scale filaments, ECM devices can exhibit "quantized conductance," where the conductance changes in discrete steps of $G\_0 \= 2e^2/h$.  
For analog computing, this abruptness is a drawback. However, specific material engineering can induce analog behavior. For example, using Ru mobile species results in a dispersed filament composed of nanoclusters. The conduction between these clusters is governed by tunneling, which allows for a more gradual, continuous modulation of resistance compared to the snap-action of a solid Ag wire.3

### **2.3. The Filament Conductivity Change Mechanism (FCM)**

While VCM and ECM rely on the geometric rupture of a filament, a third, more recently formalized mechanism offers superior linearity for analog weights: the Filament Conductivity Change Mechanism (FCM). This mechanism is distinguished by the preservation of the filament's physical continuity during switching.4

#### **2.3.1. Dual Ohmic Contacts**

Standard VCM devices often utilize a Schottky contact at one interface to rectify current and enhance the On/Off ratio. In contrast, FCM devices are engineered with dual ohmic contacts. This structural difference fundamentally alters the switching physics. Instead of creating an insulating gap (which leads to exponential resistance changes), the electric field modulates the **stoichiometry** or the **radial dimensions** of the filament core without breaking it.

#### **2.3.2. Analog Linearity**

Because the filament remains intact, the resistance scaling follows a more linear relationship ($R \\propto \\rho L / A$) rather than an exponential tunneling relationship. This "analog modulation" is crucial for implementing synaptic learning rules like Spike-Timing-Dependent Plasticity (STDP) or Backpropagation, which require symmetric and gradual weight updates. In systems like Ti/SiOx/Au, this mechanism allows for high-precision analog tuning that is robust against the abrupt "reset" often seen in filamentary devices.5

### **2.4. Interfacial vs. Filamentary Switching Regimes**

The spatial distribution of the switching event dictates the scaling properties and noise characteristics of the device.

* **Filamentary Switching:** The standard mode for most binary ReRAM. The current is confined to a nanoscopic path ($\<10$ nm diameter). Consequently, the LRS resistance is largely independent of the macroscopic device area. While this allows for extreme scaling (down to \<10 nm nodes), it introduces significant stochasticity, as the behavior is controlled by a small number of atoms.  
* **Interfacial Switching:** In complex oxides like $Pr\_{0.7}Ca\_{0.3}MnO\_3$ (PCMO) or engineered bi-layers, the resistance change occurs uniformly over the entire electrode interface.6 This is often driven by the modulation of a Schottky barrier height or width across the whole contact area.  
  * **Area Dependency:** A hallmark of interfacial switching is that resistance scales inversely with device area ($R \\propto 1/Area$).  
  * **Implications:** Interfacial switching essentially "averages out" the local variations of individual defects, resulting in far superior device-to-device uniformity and analog linearity. However, this comes at the cost of higher absolute currents (if the area is large) or difficulty in scaling to extremely small nodes without the resistance becoming too high.5

## **3\. Material Landscapes for Resistive Weights**

The choice of material stack determines the dominant switching mechanism, the voltage window, and the retention characteristics.

### **3.1. Binary and Ternary Oxides**

* **Hafnium Oxide ($HfO\_x$):** The industry workhorse due to its omnipresence in CMOS high-k gate dielectrics. $HfO\_x$-based VCM devices are mature but suffer from high variability and non-linear weight updates.  
* **Silicon Oxide ($SiO\_x$):** Historically an insulator, $SiO\_x$ becomes a potent memristive material when doped or interfaced with active metals. In Ti/SiOx/Au stacks, a unique co-existence of mechanisms has been observed. At low currents (interfacial regime), the device shows highly linear, area-dependent switching. At high currents (filamentary regime), it transitions to binary, area-independent behavior. This tunability makes $SiO\_x$ versatile for both digital memory and analog synapses.5  
* **PCMO ($PrCaMnO\_3$):** A perovskite oxide known for its strong interfacial switching properties. PCMO-based synapses exhibit excellent analog linearity but are often incompatible with standard CMOS thermal budgets and materials.6

### **3.2. 2D Materials and Atomristors**

The atomic thinness of 2D materials like Molybdenum Disulfide ($MoS\_2$) and Hexagonal Boron Nitride (h-BN) offers the ultimate limit in vertical scaling, reducing the "stochastic volume" where random variations can occur.

* **Atomristors:** These devices leverage defects (e.g., sulfur vacancies in $MoS\_2$) or phase transitions (2H semiconducting to 1T metallic phase) to achieve switching. Monolayer $MoS\_2$ devices have demonstrated low-voltage operation and high integration density.  
* **Heterostructures:** Vertical integration of 2D materials allows for the creation of "dual-barrier" structures. For example, a graphene layer can be inserted between the electrode and the oxide to act as an ion barrier, preventing the over-diffusion of metal ions and enhancing thermal stability.8

### **3.3. Organic and Halide Perovskite Systems**

* **Organic Memristors:** Polymers like azo-compounds ($PPMAz-Py^+Br^-$) offer the ability to tune switching properties through molecular synthesis. They can exhibit unique "self-healing" properties and multi-level storage capabilities, though their thermal stability remains a concern for integration.9  
* **Halide Perovskites (HPs):** Known for solar cells, HPs possess mobile ions ($I^-$, $MA^+$) that cause hysteresis—a defect in photovoltaics but a feature for memristors. HP memristors combine characteristics of VCM and ECM and can be integrated into optoelectronic systems for in-sensor computing.10

## **4\. The Analog Manifold: Signal Integrity and Noise**

The "Analog Manifold" refers to the multi-dimensional state space of conductance values available for computation. In an ideal world, this manifold is continuous and deterministic. In reality, it is granular, noisy, and time-variant.

### **4.1. Random Telegraph Noise (RTN)**

RTN is a stochastic phenomenon observed as discrete jumps in conductance between two or more levels while the device is under constant bias.

* **Physical Origin:** It originates from the capture and emission of single charge carriers by defects (traps) located near the conduction path. The trapping of an electron creates a Coulombic repulsion that effectively narrows the filament, increasing resistance. When the electron is emitted, the channel widens, and resistance drops.11  
* **Impact on Inference:** In a neural network, RTN effectively adds noise to the weights ($W\_{noisy} \= W \+ \\eta\_{RTN}$). While large-amplitude RTN is detrimental, small fluctuations can be tolerated by robust networks.  
* **Probabilistic Computing:** Interestingly, this intrinsic noise is being repurposed. The stochastic switching probability of a device in the presence of thermal noise can be used to implement "stochastic rounding" or to generate entropy for Bayesian neural networks, turning a hardware bug into a computational resource.12

### **4.2. Conductance Drift and Relaxation**

Unlike the rapid fluctuations of RTN, conductance drift is a slow, unidirectional decay of the programmed state over time.

* Mechanism: After programming, the ion distribution within the filament is often thermodynamically unstable (high concentration gradient). Over time, ions diffuse back toward equilibrium, or the amorphous matrix undergoes structural relaxation. This is phenomenologically modeled as:

  $$G(t) \= G\_0 \\cdot t^{-\\nu}$$

  where $\\nu$ is the drift coefficient.13  
* **The "Forgetting" Problem:** This drift causes the accuracy of the neural network to degrade over time. In analog accelerators, this necessitates periodic re-calibration or "refreshing" of weights, which consumes energy and downtime.

### **4.3. Linearity and Symmetry of Weight Updates**

For on-chip training, the most critical attribute of the analog manifold is the linearity of the weight update.

* **The Non-Linearity Challenge:** Filamentary devices often exhibit highly non-linear updates. Potentiation (increasing conductance) might be abrupt (positive feedback), while depression (decreasing conductance) is gradual. This asymmetry leads to a severe degradation in training accuracy compared to floating-point software training.  
* **Interfacial Solutions:** As discussed, interfacial switching mechanisms (like in $SiO\_x$ or PCMO) offer far superior linearity because the change is distributed, avoiding the "runaway" growth dynamics of filaments.5

**Table 1: Impact of Non-Idealities on Neural Network Performance**

| Non-Ideality | Physical Cause | Impact on Neural Network | Mitigation Strategy |
| :---- | :---- | :---- | :---- |
| **RTN** | Single trap capture/emission | Inference noise, weight fluctuation | Differential pairs (2T2R), larger device area |
| **Drift** | Ionic diffusion / Relaxation | Accuracy degradation over time | Periodic refreshing, drift-aware training |
| **Non-Linearity** | Filament growth feedback | Poor training convergence | Linear material stacks (FCM), incremental pulse trains |
| **Asymmetry** | Different SET/RESET kinetics | Bias in weight updates | 2T2R architecture, bidirectional updates |
| **Device Variability** | Stochastic filament nucleation | Layer-wise error accumulation | Batch normalization, MTRD training |

11

## **5\. Neurotrophic Circuit Architectures**

The physical integration of resistive elements into a computational array requires careful architectural choices to manage the trade-offs between density, precision, and energy.

### **5.1. The Passive Crossbar (0T1R)**

The simplest architecture is the passive crossbar, consisting of a grid of perpendicular wires with a memristor at each intersection.

* **Density:** Ideally $4F^2$ (where F is the feature size), offering the highest possible memory density.  
* **Sneak Paths:** The lack of a transistor means that current can flow through unselected cells in the array, corrupting the read operation. This necessitates the use of a highly non-linear selector device (e.g., threshold switch) in series with the memristor, or a "self-rectifying" memristor.16  
* **IR Drop:** The finite resistance of the metal lines causes a voltage drop along the array. This means cells far from the drivers see a lower voltage than intended, leading to systematic errors in the vector-matrix multiplication. This limits the maximum practical size of passive arrays (typically to $64 \\times 64$ or $128 \\times 128$).17

### **5.2. Active 1T1R Architectures**

To solve the sneak path problem, the standard industrial approach pairs each memristor with a transistor (1T1R).

* **Isolation:** The transistor acts as a perfect selector, cutting off sneak paths and allowing for precise control of the compliance current during programming.  
* **Density Penalty:** The cell size is dominated by the transistor, typically increasing to $12-20 F^2$.  
* **Unipolar Weights:** A single 1T1R cell provides a positive conductance $G$. However, neural network weights can be negative. Implementing signed weights with 1T1R requires referencing a separate column or complex digital subtraction.

### **5.3. Differential 2T2R Structures**

The 2T2R architecture is the gold standard for high-precision analog inference. It uses two differential 1T1R cells (representing $G^+$ and $G^-$) to encode a single weight $W$.

* **Differential Encoding:** The synaptic weight is defined as $W \= G^+ \- G^-$. This allows for native representation of positive, negative, and zero weights.  
* **Common-Mode Rejection:** Since both devices are physically close, global variations (like temperature fluctuations or power supply noise) affect both $G^+$ and $G^-$ similarly. The subtraction $G^+ \- G^-$ cancels out these common-mode errors, significantly improving the signal-to-noise ratio (SNR).  
* **MoS2 2T2R Implementation:** Recent work has demonstrated a monolithic 3D-integrated 2T2R array using $MoS\_2$ FETs as selectors and $Al\_2O\_3$ memristors. By connecting the sources of the differential pair to a common source line, the subtraction is performed directly in the analog domain (Kirchhoff’s Current Law), simplifying the peripheral circuitry.18

**Table 2: Comparison of Array Architectures**

| Feature | Passive (0T1R) | Active (1T1R) | Differential (2T2R) |
| :---- | :---- | :---- | :---- |
| **Cell Size** | $4F^2$ (High Density) | $15-20 F^2$ | $30-40 F^2$ |
| **Sneak Paths** | Severe (Requires Selector) | Eliminated | Eliminated |
| **Weight Sign** | Positive Only | Positive Only | Native Signed ($+/-$) |
| **Linearity** | Device Dependent | Improved via Gate Control | Highest (Differential Cancellation) |
| **Noise Immunity** | Low | Medium | High (Common-mode rejection) |
| **Use Case** | Dense Storage / Low-End Inference | General Purpose Inference | High-Precision / Training |

17

## **6\. Algorithmic Co-Design: Taming the Analog Imperfections**

Given the inherent stochasticity of the physics, hardware perfection is unattainable. The solution lies in "Hardware-Aware" algorithms that embrace these imperfections.

### **6.1. Multi-Teacher Robust Distillation (MTRD)**

The MTRD framework is a novel training strategy designed to robustify neural networks against memristor non-idealities without requiring complex on-chip retraining.

* **Concept:** It employs a knowledge distillation approach. A "clean" teacher model (trained on ideal weights) and multiple "robust" teacher models (trained with injected noise representing drift, RTN, and variability) concurrently guide a "student" model.  
* **Mechanism:** The student model learns to find weight solutions that lie in "flat" minima of the loss landscape—regions where small perturbations in weights (due to hardware noise) do not result in significant accuracy loss. This effectively inoculates the network against the physical flaws of the ReRAM array.15

### **6.2. Write-Verify and Pulse Modulation**

To program a device to a precise analog state, a simple "blind" write pulse is insufficient due to probabilistic switching.

* **Iterative Write-Verify:** The controller applies a write pulse, reads the conductance, compares it to the target, and applies a corrective pulse if necessary. This loop continues until the conductance is within a specified tolerance. While accurate, this is slow and energy-intensive.17  
* **Pulse Train Modulation:** Instead of varying the amplitude of the write voltage (which is non-linear), systems often use a train of identical pulses. The *number* of pulses determines the state. This "accumulation mode" exploits the cumulative motion of ions and generally yields more linear updates than amplitude modulation.18

## **7\. Benchmarking and Standardization**

The field of neuromorphic computing has long suffered from a lack of standardized metrics, making it difficult to compare the "efficiency" of an HfOx-based accelerator against a CMOS-based one.

### **7.1. The NeuroBench Initiative**

The "NeuroBench" framework aims to establish a collaborative, fair, and representative suite of benchmarks.

* **Beyond MNIST:** Simple static image classification (MNIST/CIFAR) is insufficient for evaluating neuromorphic hardware, which excels in temporal processing. NeuroBench emphasizes tasks that require temporal integration, such as keyword spotting, bio-signal analysis, and event-based vision tasks.  
* **Metrics:** It standardizes the reporting of Energy-Delay Product (EDP), accuracy drop relative to floating-point baselines, and silicon area efficiency ($TOPS/mm^2$).19

### **7.2. Temporal Evaluation Gaps**

Recent critiques have highlighted that many "neuromorphic" benchmarks (like N-MNIST) do not actually require temporal processing; they can be solved by collapsing events into static frames. True benchmarking of resistive encoding requires datasets where the *timing* of spikes carries information, testing the device's ability to integrate dynamics over time (short-term plasticity).20

## **8\. Conclusion**

The transition to resistive encoding for AI weights marks a pivotal shift in the physics of computing. By harnessing the migration of ions and vacancies within nanoscopic filaments, we can create computational substrates that rival the density and efficiency of the biological brain.  
Our analysis reveals a complex landscape. The **Valence Change Mechanism (VCM)** offers mature, CMOS-compatible scaling but struggles with linearity. **Electrochemical Metallization (ECM)** provides distinct advantages in low-voltage operation but faces challenges with abruptness. The emerging **Filament Conductivity Change Mechanism (FCM)** and **Interfacial Switching** regimes in materials like $SiO\_x$ and PCMO offer the most promising path toward the high-linearity analog manifold required for accurate learning.  
However, the "analog manifold" is inherently stochastic. **Random Telegraph Noise** and **Conductance Drift** are unavoidable physical realities. The path forward lies not in eliminating these noises entirely, but in architectural and algorithmic resilience. **Differential 2T2R structures** effectively cancel common-mode noise, while **2D material barriers** suppress ion diffusion. Simultaneously, algorithms like **Multi-Teacher Robust Distillation (MTRD)** enable neural networks to learn *around* the hardware's imperfections.  
Ultimately, the successful deployment of resistive AI will rely on this tight integration of material physics, circuit topology, and robust algorithms—a "neurotrophic" synergy that mirrors the resilience of biological systems.

## **9\. References**

* 1 MDPI, "Valence Change Mechanism", 2015\.  
* 2 ResearchGate, "Conduction Mechanism of Valence Change Resistive Switching Memory", 2015\.  
* 4 PMC, "Filament Conductivity Change Mechanism (FCM)", 2025\.  
* 10 ACS Publications, "Halide-Perovskite ReRAM", 2015\.  
* 3 RSC, "Electrochemical Metallization (ECM)", 2025\.  
* 6 ResearchGate, "In situ TEM observation on interface-type resistive switching", 2016\.  
* 21 Frontiers, "Mo/TiOx RRAM hybrid synapse", 2021\.  
* 7 ResearchGate, "Tunnel Barrier Charging as Switching Mechanism", 2017\.  
* 22 ResearchGate, "Neuromorphic speech systems using ReRAM", 2015\.  
* 5 ResearchGate, "Co-existence of interfacial and filamentary switching in Ti/SiOx/Au", 2023\.  
* 11 ResearchGate, "Random Telegraph Noise in ReRAM", 2018\.  
* 12 ResearchGate, "Statistical Investigation of RTN in HfO2", 2015\.  
* 14 ResearchGate, "Noise-Induced Resistance Broadening", 2015\.  
* 15 ResearchGate, "Robust distillation for compute-in-memory", 2025\.  
* 23 IEDM, "IEDM 2024 Archive", 2024\.  
* 24 PMC, "IEEE Standard for Neuromorphic Computing", 2025\.  
* 17 ResearchGate, "64-Kb Hybrid CIM/Digital RRAM Macro", 2021\.  
* 25 ResearchGate, "Importance-Driven Weight Allocation", 2025\.  
* 26 ResearchGate, "ReRAM Based on Metal Oxides", 2012\.  
* 27 ResearchGate, "8-Mb DC-Current-Free Binary-to-8b Precision ReRAM", 2022\.  
* 28 ResearchGate, "2-Bit-per-Cell RRAM based In-Memory Computing", 2020\.  
* 9 ACS Publications, "Organic memristors", 2025\.  
* 29 ACS Publications, "Nanostructured organic materials", 2025\.  
* 30 RSC, "2D material-based memristors", 2025\.  
* 31 MDPI, "2D layered materials for neuromorphic computing", 2025\.  
* 8 Discovery, "2D material-based memristor arrays", 2025\.  
* 18 PMC, "Neuromorphic computing based on 2D materials", 2025\.  
* 20 arXiv, "Evaluation of Neuromorphic Benchmarks", 2025\.  
* 19 NIST, "NeuroBench", 2023\.  
* 13 IEEE JEDS, "Phenomenological Modeling on Nonideal Factors", 2025\.  
* 16 ResearchGate, "Sneak Path Current Modeling", 2025\.  
* 5 ResearchGate, "Detailed mechanism of FCM", 2023\.  
* 3 RSC, "Ion migration in ECM", 2025\.  
* 15 ResearchGate, "Multi-teacher robust distillation", 2025\.  
* 18 PMC, "2T2R circuit architecture details", 2025\.

#### **Works cited**

1. Conduction Mechanism of Valence Change Resistive Switching Memory: A Survey \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9292/4/3/586](https://www.mdpi.com/2079-9292/4/3/586)  
2. Conduction Mechanism of Valence Change Resistive Switching Memory: A Survey, accessed December 24, 2025, [https://www.researchgate.net/publication/281664342\_Conduction\_Mechanism\_of\_Valence\_Change\_Resistive\_Switching\_Memory\_A\_Survey](https://www.researchgate.net/publication/281664342_Conduction_Mechanism_of_Valence_Change_Resistive_Switching_Memory_A_Survey)  
3. Advances in resistive switching memory: comprehensive insights ..., accessed December 24, 2025, [https://pubs.rsc.org/en/content/articlehtml/2025/ma/d5ma00337g](https://pubs.rsc.org/en/content/articlehtml/2025/ma/d5ma00337g)  
4. Electrochemical ohmic memristors for continual learning \- PMC, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11890563/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11890563/)  
5. Co-existence of interfacial and filamentary ... \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/371663160\_Co-existence\_of\_interfacial\_and\_filamentary\_resistance\_switching\_in\_TiSiOxAu\_resistive\_memory\_devices/fulltext/648e1acbc41fb852dd0d8e45/Co-existence-of-interfacial-and-filamentary-resistance-switching-in-Ti-SiOx-Au-resistive-memory-devices.pdf](https://www.researchgate.net/publication/371663160_Co-existence_of_interfacial_and_filamentary_resistance_switching_in_TiSiOxAu_resistive_memory_devices/fulltext/648e1acbc41fb852dd0d8e45/Co-existence-of-interfacial-and-filamentary-resistance-switching-in-Ti-SiOx-Au-resistive-memory-devices.pdf)  
6. In situ TEM observation on the interface-type resistive switching by electrochemical redox reactions at a TiN/PCMO interface | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/309815039\_In\_situ\_TEM\_observation\_on\_the\_interface-type\_resistive\_switching\_by\_electrochemical\_redox\_reactions\_at\_a\_TiNPCMO\_interface](https://www.researchgate.net/publication/309815039_In_situ_TEM_observation_on_the_interface-type_resistive_switching_by_electrochemical_redox_reactions_at_a_TiNPCMO_interface)  
7. (PDF) Spectroscopic Indications of Tunnel Barrier Charging as the Switching Mechanism in Memristive Devices \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/320301299\_Spectroscopic\_Indications\_of\_Tunnel\_Barrier\_Charging\_as\_the\_Switching\_Mechanism\_in\_Memristive\_Devices](https://www.researchgate.net/publication/320301299_Spectroscopic_Indications_of_Tunnel_Barrier_Charging_as_the_Switching_Mechanism_in_Memristive_Devices)  
8. 2D Material-Based Memristor Arrays for Flexible and Thermally Stable Neuromorphic Applications. \- R Discovery, accessed December 24, 2025, [https://discovery.researcher.life/article/2d-material-based-memristor-arrays-for-flexible-and-thermally-stable-neuromorphic-applications/e0b570edd93434528d84b072392666e0](https://discovery.researcher.life/article/2d-material-based-memristor-arrays-for-flexible-and-thermally-stable-neuromorphic-applications/e0b570edd93434528d84b072392666e0)  
9. Molecular-Potential and Redox Coregulated Cathodic ..., accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acsami.3c19527](https://pubs.acs.org/doi/10.1021/acsami.3c19527)  
10. A Comprehensive Review of Electrochemical Metallization and Valence Change Mechanisms in Filamentary Resistive Switching of Halide Perovskite-Based Memory Devices \- ACS Publications, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acsami.5c09862](https://pubs.acs.org/doi/10.1021/acsami.5c09862)  
11. Random Telegraph Noise in Resistive Random Access Memories: Compact Modeling and Advanced Circuit Design | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/325184305\_Random\_Telegraph\_Noise\_in\_Resistive\_Random\_Access\_Memories\_Compact\_Modeling\_and\_Advanced\_Circuit\_Design](https://www.researchgate.net/publication/325184305_Random_Telegraph_Noise_in_Resistive_Random_Access_Memories_Compact_Modeling_and_Advanced_Circuit_Design)  
12. A Complete Statistical Investigation of RTN in HfO 2 \-Based RRAM in High Resistive State, accessed December 24, 2025, [https://www.researchgate.net/publication/280353753\_A\_Complete\_Statistical\_Investigation\_of\_RTN\_in\_HfO2-Based\_RRAM\_in\_High\_Resistive\_State](https://www.researchgate.net/publication/280353753_A_Complete_Statistical_Investigation_of_RTN_in_HfO2-Based_RRAM_in_High_Resistive_State)  
13. Phenomenological Modeling on the Nonideal Factors of Memristor Based on Single-Crystalline LiNbO₃ Thin Film \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/393690199\_Phenomenological\_Modeling\_on\_the\_Nonideal\_Factors\_of\_Memristor\_Based\_on\_Single-Crystalline\_LiNbO\_Thin\_Film](https://www.researchgate.net/publication/393690199_Phenomenological_Modeling_on_the_Nonideal_Factors_of_Memristor_Based_on_Single-Crystalline_LiNbO_Thin_Film)  
14. Noise-Induced Resistance Broadening in Resistive Switching Memory--Part I: Intrinsic Cell Behavior | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/282350343\_Noise-Induced\_Resistance\_Broadening\_in\_Resistive\_Switching\_Memory--Part\_I\_Intrinsic\_Cell\_Behavior](https://www.researchgate.net/publication/282350343_Noise-Induced_Resistance_Broadening_in_Resistive_Switching_Memory--Part_I_Intrinsic_Cell_Behavior)  
15. (PDF) Robust distillation for compute-in-memory: Realizing reliable ..., accessed December 24, 2025, [https://www.researchgate.net/publication/396852760\_Robust\_distillation\_for\_compute-in-memory\_Realizing\_reliable\_intelligence\_using\_imperfect\_memristors](https://www.researchgate.net/publication/396852760_Robust_distillation_for_compute-in-memory_Realizing_reliable_intelligence_using_imperfect_memristors)  
16. Sneak Path Current Modeling in Memristor Crossbar Arrays for Analog In-Memory Computing | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/398134392\_Sneak\_Path\_Current\_Modeling\_in\_Memristor\_Crossbar\_Arrays\_for\_Analog\_In-Memory\_Computing](https://www.researchgate.net/publication/398134392_Sneak_Path_Current_Modeling_in_Memristor_Crossbar_Arrays_for_Analog_In-Memory_Computing)  
17. A 40-nm, 64-Kb, 56.67 TOPS/W Voltage-Sensing Computing-In ..., accessed December 24, 2025, [https://www.researchgate.net/publication/353787123\_A\_40-nm\_64-Kb\_5667\_TOPSW\_Voltage-Sensing\_Computing-In-MemoryDigital\_RRAM\_Macro\_Supporting\_Iterative\_Write\_With\_Verification\_and\_Online\_Read-Disturb\_Detection](https://www.researchgate.net/publication/353787123_A_40-nm_64-Kb_5667_TOPSW_Voltage-Sensing_Computing-In-MemoryDigital_RRAM_Macro_Supporting_Iterative_Write_With_Verification_and_Online_Read-Disturb_Detection)  
18. Two-dimensional materials based two-transistor-two-resistor ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12064777/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12064777/)  
19. NeuroBench: advancing neuromorphic computing through collaborative, fair and representative benchmarking | NIST, accessed December 24, 2025, [https://www.nist.gov/publications/neurobench-advancing-neuromorphic-computing-through-collaborative-fair-and](https://www.nist.gov/publications/neurobench-advancing-neuromorphic-computing-through-collaborative-fair-and)  
20. Spiking Neural Networks for Temporal Processing: Status Quo and Future Prospects \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2502.09449v1](https://arxiv.org/html/2502.09449v1)  
21. Neural Network Training Acceleration With RRAM-Based Hybrid Synapses \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2021.690418/pdf](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2021.690418/pdf)  
22. Neuromorphic speech systems using advanced ReRAM-based synapse | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/271426043\_Neuromorphic\_speech\_systems\_using\_advanced\_ReRAM-based\_synapse](https://www.researchgate.net/publication/271426043_Neuromorphic_speech_systems_using_advanced_ReRAM-based_synapse)  
23. IEDM+2024+Archive.pdf \- Squarespace, accessed December 24, 2025, [https://static1.squarespace.com/static/67a3eee4385dfb3390804f02/t/67d4b9026674b64dfd6d035b/1741994245226/IEDM+2024+Archive.pdf](https://static1.squarespace.com/static/67a3eee4385dfb3390804f02/t/67d4b9026674b64dfd6d035b/1741994245226/IEDM+2024+Archive.pdf)  
24. Roadmap on Emerging Hardware and Technology for Machine ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11411818/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11411818/)  
25. IDWA: A Importance-Driven Weight Allocation Algorithm for Low Write–Verify Ratio RRAM-Based In-Memory Computing | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/392881396\_IDWA\_A\_Importance-Driven\_Weight\_Allocation\_Algorithm\_for\_Low\_Write-Verify\_Ratio\_RRAM-Based\_In-Memory\_Computing](https://www.researchgate.net/publication/392881396_IDWA_A_Importance-Driven_Weight_Allocation_Algorithm_for_Low_Write-Verify_Ratio_RRAM-Based_In-Memory_Computing)  
26. Resistive Random Access Memory (ReRAM) Based on Metal Oxides \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/224184428\_Resistive\_Random\_Access\_Memory\_ReRAM\_Based\_on\_Metal\_Oxides](https://www.researchgate.net/publication/224184428_Resistive_Random_Access_Memory_ReRAM_Based_on_Metal_Oxides)  
27. An 8-Mb DC-Current-Free Binary-to-8b Precision ReRAM Nonvolatile Computing-in-Memory Macro using Time-Space-Readout with 1286.4-21.6TOPS/W for Edge-AI Devices \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/359326880\_An\_8-Mb\_DC-Current-Free\_Binary-to-8b\_Precision\_ReRAM\_Nonvolatile\_Computing-in-Memory\_Macro\_using\_Time-Space-Readout\_with\_12864-216TOPSW\_for\_Edge-AI\_Devices](https://www.researchgate.net/publication/359326880_An_8-Mb_DC-Current-Free_Binary-to-8b_Precision_ReRAM_Nonvolatile_Computing-in-Memory_Macro_using_Time-Space-Readout_with_12864-216TOPSW_for_Edge-AI_Devices)  
28. 2-Bit-per-Cell RRAM based In-Memory Computing for Area-/Energy-Efficient Deep Learning | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/343115751\_2-Bit-per-Cell\_RRAM\_based\_In-Memory\_Computing\_for\_Area-Energy-Efficient\_Deep\_Learning](https://www.researchgate.net/publication/343115751_2-Bit-per-Cell_RRAM_based_In-Memory_Computing_for_Area-Energy-Efficient_Deep_Learning)  
29. Organic Nanomaterials in Memristor-Based Devices for Information Processing and Artificial Intelligence Applications: A Review | ACS Applied Nano Materials, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acsanm.5c02022](https://pubs.acs.org/doi/10.1021/acsanm.5c02022)  
30. Fast prototyping of memristors for ReRAMs and neuromorphic computing \- RSC Publishing, accessed December 24, 2025, [https://pubs.rsc.org/en/content/articlehtml/2025/nr/d5nr02690c](https://pubs.rsc.org/en/content/articlehtml/2025/nr/d5nr02690c)  
31. Recent Advancements in 2D Material-Based Memristor Technology Toward Neuromorphic Computing \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2072-666X/15/12/1451](https://www.mdpi.com/2072-666X/15/12/1451)