# **Phase-Change Dynamics and Conductance Drift: Physics, Modeling, and Mitigation in Analog Neuromorphic Computing**

## **1\. Introduction: The Analog Computing Paradigm and the Stability Challenge**

The trajectory of modern computing is increasingly defined by the divergence between the capabilities of traditional von Neumann architectures and the demands of data-intensive Artificial Intelligence (AI) workloads. The physical separation of processing units and memory creates a bandwidth bottleneck and significant energy latency, prompting a shift toward In-Memory Computing (IMC) and neuromorphic architectures. Within this emerging paradigm, Phase-Change Memory (PCM) has established itself as a frontrunner, offering the density, scalability, and non-volatility required to implement synaptic weights in hardware Neural Networks (NNs).1  
However, the transition from binary storage—where a cell is simply "0" or "1"—to analog computing requires the reliable storage of a continuum of conductance states. These states represent the synaptic weights essential for Deep Neural Networks (DNNs) and Spiking Neural Networks (SNNs). The efficacy of analog PCM is predicated on the ability to program these weights with high linearity and, critically, to maintain their values over time against thermodynamic pressures.3  
This report provides an exhaustive analysis of the physical phenomena that threaten this stability: **Phase-Change Dynamics** and **Conductance Drift**. While the crystalline phase of chalcogenide glass (the active material in PCM) is thermodynamically stable, the amorphous phase—essential for creating high-resistance states—is metastable. It undergoes spontaneous structural relaxation, leading to a temporal evolution of electrical resistance known as drift. This phenomenon fundamentally compromises the precision of analog weights, leading to inference accuracy degradation over time.1  
We explore the atomic-level origins of this relaxation, extending from bulk effects to interface-dominated Schottky barrier dynamics. We analyze the kinetics of crystallization through the lens of Johnson-Mehl-Avrami-Kolmogorov (JMAK) theory to understand how programming pulses can be shaped to achieve linear analog updates. Finally, we evaluate advanced mitigation strategies, including Projected PCM (P-PCM) device architectures, material engineering of growth-dominated alloys like AlSb, and algorithmic compensation schemes that allow neural networks to function robustly despite the stochastic nature of the underlying hardware.6

## ---

**2\. Thermodynamics and Physics of the Amorphous State**

The fundamental challenge in analog PCM lies in the nature of the amorphous state. Unlike the crystalline phase, which possesses long-range atomic order and a defined minimum energy configuration, the amorphous phase is a disordered, metastable glass. To understand conductance drift, one must first understand the thermodynamic forces driving the material's evolution.

### **2.1. The Melt-Quench Process and Metastability**

The amorphous state in PCM devices is typically generated via a melt-quench process. A high-current electrical pulse (RESET pulse) heats the chalcogenide material—typically $Ge\_2Sb\_2Te\_5$ (GST)—above its melting temperature ($T\_m \\approx 600^\\circ C$). This is followed by a rapid thermal quench, often exceeding cooling rates of $10^9$ K/s, which freezes the atomic structure in a disordered configuration before it can crystallize.2  
This process traps the material in a local energy minimum, but not the global minimum. The resulting amorphous structure is highly stressed and contains a significant density of defects, distorted bond angles, and "wrong" bonds (e.g., Ge-Ge or Sb-Sb homopolar bonds in a lattice that prefers heteropolar bonding). The material is effectively a "frozen liquid" that retains a memory of its high-temperature, high-entropy state.2

### **2.2. Structural Relaxation: The Origin of Drift**

Following the quench, the amorphous material undergoes **structural relaxation** (SR), a process of physical aging. Driven by the thermodynamic imperative to minimize Gibbs free energy, the atoms within the amorphous matrix spontaneously rearrange themselves. This is not a phase transition to the crystalline state (which would involve nucleation and growth), but rather a subtle densification and ordering within the amorphous phase itself.10  
The relaxation process is characterized by the annihilation of defects and the relief of internal Peierls distortions. As the material relaxes toward an "ideal glass" state, several electronic changes occur:

1. **Bandgap Widening:** The reduction in disorder reduces the tails of localized states at the band edges, effectively widening the mobility gap.  
2. **Defect Annihilation:** The density of trap states deep within the bandgap decreases. Since conduction in amorphous chalcogenides is typically trap-limited (governed by Poole-Frenkel emission or hopping conduction), a reduction in trap density significantly impedes charge transport.  
3. **Mobility Reduction:** The structural rearrangement increases the average activation energy required for carriers to hop between localized sites, leading to a decrease in carrier mobility.1

#### **2.2.1. Phenomenological Models: Gibbs vs. Collective Relaxation**

Recent research has sought to model this relaxation mathematically. Two primary phenomenological models are often debated:

* **The Gibbs Relaxation Model:** This model posits that the activation energy for relaxation is distributed over a spectrum. As time progresses, processes with lower activation energies are exhausted, and only those with higher barriers remain, slowing the rate of relaxation logarithmically.  
* **The Collective Relaxation Model:** This approach views the relaxation as a cooperative phenomenon involving clusters of atoms. It suggests that the relaxation of one region alters the stress field of its neighbors, creating a cascading effect. Experimental data on drift over 9 orders of magnitude in time (from 100 K to 300 K) has been used to appraise these models, confirming that relaxation is a fundamental property of the melt-quenched state that begins almost immediately after programming.2

### **2.3. The Interface: Schottky Barrier Height (SBH) Drift**

While classical theory attributes drift primarily to bulk structural relaxation, recent studies on scaled PCM devices (sub-100 nm) have identified a critical interface mechanism. In these nanoscale regimes, the electrical resistance is often dominated not by the bulk chalcogenide, but by the contact resistance at the interface between the PCM and the electrode (commonly Titanium Nitride, TiN, or Tungsten).  
Experimental evidence indicates that the **Schottky barrier height (SBH)** for hole injection at this interface is not static. As the amorphous material at the interface undergoes structural relaxation, the Fermi level pinning shifts, or the density of interface trap states changes. This results in a time-dependent increase in the barrier height. Consequently, the resistance drift observed in highly scaled devices is effectively a "Schottky barrier drift" rather than purely a bulk resistivity change.12 This insight is pivotal for device engineering, suggesting that interface passivation and contact material selection are as critical as bulk material optimization.

## ---

**3\. Conductance Drift: Mathematical Modeling and Variability**

The macroscopic manifestation of structural relaxation is a monotonic increase in electrical resistance over time. For analog computing, where specific resistance values map to synaptic weights, quantifying and predicting this drift is essential.

### **3.1. The Power Law of Drift**

The temporal evolution of resistance $R(t)$ in the amorphous state is empirically described by a power law, which has been validated across decades of time and varying environmental conditions 1:

$$R(t) \= R\_0 \\left( \\frac{t}{t\_0} \\right)^\\nu$$  
Where:

* $R(t)$ is the resistance at time $t$.  
* $t\_0$ is an arbitrary reference time (usually the time of the first read after programming, e.g., 1 second).  
* $R\_0$ is the resistance measured at $t\_0$.  
* $\\nu$ (nu) is the **drift coefficient**, a dimensionless parameter representing the slope of the resistance evolution on a log-log scale.

### **3.2. Variability of the Drift Coefficient ($\\nu$)**

The drift coefficient $\\nu$ is not a universal constant; it is a variable parameter influenced by the physical state of the cell, the device geometry, and the ambient temperature.

* **State Dependence:** The magnitude of drift is strongly correlated with the amount of amorphous volume.  
  * **Crystalline State:** $\\nu \\approx 0$ (Negligible drift).  
  * **Fully Amorphous State:** $\\nu \\approx 0.1$ (Significant drift).  
  * **Partial (Analog) States:** For intermediate states used in neuromorphic weights, $\\nu$ scales with the resistance. Higher resistance states (more amorphous) drift faster than lower resistance states.8 This creates a non-linear distortion of the synaptic weight vector over time.  
* **Geometric Dependence:** Research on nanowire PCM devices indicates that drift is influenced by the surface-to-volume ratio. Thinner nanowires (e.g., 45 nm) have shown lower drift coefficients ($\\nu \\approx 0.002$) compared to thicker wires or thin films ($\\nu \\approx 0.01 \- 0.1$). This suggests that surface effects or mechanical stress relaxation at boundaries play a role in the kinetics of the glassy state.13  
* **Temperature Dependence:** Drift is thermally activated. Higher temperatures accelerate the structural relaxation process, effectively increasing the rate of resistance rise. This poses a significant challenge for automotive or industrial AI applications where operating temperatures can fluctuate significantly.14

### **3.3. Stochasticity and Noise**

Beyond the deterministic mean drift, the relaxation process is inherently stochastic. An array of 10,000 PCM devices programmed to the same target conductance will not drift identically. Instead, they will exhibit a distribution of drift coefficients, typically modeled as a Gaussian or log-normal distribution.  
The spread of the conductance distribution $\\sigma\_G(t)$ broadens over time according to:

$$\\sigma\_G(t) \\approx \\sigma\_{G0} \+ \\sigma\_\\nu \\ln\\left(\\frac{t}{t\_0}\\right)$$  
This equation implies that the signal-to-noise ratio (SNR) of the stored weights degrades logarithmically with time. Additionally, the amorphous phase exhibits $1/f$ noise and Random Telegraph Noise (RTN) due to carrier trapping/detrapping in the bandgap states. This read noise is superimposed on the drift, making high-precision analog readout increasingly difficult as the device ages.14

### **3.4. Drift Re-initialization**

A critical phenomenon for training algorithms is **drift re-initialization**. When a PCM cell receives a programming pulse (even a partial SET pulse to adjust the weight slightly), the structural relaxation history is erased. The thermal energy of the pulse effectively "resets the clock," and the drift process restarts from $t=0$.16  
In a neural network being trained in-situ, weights are updated at different frequencies. "Hot" weights (frequently updated) will have their drift constantly reset, while "cold" weights (rarely updated) will accumulate significant drift. This creates a temporal heterogeneity across the array, where the effective "age" of synapses varies, complicating global compensation schemes.17

## ---

**4\. Phase-Change Kinetics: Nucleation and Growth**

While drift dictates the retention behavior, the programming of analog weights is governed by the kinetics of the phase transition itself. Achieving a continuum of conductance levels requires precise control over the partial crystallization (SET) or partial amorphization (RESET) of the cell. This dynamics is described by the Johnson-Mehl-Avrami-Kolmogorov (JMAK) theory.

### **4.1. JMAK Theory and Crystallization Dynamics**

The transformation from the metastable amorphous phase to the stable crystalline phase is driven by the reduction in Gibbs free energy. Under isothermal conditions, the crystallized volume fraction $X(t)$ as a function of time $t$ is given by the Avrami equation 18:

$$X(t) \= 1 \- \\exp(-Kt^n)$$  
Where:

* $K$ is a rate constant dependent on the nucleation rate ($I$) and the crystal growth velocity ($u$). It is strongly temperature-dependent, typically following an Arrhenius behavior.  
* $n$ is the **Avrami exponent**, a critical parameter that reflects the dimensionality of growth and the nature of nucleation.

#### **4.1.1. Interpreting the Avrami Exponent ($n$)**

The value of $n$ provides a window into the microscopic physics of the switching mechanism:

* **$n \\approx 4$:** Implies homogeneous nucleation (constant nucleation rate) and three-dimensional growth. This is characteristic of bulk crystallization where nuclei form randomly throughout the volume.  
* **$n \\approx 3$:** Implies site-saturated nucleation (all nuclei form instantaneously at $t=0$) followed by 3D growth.  
* **$n \< 2$:** Often observed in PCM thin films and nanostructures. This indicates reduced dimensionality (e.g., 1D or 2D growth of dendrites) or diffusion-controlled growth with a decreasing nucleation rate.20

For standard GST devices, crystallization is often **nucleation-dominated**, meaning the transformation is driven by the spontaneous formation of many crystal nuclei which then grow and coalesce. This can lead to abrupt switching behavior (a percolation threshold), which is non-ideal for analog applications.

### **4.2. Growth-Dominated vs. Nucleation-Dominated Materials**

To achieve linear analog programming, materials engineering often focuses on shifting the kinetics from nucleation-dominated to **growth-dominated** behavior.

* **Nucleation-Dominated (e.g., pure GST):** Switching is stochastic and abrupt. It is difficult to control the exact fraction of crystallinity because once nucleation starts, it proceeds rapidly throughout the volume.6  
* **Growth-Dominated (e.g., AgInSbTe or doped GST):** Crystallization proceeds primarily by the growth of the crystal front from the existing crystal/amorphous interface. The speed of the front velocity is controlled by the temperature (pulse amplitude). This allows for "domain wall displacement," where the amorphous plug can be shrunk continuously and linearly, enabling high-precision analog states.21

### **4.3. Pulse Shaping for Linear Kinetics**

Understanding JMAK kinetics allows for the design of sophisticated programming pulses that linearize the weight update trajectory.

* **Box Pulses:** A standard rectangular pulse may cause uncontrolled crystallization if the temperature is too high.  
* **Ramp-Down Pulses:** A pulse that slowly ramps down in amplitude allows the material to cool slowly, promoting crystallization. By controlling the slope, one can control the total time the material spends in the crystallization temperature window.22  
* **Multi-Pulse Trains:** Applying a sequence of short pulses can incrementally advance the crystallization front. Optical monitoring studies utilizing JMAK modeling have shown that real-time feedback loops can program arbitrary partial crystallization states by exploiting the time-dependence of the phase change.23

## ---

**5\. Device and Material Engineering for Stability**

Addressing the root causes of drift and non-linearity requires innovation at the device and material levels. Two major avenues of research are Projected PCM architectures and compositional tuning.

### **5.1. Projected PCM (P-PCM): Decoupling Storage from Readout**

Projected PCM is arguably the most significant hardware innovation for mitigating drift in analog computing. The central concept is to decouple the physical mechanism of resistance storage (the phase state) from the mechanism of information retrieval (electrical readout).2

#### **5.1.1. Device Architecture and Mechanism**

In a conventional PCM cell, the read current must traverse the amorphous volume, which is the source of high resistance and drift. In a Projected PCM cell, a non-insulating **projection segment** is fabricated in parallel with the phase-change material.

* **Projection Material:** The projection segment is composed of a material with a resistivity that lies *between* that of the crystalline and amorphous states of the PCM. Common materials include metal nitrides ($Ti\_xN\_y$) or amorphous carbon.24  
* **Read Operation:**  
  * **SET State (Crystalline):** Since the crystalline PCM is highly conductive ($R\_{cryst} \< R\_{proj}$), the read current flows primarily through the PCM segment.  
  * **RESET State (Amorphous):** Since the amorphous PCM is highly resistive ($R\_{amorph} \> R\_{proj}$), the read current bypasses the amorphous volume and flows through the parallel projection liner.

#### **5.1.2. Drift Suppression**

The net resistance of the device in the RESET state is dominated by the projection liner. Since the projection material (e.g., TiN or Carbon) does not undergo structural relaxation, it does not drift. The amorphous PCM volume still exists and defines the state (by blocking the current path through the PCM), but its drifting resistivity does not contribute to the measured resistance.

* **Results:** P-PCM devices demonstrate a reduction in the apparent drift coefficient by orders of magnitude. Furthermore, they significantly attenuate $1/f$ noise, which typically originates from carrier fluctuations in the amorphous chalcogenide.15

### **5.2. Material Composition and Heterostructures**

* **AlSb Alloys:** Research into Al-Sb binary alloys has revealed a stable amorphous phase with a high crystallization temperature. These materials exhibit "nanoscale compositional heterogeneity" in the amorphous state, which suppresses long-range structural relaxation. Consequently, AlSb-based PCM cells have demonstrated drift coefficients as low as $\\nu \\approx 0.06$, significantly lower than standard GST, while maintaining high resistance contrast.6  
* **Doping:** The addition of dopants like Nitrogen, Carbon, or Silicon into the GST matrix can increase the activation energy for structural relaxation, effectively slowing down the aging process. However, this often increases the voltage required for switching.  
* **Confined Geometries:** As noted in the physics section, confining the PCM in nanowires or pore structures alters the crystallization kinetics and stress relaxation mechanisms. While surface effects become prominent, the mechanical constraint can inhibit the volume expansion associated with amorphization, potentially reducing drift.13

## ---

**6\. Impact on Neural Network Performance**

The ultimate test of these physical and engineering principles is the performance of the PCM arrays in neuromorphic workloads. Conductance drift translates directly into weight decay, which impacts the accuracy of Deep Neural Networks (DNNs) and Spiking Neural Networks (SNNs).

### **6.1. Inference Accuracy Degradation**

In an inference scenario, weights are trained and programmed once. As time passes, drift causes the conductance values to decrease.

* **Accuracy Drop:** Studies employing models like ResNet or Multi-Layer Perceptrons (MLP) on datasets such as CIFAR-10 have quantified this impact. While a uniform drift across all weights might only scale the output activations (preserving classification accuracy due to the scale-invariance of Softmax), the stochastic and state-dependent nature of drift causes weights to evolve non-uniformly.  
* **Quantified Losses:** Uncompensated drift can lead to significant accuracy losses. Research indicates that without compensation, accuracy on complex tasks can drop by \>10% over time. However, with drift variability (device-to-device variation), even minor mean drift can degrade accuracy by \~1-2% in robust networks, and much more in sensitive ones.8 In SNNs, unmitigated drift has been shown to degrade performance by \~12.4% over time.26

### **6.2. In-Situ Training Challenges**

Training on-chip is particularly sensitive to the **drift re-initialization** phenomenon.

* **The "Stale Weight" Problem:** In a training run, some weights are updated frequently (resetting their drift clock), while others may not be updated for many epochs (accumulating drift). This creates a temporal noise pattern where the effective weight values read by the network differ from the intended values based on their update history.  
* **Convergence Issues:** This history-dependent noise can prevent the gradient descent algorithm from converging to a global minimum, or it may force the network into a suboptimal solution that is robust to noise but less accurate.16

### **6.3. SNN Specifics: Timing and Sparsity**

Spiking Neural Networks rely on the precise timing of spikes. Conductance drift changes the integration time of the post-synaptic neuron (since $I \= G \\cdot V$), potentially delaying firing times.

* **Spontaneous Sparse Learning (SSL):** Interestingly, some research suggests utilizing drift as a feature rather than a bug. In SSL, drift is treated as a natural "forgetting" mechanism. Synapses that are not reinforced (updated) drift toward high resistance (zero weight) and are effectively pruned. This mimics biological synaptic pruning and can lead to highly efficient, sparse networks.27

## ---

**7\. Comparison with Competing Technologies: PCM vs. RRAM**

To contextualize the position of PCM in the analog AI landscape, it is necessary to compare its dynamics with Resistive Random Access Memory (RRAM), the other primary candidate for non-volatile analog weights.

### **7.1. Mechanisms of Instability**

* **PCM:** Instability arises from **Structural Relaxation** (thermodynamic aging of the glass) and **Schottky Barrier Drift**. It follows a power law $R \\propto t^\\nu$. The mechanism is a bulk/interface material property.1  
* **RRAM:** Instability arises from **Conductance Relaxation**, driven by the diffusion and redistribution of oxygen vacancies (or metal ions) within the conductive filament. This is often caused by the concentration gradient of defects. RRAM relaxation also typically follows a logarithmic or power law behavior but is driven by ionic kinetics rather than glassy thermodynamics.28

### **7.2. Performance Trade-offs**

The following table summarizes the key differences relevant to analog computing:

| Feature | Phase-Change Memory (PCM) | Resistive RAM (RRAM) |
| :---- | :---- | :---- |
| **Switching Mechanism** | Phase Transition (Amorphous $\\leftrightarrow$ Crystalline) | Filament Formation/Rupture (Ion Migration) |
| **Drift Origin** | Structural Relaxation (Glassy state) | Ion Diffusion / Charge Trapping |
| **Drift Coefficient ($\\nu$)** | Typically 0.01 – 0.1 (High) | Variable, often lower but stochastic |
| **Drift Behavior** | Monotonic resistance increase | Can be bidirectional (relaxation/recovery) |
| **Linearity** | High (especially with P-PCM and pulse shaping) | Often abrupt (set/reset), harder to control linearly |
| **Write Speed** | Limited by crystallization time (\~50-100 ns) | Very fast (\~ns range possible) 28 |
| **Endurance** | High ($10^9$ cycles) | Moderate ($10^6$ \- $10^9$ cycles) |
| **Retention** | Excellent (years), but drifts | Good, but filament dissolution can occur |

PCM is generally favored for applications requiring high precision and linearity (like DNN inference accelerators), especially when drift compensation is applied. RRAM is often favored for low-power, high-speed edge training where the write latency of PCM is a bottleneck.30

## ---

**8\. Mitigation and Compensation Strategies**

Given that drift is intrinsic to the physics of amorphous materials, a multi-layered approach involving circuit and algorithmic compensation is required to achieve commercial-grade reliability.

### **8.1. Circuit-Level Compensation**

* **Reference Cell Conductance Tracking (RCCT):** A subset of cells in the array is designated as reference cells. They are programmed to known states and allowed to drift. By reading these cells, the system calculates a global drift correction factor (bias) that is applied to the entire array during readout. This effectively "normalizes" the drift.14  
* **Differential Pairs (Conductance Ratio):** Synaptic weights are stored not as a single conductance $G$, but as the difference between two devices: $W \= G^+ \- G^-$. Since drift affects both devices similarly (common-mode noise), the difference remains relatively stable. This technique has been shown to improve MAC (Multiply-Accumulate) accuracy to \~95% even in the presence of drift.3  
* **Non-Linear Current Scaling:** Advanced readout circuits employ non-linear scaling of the read current to counteract the state-dependence of drift. By attenuating the current based on the resistance level, the circuit can flatten the drift characteristics, reducing the drift coefficient by up to \~57% and variability by \~5%.26

### **8.2. Algorithmic Resilience**

* **Noise-Aware Training:** Neural networks are trained with "injected noise" that mimics the statistical properties of drift and variability. This forces the optimization algorithm to find flat minima in the loss landscape, ensuring that the network remains accurate even when weights drift.4  
* **Global Bias Correction:** A simple scalar correction term is added to the batch normalization layers of the network. This scalar is updated periodically based on the average drift of the array, realigning the activation distributions without reprogramming the weights.14  
* **Multi-PCM Synapses:** To combat stochasticity, a single synapse can be represented by $N$ parallel PCM devices. The law of large numbers reduces the relative variability by $1/\\sqrt{N}$, improving linearity and retention at the cost of area density.5

## ---

**9\. Conclusion**

The integration of Phase-Change Memory into analog neuromorphic systems represents a convergence of condensed matter physics and advanced computing. The challenges posed by **Phase-Change Dynamics** and **Conductance Drift** are non-trivial; they are rooted in the fundamental thermodynamics of the glassy state and the kinetics of nucleation. The structural relaxation of the amorphous phase, coupled with Schottky barrier instabilities at the interface, creates a time-varying substrate for computation that defies the static abstractions of digital logic.  
However, the field has moved beyond viewing these phenomena merely as obstacles. Through rigorous mathematical modeling—applying JMAK theory to linearize programming and power laws to predict retention—and innovative engineering, the stability of analog weights has been dramatically enhanced. **Projected PCM** stands out as a paradigm-shifting device architecture that effectively cloaks the instability of the storage medium from the readout circuitry. Concurrently, **growth-dominated materials** like AlSb and **algorithmic innovations** like noise-aware training and spontaneous sparse learning are turning physical constraints into manageable, and sometimes advantageous, features.  
The future of analog PCM lies in the holistic integration of these layers: materials designed for thermodynamic stability, devices designed for electrical isolation of drift, and algorithms designed for stochastic resilience. As these domains coalesce, PCM is poised to enable a new generation of energy-efficient, brain-inspired computing hardware that rivals the connectivity and efficiency of biological synapses.

#### **Works cited**

1. Programming Schemes To Reduce Conductance Drift In Analog PCM Arrays, accessed December 24, 2025, [https://eureka.patsnap.com/report-programming-schemes-to-reduce-conductance-drift-in-analog-pcm-arrays](https://eureka.patsnap.com/report-programming-schemes-to-reduce-conductance-drift-in-analog-pcm-arrays)  
2. Abstract \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2401.09462v1](https://arxiv.org/html/2401.09462v1)  
3. Combined HW/SW Drift and Variability Mitigation for PCM-Based Analog In-Memory Computing for Neural Network Applications | IEEE Journals & Magazine, accessed December 24, 2025, [https://ieeexplore.ieee.org/document/10035378/](https://ieeexplore.ieee.org/document/10035378/)  
4. Combined HW/SW Drift and Variability Mitigation for PCM-Based Analog In-Memory Computing for Neural Network Applications \- Unibo, accessed December 24, 2025, [https://cris.unibo.it/bitstream/11585/913711/5/Combined\_HW\_SW\_Drift\_and\_Variability\_Mitigation\_for\_PCM-based\_Analog\_In-memory\_Computing\_for\_Neural\_Network\_Applications\_IRIS.pdf](https://cris.unibo.it/bitstream/11585/913711/5/Combined_HW_SW_Drift_and_Variability_Mitigation_for_PCM-based_Analog_In-memory_Computing_for_Neural_Network_Applications_IRIS.pdf)  
5. Impact of conductance drift on multi-PCM synaptic architectures \- Semantic Scholar, accessed December 24, 2025, [https://www.semanticscholar.org/paper/Impact-of-conductance-drift-on-multi-PCM-synaptic-Boybat-Nandakumar/f17e3c0f610bb30a3ed496220ef1cf221522a5b4](https://www.semanticscholar.org/paper/Impact-of-conductance-drift-on-multi-PCM-synaptic-Boybat-Nandakumar/f17e3c0f610bb30a3ed496220ef1cf221522a5b4)  
6. Benchmarking PCM technologies a Resistance drift coefficient vs. reset... \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/figure/Benchmarking-PCM-technologies-a-Resistance-drift-coefficient-vs-reset-energy-and-b\_fig4\_377587206](https://www.researchgate.net/figure/Benchmarking-PCM-technologies-a-Resistance-drift-coefficient-vs-reset-energy-and-b_fig4_377587206)  
7. Projected phase-change memory devices \- PubMed, accessed December 24, 2025, [https://pubmed.ncbi.nlm.nih.gov/26333363/](https://pubmed.ncbi.nlm.nih.gov/26333363/)  
8. The impact of Resistance Drift of PCM on Conventional Neural Networks \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/ielaam/55/8771270/8753712-aam.pdf](https://ieeexplore.ieee.org/ielaam/55/8771270/8753712-aam.pdf)  
9. Unsupervised Learning by Spike Timing Dependent Plasticity in Phase Change Memory (PCM) Synapses \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2016.00056/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2016.00056/full)  
10. Amorphous phase-change memory alloy with no resistance drift \- PubMed, accessed December 24, 2025, [https://pubmed.ncbi.nlm.nih.gov/41034616/](https://pubmed.ncbi.nlm.nih.gov/41034616/)  
11. Statistics of Resistance Drift Due to Structural Relaxation in Phase-Change Memory Arrays | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/224169328\_Statistics\_of\_Resistance\_Drift\_Due\_to\_Structural\_Relaxation\_in\_Phase-Change\_Memory\_Arrays](https://www.researchgate.net/publication/224169328_Statistics_of_Resistance_Drift_Due_to_Structural_Relaxation_in_Phase-Change_Memory_Arrays)  
12. Drift of Schottky Barrier Height in Phase Change Materials | ACS Nano, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acsnano.3c11019](https://pubs.acs.org/doi/10.1021/acsnano.3c11019)  
13. Extremely low drift of resistance and threshold voltage in amorphous phase change nanowire devices \- UCSB Materials, accessed December 24, 2025, [https://labs.materials.ucsb.edu/gianola/dan/sites/labs.materials.ucsb.edu.gianola.dan/files/pubs/mitra\_pcm\_drift\_published\_apl\_2010.pdf](https://labs.materials.ucsb.edu/gianola/dan/sites/labs.materials.ucsb.edu.gianola.dan/files/pubs/mitra_pcm_drift_published_apl_2010.pdf)  
14. A Readout Scheme for PCM-Based Analog In-Memory Computing With Drift Compensation Through Reference Conductance Tracking \- Unibo, accessed December 24, 2025, [https://cris.unibo.it/bitstream/11585/976134/3/A\_Readout\_Scheme\_for\_PCM-Based\_Analog\_In-Memory\_Computing\_With\_Drift\_Compensation\_Through\_Reference\_Conductance\_Tracking.pdf](https://cris.unibo.it/bitstream/11585/976134/3/A_Readout_Scheme_for_PCM-Based_Analog_In-Memory_Computing_With_Drift_Compensation_Through_Reference_Conductance_Tracking.pdf)  
15. State dependence and temporal evolution of resistance in projected phase change memory, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC7237438/](https://pmc.ncbi.nlm.nih.gov/articles/PMC7237438/)  
16. Impact of conductance drift on multi-PCM synaptic architectures for ..., accessed December 24, 2025, [https://research.ibm.com/publications/impact-of-conductance-drift-on-multi-pcm-synaptic-architectures](https://research.ibm.com/publications/impact-of-conductance-drift-on-multi-pcm-synaptic-architectures)  
17. Impact of conductance drift on multi-PCM synaptic architectures \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/330581957\_Impact\_of\_conductance\_drift\_on\_multi-PCM\_synaptic\_architectures](https://www.researchgate.net/publication/330581957_Impact_of_conductance_drift_on_multi-PCM_synaptic_architectures)  
18. Avrami equation \- Wikipedia, accessed December 24, 2025, [https://en.wikipedia.org/wiki/Avrami\_equation](https://en.wikipedia.org/wiki/Avrami_equation)  
19. A Note on the Johnson–Mehl–Avrami–Kolmogorov Kinetic Model: An Attempt Aiming to Introduce Time Non-Locality \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2673-4117/6/2/24](https://www.mdpi.com/2673-4117/6/2/24)  
20. Non-Isothermal Analysis of the Crystallization Kinetics of Amorphous Mg72Zn27Pt1 and Mg72Zn27Ag1 Alloys \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/1996-1944/17/2/408](https://www.mdpi.com/1996-1944/17/2/408)  
21. Models for phase-change of Ge2Sb2Te5 in optical and electrical memory devices, accessed December 24, 2025, [https://www.researchgate.net/publication/234949993\_Models\_for\_phase-change\_of\_Ge2Sb2Te5\_in\_optical\_and\_electrical\_memory\_devices](https://www.researchgate.net/publication/234949993_Models_for_phase-change_of_Ge2Sb2Te5_in_optical_and_electrical_memory_devices)  
22. Characterization and Programming Algorithm of Phase Change Memory Cells for Analog In-Memory Computing \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC8037667/](https://pmc.ncbi.nlm.nih.gov/articles/PMC8037667/)  
23. arXiv:2306.17631v1 \[physics.app-ph\] 30 Jun 2023, accessed December 24, 2025, [https://arxiv.org/pdf/2306.17631](https://arxiv.org/pdf/2306.17631)  
24. US11665984B2 \- Projected memory device with carbon-based projection component \- Google Patents, accessed December 24, 2025, [https://patents.google.com/patent/US11665984B2/en](https://patents.google.com/patent/US11665984B2/en)  
25. Projected memory device with reduced minimum conductance state \- Google Patents, accessed December 24, 2025, [https://patents.google.com/patent/US11665985/en](https://patents.google.com/patent/US11665985/en)  
26. Phase Change Memory Drift Compensation in Spiking Neural Networks Using a Non-Linear Current Scaling Strategy \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9268/14/4/50](https://www.mdpi.com/2079-9268/14/4/50)  
27. Spontaneous sparse learning for PCM-based memristor neural networks \- PMC, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC7803975/](https://pmc.ncbi.nlm.nih.gov/articles/PMC7803975/)  
28. RRAM vs PCM: Which Delivers Higher Write Speed? \- Patsnap Eureka, accessed December 24, 2025, [https://eureka.patsnap.com/report-rram-vs-pcm-which-delivers-higher-write-speed](https://eureka.patsnap.com/report-rram-vs-pcm-which-delivers-higher-write-speed)  
29. Efficient Calibration for RRAM-based In-Memory Computing using DoRA \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2504.03763](https://arxiv.org/pdf/2504.03763)  
30. Driving for More Moore on Computing Devices with Advanced Non-Volatile Memory Technology \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9292/14/17/3456](https://www.mdpi.com/2079-9292/14/17/3456)  
31. Emerging Storage Technologies: MRAM, RRAM, and PCRAM \- Utmel, accessed December 24, 2025, [https://www.utmel.com/blog/categories/memory%20chip/emerging-storage-technologies-mram-rram-and-pcram](https://www.utmel.com/blog/categories/memory%20chip/emerging-storage-technologies-mram-rram-and-pcram)  
32. Differential Phase Change Memory (PCM) Cell for Drift-Compensated In-Memory Computing \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/385258473\_Differential\_Phase\_Change\_Memory\_PCM\_Cell\_for\_Drift-Compensated\_In-Memory\_Computing](https://www.researchgate.net/publication/385258473_Differential_Phase_Change_Memory_PCM_Cell_for_Drift-Compensated_In-Memory_Computing)  
33. Reducing the Impact of Phase-Change Memory Conductance Drift on the Inference of large-scale Hardware Neural Networks for IEDM 2019 \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/publications/reducing-the-impact-of-phase-change-memory-conductance-drift-on-the-inference-of-large-scale-hardware-neural-networks](https://research.ibm.com/publications/reducing-the-impact-of-phase-change-memory-conductance-drift-on-the-inference-of-large-scale-hardware-neural-networks)