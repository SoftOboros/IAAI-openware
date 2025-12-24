# **Phase-Change Memory (PCM) for Analog AI Weights: Physics, Conductance Drift, and Neurotrophic Dynamics**

## **Executive Summary**

The impending end of Moore’s Law and the synchronous rise of data-centric computing have precipitated a fundamental architectural crisis. Traditional von Neumann systems, characterized by the physical separation of processing units and memory, face an insurmountable "memory wall" where data movement consumes orders of magnitude more energy and latency than the computation itself. In this landscape, Analog In-Memory Computing (AIMC) has emerged not merely as an optimization but as a necessary paradigm shift, particularly for the acceleration of Deep Neural Networks (DNNs) and Spiking Neural Networks (SNNs). Among the candidate technologies for AIMC, Phase-Change Memory (PCM) stands out for its maturity, scalability, and unique physical properties that allow for the storage of continuous analog values—synaptic weights—within a single nanoscale device.  
However, the deployment of PCM as an analog synaptic element is fraught with complex challenges rooted in the very condensed matter physics that enables its operation. The stochastic nature of crystallization kinetics introduces programming noise, while the thermodynamic metastability of the amorphous phase leads to "conductance drift," a time-dependent relaxation phenomenon that degrades the accuracy of stored weights over time. Furthermore, the inherent asymmetry between the gradual crystallization (potentiation) and abrupt amorphization (depression) processes complicates the implementation of biologically plausible learning rules like Spike-Timing Dependent Plasticity (STDP).  
This report provides an exhaustive, expert-level analysis of the physics, engineering, and architectural integration of PCM for analog AI. It dissects the microscopic mechanisms of nucleation and growth, models the structural relaxation governing drift, and evaluates advanced mitigation strategies ranging from Projected PCM (Pro-PCM) devices to superlattice material stacks. Finally, it synthesizes the 2024-2025 industrial landscape, incorporating insights from the International Roadmap for Devices and Systems (IRDS) and recent benchmarking of Transformer and Mixture-of-Experts (MoE) models, to define the trajectory of this critical technology.

## ---

**1\. The Physics of Phase-Change via Chalcogenide Glasses**

The fundamental utility of PCM in analog computing lies in its ability to modulate electrical conductance over several orders of magnitude by altering the structural phase of a chalcogenide alloy—typically Germanium-Antimony-Tellurium (GST)—within an active volume. Unlike binary digital storage, which toggles between fully amorphous (RESET) and fully crystalline (SET) states, analog weight storage requires the precise, repeatable creation of intermediate mixed-phase states. Understanding the stability and programming of these states requires a deep dive into the thermodynamics and kinetics of phase transitions in chalcogenide glasses.

### **1.1 Electronic and Structural Properties of Chalcogenides**

The unique behavior of phase-change materials stems from their bonding characteristics. In the stable crystalline phase, GST typically adopts a distorted rock-salt structure where atoms are bonded via resonant p-orbitals. This "resonant bonding" creates a high degree of electronic delocalization, resulting in narrow bandgaps and high electrical conductivity (metallic or semi-metallic behavior). In contrast, the amorphous phase, formed by rapid melt-quenching, lacks long-range order and relies on localized covalent bonding. This structural disorder opens the bandgap and creates a manifold of localized trap states, resulting in high electrical resistivity and semiconducting behavior.1  
The contrast in electrical resistivity between the two phases can exceed three to four orders of magnitude, providing the dynamic range necessary for analog weight representation. However, the "analog" value is not a discrete material property but a macroscopic average of the current percolation through a heterogeneous mixture of amorphous and crystalline regions. The physics governing the transition between these states—specifically the crystallization kinetics—dictates the programming speed, granularity, and noise characteristics of the synaptic device.

### **1.2 Thermodynamics of the Amorphous-Crystalline Transition**

Crystallization is a first-order phase transition driven by the reduction in Gibbs free energy ($ \\Delta G $) as the material transforms from the metastable supercooled liquid (amorphous) state to the stable crystalline solid.  
The driving force for crystallization is given by:

$$\\Delta G \= \\Delta H \- T\\Delta S$$

where $\\Delta H$ is the enthalpy of fusion and $\\Delta S$ is the entropy change.  
While the crystalline state is energetically favorable below the melting temperature ($T\_m$), the transformation is inhibited by an energy barrier associated with the formation of a solid-liquid interface. Classical Nucleation Theory (CNT) describes this barrier. The free energy change to form a spherical nucleus of radius $r$ is:

$$\\Delta G(r) \= 4\\pi r^2 \\gamma \- \\frac{4}{3}\\pi r^3 \\Delta G\_v$$

Here, $\\gamma$ represents the interfacial surface tension, and $\\Delta G\_v$ is the volume free energy difference. The interplay between the surface energy term (scaling with $r^2$) and the volume energy term (scaling with $r^3$) creates a critical nucleation barrier $\\Delta G^\*$ at a critical radius $r^\*$. Nuclei smaller than $r^\*$ are unstable and dissolve back into the amorphous matrix, while those larger than $r^\*$ grow spontaneously.3  
This thermodynamic barrier is the primary source of "stochasticity" in PCM programming. The formation of a stable nucleus is a probabilistic event driven by thermal fluctuations. In an analog PCM cell, where the goal is to create a precise partial volume of crystallinity, this stochasticity manifests as "programming noise." Even with identical voltage pulses, the number and location of stable nuclei formed will vary from cycle to cycle, leading to variations in the final device conductance.

### **1.3 Nucleation-Dominated vs. Growth-Dominated Kinetics**

The crystallization mechanism is material-dependent and defines the "personality" of the synaptic device. Chalcogenides are broadly categorized into two kinetic classes: nucleation-dominated and growth-dominated.

#### **1.3.1 Nucleation-Dominated Systems (Ge₂Sb₂Te₅)**

The industry-standard alloy, $Ge\_2Sb\_2Te\_5$ (GST), is nucleation-dominated. In this regime, crystallization is initiated by the formation of many critical nuclei distributed throughout the amorphous volume.

* **Incubation Time:** A finite "incubation time" is required for clusters to reach critical size. This introduces a lag in the programming response and contributes to the abruptness of threshold switching.  
* **Analog Granularity:** Once nucleation begins, the process can be self-accelerating. Achieving fine-grained intermediate states requires careful control of the pulse amplitude to modulate the *rate* of nucleation ($I$) without triggering uncontrolled bulk crystallization. The nucleation rate is highly sensitive to temperature ($I \\propto \\exp(-\\Delta G^\*/kT)$), making the programming curve steep and sensitive to thermal variations.1  
* **Application:** GST is favored for its stability and high endurance but requires complex iterative algorithms to achieve high-precision analog states due to the randomness of nuclei distribution.

#### **1.3.2 Growth-Dominated Systems (Ag-In-Sb-Te, Sc-doped SbTe)**

Materials like Ag-In-Sb-Te (AIST), used in optical storage, and emerging Scandium-doped variants, are growth-dominated. In these systems, the nucleation probability is low, and crystallization proceeds primarily via the propagation of the crystalline interface from the pre-existing crystalline rim (the boundary between the amorphous mark and the surrounding crystalline material).

* **Interface Velocity:** The crystallization speed is determined by the growth velocity ($v\_g$), which has an Arrhenius dependence on temperature. Since growth starts from an existing interface, there is no stochastic incubation time.  
* **Linearity:** This mechanism is inherently more deterministic. The position of the amorphous-crystalline interface can be "pushed" in a continuous manner by varying the pulse duration or amplitude. This results in more linear programming characteristics, which is highly advantageous for analog "Accumulation" operations in neural network training.1  
* **Doping Effects:** Recent research into Sc-doped SbTe has shown that Scandium doping creates stable precursors that reduce the kinetic barrier for growth, enabling sub-nanosecond switching speeds while maintaining amorphous stability. This "interfacial regulation" is critical for high-speed neuromorphic inference engines.2

### **1.4 Percolation Theory and the Conductance-Volume Relationship**

A critical insight for analog PCM is that the electrical conductance ($G$) does not scale linearly with the crystalline volume fraction ($f\_c$). The current transport through the mixed-phase cell is a percolation problem.  
The amorphous phase is highly resistive, so current preferentially flows through crystalline filaments.

* **The Percolation Threshold ($f\_{th}$):** At low crystalline fractions, crystalline grains are isolated islands in an amorphous sea. Conductance remains low and is dominated by the amorphous phase. As $f\_c$ increases, grains begin to touch. At the percolation threshold ($f\_{th} \\approx 16-20\\%$ for 3D random networks), a continuous crystalline path connects the electrodes.  
* **Sensitivity:** Around $f\_{th}$, the conductance rises exponentially. A small change in crystal volume (e.g., adding one grain that bridges a gap) causes a massive jump in current. Conversely, well above the threshold, the conductance scales more linearly with the cross-sectional area of the filament.  
* **Implications for Analog Weights:** This non-linearity poses a significant control challenge. Operating a synapse near the percolation threshold results in high "weight noise" and poor reproducibility. Effective analog storage typically targets the regime well above the threshold, or utilizes "Confined Cell" or "Line Cell" geometries that constrain the current path to force a more linear dependence of resistance on the amorphous length, rather than complex 3D percolation dynamics.8

## ---

**2\. The Conductance Drift Phenomenon**

If stochastic crystallization is the challenge of *writing* analog weights, "conductance drift" is the challenge of *retaining* them. Conductance drift refers to the monotonic increase in the electrical resistance of the amorphous phase over time following a programming event. In digital memory, drift is a minor nuisance managed by read margins. In analog AI, where precise conductance values map directly to neural weights, drift is a catastrophic source of error that degrades inference accuracy over time.

### **2.1 Structural Relaxation: The Microscopic Mechanism**

The origin of drift lies in the thermodynamic nature of the amorphous phase. The amorphous state created by a RESET pulse is a "glass"—a frozen, non-equilibrium supercooled liquid. It possesses excess free energy and entropy compared to the ideal supercooled liquid state at that temperature. Consequently, the atomic structure is kinetically unstable and continuously evolves toward a more favorable energetic configuration, a process known as "structural relaxation" or "physical aging".11

#### **2.1.1 Defect Annihilation and Peierls Distortion**

There are two dominant theoretical frameworks describing this relaxation:

1. **Defect Annihilation:** The melt-quenched structure contains a high density of defects, such as homopolar bonds (Ge-Ge, Sb-Sb) and coordination errors. Over time, atoms locally rearrange to eliminate these high-energy bonds, increasing the local order (though remaining amorphous). This reduces the density of localized electronic states that facilitate hopping conduction, thereby increasing resistance.  
2. **Peierls Distortion Enhancement:** More recent analyses suggest that relaxation involves the collective adjustment of bond angles and lengths to maximize Peierls distortion. This structural distortion widens the optical bandgap and increases the activation energy for conduction. Since amorphous chalcogenides behave as Poole-Frenkel or hopping conductors, a slight increase in the activation energy leads to an exponential decrease in carrier mobility and concentration.2

Crucially, this relaxation is "collective"—it involves the cooperative motion of many atoms—and thus exhibits a broad distribution of relaxation times. This results in drift dynamics that persist over many decades of time, from nanoseconds to years, without saturating.12

### **2.2 Mathematical Modeling of Drift Dynamics**

Macroscopically, the evolution of conductance $G(t)$ follows a power-law relationship, often referred to as the "drift equation":

$$G(t) \= G(t\_0) \\left( \\frac{t}{t\_0} \\right)^{-\\nu}$$  
where $t\_0$ is an arbitrary reference time (typically 1 second or 1 microsecond after programming) and $\\nu$ is the **drift coefficient**.11

#### **2.2.1 The Drift Coefficient ($\\nu$)**

The drift exponent $\\nu$ is the critical metric for analog stability. It is not a universal constant but depends on the physical state of the cell:

* **Phase Dependency:** The crystalline phase is thermodynamically stable and does not drift ($\\nu\_{cryst} \\approx 0$). The amorphous phase drifts significantly ($\\nu\_{amorph} \\approx 0.03 \- 0.1$).  
* **State Dependency:** In a mixed-phase analog cell, the effective drift coefficient $\\nu\_{eff}$ depends on the amorphous volume fraction. A highly amorphous cell (low conductance) drifts faster than a mostly crystalline cell (high conductance). This state-dependent drift rate causes the distribution of weights to compress over time, reducing the "memory window" or dynamic range of the neural network.12  
* **Stoichiometry and Thickness:** Ge-rich alloys tend to have higher drift coefficients due to the flexibility of the Ge tetrahedral bonding network. Conversely, pure Antimony (Sb) or Sb-rich alloys show lower drift. Reducing the physical thickness of the PCM layer (e.g., in nanowires) has also been shown to reduce $\\nu$, likely due to surface pinning effects that inhibit structural relaxation.2

### **2.3 Stochasticity and Noise Spectral Density**

While the power law describes the mean behavior, drift is inherently stochastic. The drift coefficient $\\nu$ varies from cell to cycle and from cycle to cycle within the same cell. This variability ($\\sigma\_{\\nu}$) introduces a time-dependent noise term to the synaptic weight.  
The Signal-to-Noise Ratio (SNR) of an analog weight degrades over time as:

$$SNR(t) \\propto \\frac{1}{\\sigma\_{\\nu} \\ln(t/t\_0)}$$

This logarithmic degradation implies that while short-term accuracy might be high, the reliability of the neural network falters at long timescales (days/months) unless specific compensation techniques are employed. Furthermore, the noise spectral density of the amorphous phase exhibits $1/f$ (flicker) noise characteristics, which is consistent with a broad distribution of fluctuators (trap states) undergoing relaxation.12

### **2.4 Threshold Voltage Drift vs. Resistance Drift**

It is vital to distinguish between resistance drift and threshold voltage ($V\_{th}$) drift. While resistance drift affects the sub-threshold read current (inference), $V\_{th}$ drift affects the voltage required to switch the device (programming). $V\_{th}$ also drifts upwards with time ($\\Delta V\_{th} \\propto \\ln t$). This means that an "aged" cell is harder to switch than a "fresh" cell. In neuro-inspired learning algorithms that rely on precise voltage thresholds for updates (like STDP), $V\_{th}$ drift can desensitize the synapse to plasticity events over time, effectively "freezing" the weights.12

## ---

**3\. Neurotrophic Dynamics and Synaptic Emulation**

The ultimate goal of analog PCM is to emulate the plasticity of biological synapses, enabling hardware that learns from experience. This requires mapping biological learning rules—specifically Spike-Timing Dependent Plasticity (STDP)—onto the physical phase transitions of the chalcogenide glass.

### **3.1 Spike-Timing Dependent Plasticity (STDP) Implementation**

STDP is a Hebbian learning rule where the synaptic weight change ($\\Delta W$) depends on the temporal correlation ($\\Delta t \= t\_{post} \- t\_{pre}$) between pre-synaptic and post-synaptic spikes.

* **Long-Term Potentiation (LTP):** If the pre-spike precedes the post-spike ($\\Delta t \> 0$, causal), the synapse is strengthened.  
* **Long-Term Depression (LTD):** If the post-spike precedes the pre-spike ($\\Delta t \< 0$, anti-causal), the synapse is weakened.

In PCM, LTP is mapped to **crystallization** (conductance increase), and LTD is mapped to **amorphization** (conductance decrease). The implementation typically involves engineering the voltage waveforms of the pre- and post-neurons such that their superposition across the 1T1R (one-transistor, one-resistor) PCM cell exceeds the programming thresholds only at specific timing overlaps.

* **Waveform Engineering:** A common scheme uses a short positive pulse for the pre-spike and a shaped bipolar pulse (negative then positive) for the post-spike.  
  * For $\\Delta t \> 0$, the pre-pulse overlaps with the negative tail of the post-pulse, creating a large potential difference that exceeds the threshold for crystallization (SET).  
  * For $\\Delta t \< 0$, the overlap creates a voltage exceeding the melting threshold, triggering amorphization (RESET).  
    The precise shape of the pulses (e.g., exponential decay) determines the shape of the STDP learning window (the function $\\Delta W$ vs. $\\Delta t$).16

### **3.2 The Asymmetry Problem: Gradual SET vs. Abrupt RESET**

A fundamental mismatch exists between the physics of PCM and the requirements of ideal STDP: **Asymmetry**.

* **Gradual SET (LTP):** Crystallization is an accumulative process. A sequence of partial SET pulses can progressively grow the crystalline filament, allowing for a smooth, continuous increase in conductance. This provides high-resolution "learning" steps (e.g., 50-100 discrete states).  
* **Abrupt RESET (LTD):** Amorphization is a melt-quench process. Melting is a threshold phenomenon that tends to affect the entire hottest region of the filament simultaneously. It is extremely difficult to "partially melt" a filament in fine increments. Consequently, a single RESET pulse often obliterates the conductive path, dropping the conductance from high to low in one giant step.

This "Abrupt Reset" behavior is catastrophic for standard learning algorithms. If depression is binary while potentiation is analog, the weights tend to saturate at the maximum value, and the network loses the ability to fine-tune errors. The "synaptic blockade" phenomenon occurs where synapses cannot effectively be depressed to intermediate values, severely limiting the learning capacity of the network.16

### **3.3 Synaptic Architectures: 1T1R, 2-PCM, and Differential Pairs**

To overcome asymmetry and drift, architects have moved beyond the single device (1T1R) synapse.

#### **3.3.1 The 2-PCM Synapse (Differential Pair)**

This architecture uses *two* PCM devices to represent a single synaptic weight $W \= G^+ \- G^-$.

* **LTP:** Potentiation is achieved by sending a partial SET pulse to the $G^+$ device (increasing $W$).  
* **LTD:** Depression is achieved by sending a partial SET pulse to the $G^-$ device (increasing $G^-$, which decreases $W$).  
* **Key Advantage:** Both operations use **Crystallization (SET)**, which is the gradual, controllable regime of PCM. The problematic "Abrupt Reset" is avoided entirely during the learning phase.  
* **Drift Cancellation:** Since both $G^+$ and $G^-$ are PCM devices, they both suffer from conductance drift. However, if they drift at similar rates (common-mode noise), the difference $G^+ \- G^-$ remains relatively stable. This provides inherent drift resilience at the circuit level.14  
* **Refresh Cycles:** Eventually, both $G^+$ and $G^-$ will saturate at maximum conductance. A periodic "Refresh" or "Reset" cycle is required to read the net weight, reset both devices, and re-program them to the center of their dynamic range.

### **3.4 Sensitivity Modulation and Pulse Engineering**

Even within the gradual SET regime, the change in conductance ($\\Delta G$) is not constant; it depends on the current conductance $G$. Typically, PCM exhibits a non-linear update response where $\\Delta G$ decreases as $G$ approaches saturation. This non-linearity distorts the learning rule.

* **Pulse Engineering:** To linearize the weight update, "Program-and-Verify" algorithms or variable-amplitude pulse schemes are used. By increasing the pulse amplitude as the conductance increases, the driving force for crystallization is maintained, linearizing the LTP curve.  
* **Architectural Tolerance:** Simulations suggest that SNNs can tolerate significant non-linearity and asymmetry if the network depth is sufficient (3-layer architectures) and if input noise is used to regularize the learning. Benchmarks indicate that even with imperfect devices, recognition accuracy \>85% is achievable if drift is managed.16

## ---

**4\. Material and Device Engineering for Drift Mitigation**

While algorithmic compensation is powerful, the root cause of the error—the material instability—must be addressed for high-performance systems. The years 2024-2025 have seen the maturation of device structures that physically suppress or bypass drift.

### **4.1 Projected Phase-Change Memory (Pro-PCM)**

Projected PCM is a structural innovation that decouples the storage medium from the readout path.

* **Device Structure:** A standard PCM cell is modified by placing a highly conductive but resistive "projection liner" (e.g., Titanium Nitride or amorphous Carbon) in parallel with the phase-change material segment.  
* **Mechanism:**  
  * **Write Operation:** The write pulse is high voltage. The PCM material (in the amorphous state) breaks down (threshold switching) and becomes conductive, allowing sufficient current to flow through the PCM volume to heat and switch it. The liner shunts some current, but the majority of the *power* dissipates in the PCM.  
  * **Read Operation:** The read voltage is low (sub-threshold). The amorphous PCM is highly resistive ($\\sim M\\Omega$). The projection liner has a lower resistance ($\\sim 10-100 k\\Omega$). Consequently, the read current **bypasses** the amorphous volume and flows through the stable liner.  
* **Drift Bypass:** The "information" is stored in the *length* of the amorphous mark, which blocks a specific portion of the liner. As the amorphous material drifts (resistance increases from $10M\\Omega$ to $100M\\Omega$), it effectively remains an open circuit relative to the liner. The read current, flowing through the non-drifting liner, sees a constant resistance.  
* **Impact:** Pro-PCM effectively reduces the drift coefficient $\\nu$ to near zero for the readout path, enabling high-precision analog inference that remains stable for days or weeks without reprogramming.7

### **4.2 Superlattice and Nanocomposite Structures**

Material scientists have developed "Interfacial Phase Change Memory" (iPCM) to alter the thermodynamics of the glass itself.

* **Superlattices:** Stacks of alternating nanolayers (e.g., $GeTe / Sb\_2Te\_3$) confine the phase change to the 2D interfaces between layers. The switching is believed to be a reorientation of atoms at the interface rather than a bulk melt-quench. This reduces the entropic randomness of the amorphous state, thereby reducing the driving force for relaxation (drift).  
* **Ge₄Sb₆Te₇ Nanocomposites:** A 2024 breakthrough involves integrating nanocomposites with superlattices. The interfaces create **Van der Waals gaps** that act as thermal barriers, confining heat within the active layer.  
  * **Energy Efficiency:** This confinement reduces the switching power density to a record low of $\\approx 5 MW/cm^2$.  
  * **Stability:** The spatial confinement of the atoms inhibits long-range diffusion, suppressing crystal growth in the amorphous phase and significantly lowering the drift coefficient. These devices can operate at lower voltages ($0.7 V$), compatible with modern logic processes.24

### **4.3 Compositional Tuning: The Role of Stoichiometry**

Adjusting the Ge/Sb/Te ratio directly impacts the rigidity of the amorphous network.

* **Antimony-Rich Alloys:** Pure Sb or Sb-rich alloys form a more rigid amorphous structure with fewer degrees of freedom for relaxation than Ge-rich GST. This results in naturally lower drift coefficients ($\\nu \< 0.01$). However, these materials often have lower crystallization temperatures, trading retention time (data longevity) for stability (drift noise).2  
* **Doping:** Incorporating transition metals or Nitrogen can "stuff" the lattice or passivate defects, reducing the density of trap states involved in drift.

## ---

**5\. Circuit and Algorithmic Compensation**

Beyond the device, the peripheral circuitry and control algorithms play a crucial role in managing analog non-idealities.

### **5.1 Time-Based Encoding: PWM and PFM**

Traditional analog computing relies on Amplitude Modulation (AM), where the weight is represented by conductance $G$ and the input by voltage $V$, yielding current $I \= G \\cdot V$. This is directly susceptible to drift in $G$.  
Time-Based Encoding shifts the domain to time.

* **Pulse-Width Modulation (PWM):** The input activation is encoded as the *duration* of a pulse ($T\_{in}$). The total charge integrated at the output is $Q \= \\int\_{0}^{T\_{in}} I dt \= G \\cdot V \\cdot T\_{in}$.  
* **Drift Compensation:** While $G$ still drifts, the impact can be compensated by globally scaling the integration time $T\_{in}$ or the reference voltage $V$. If the average drift of the array is tracked, the system can simply "listen longer" to compensate for the reduced current of aged devices.  
* **Pulse Frequency Modulation (PFM):** Inputs are encoded as spike trains. The computation becomes a "spike counting" operation. This binary/digital representation of the *input* makes the system robust to noise in the input lines, although the weight multiplication remains analog. PFM is particularly well-suited for SNNs.26

### **5.2 Differential Readout and Reference Tracking**

To mitigate global variations (temperature, aging, common-mode drift), circuit designers use differential sensing.

* **Reference Cells ($G\_{ref}$):** A subset of PCM cells is programmed to known states and distributed throughout the array. These cells undergo the same thermal history and drift as the weight cells.  
* **Tracking:** The readout circuit measures the $G\_{ref}$ cells periodically. If $G\_{ref}$ has drifted by 10%, the sense amplifiers adjust their gain or integration window by a corresponding factor to cancel the error in the active weights.15

### **5.3 Iterative Programming and Slope Correction**

Achieving precise analog weights initially requires "Program-and-Verify" (Iterative programming).

1. Apply a partial SET pulse.  
2. Read the conductance.  
3. Calculate the error.  
4. Apply a corrective pulse (with adjusted amplitude).  
5. Repeat until within tolerance.  
   While this guarantees high initial precision (e.g., 4-5 bits), it is slow.  
* **Slope Correction:** For inference, the drift slope ($\\nu$) of the array is characterized. The system applies a time-dependent scalar correction factor to the output of the MVM engine: $I\_{corrected} \= I\_{read} \\cdot (t/t\_0)^{+\\nu}$. This digital post-processing effectively flattens the drift curve for the downstream digital logic.32

## ---

**6\. Analog AI Architectures and Performance Benchmarks**

The integration of these physics-aware strategies has enabled the deployment of PCM in high-performance AI accelerators. The benchmarks from 2024-2025 demonstrate that analog PCM is no longer just an academic curiosity but a viable competitor to GPUs for specific workloads.

### **6.1 The Matrix-Vector Multiplication (MVM) Engine**

The core value proposition of PCM is the physical execution of MVM in $O(1)$ time.

$$\\mathbf{y} \= W \\mathbf{x}$$

In a PCM crossbar array:

* $W\_{ij}$ is the conductance of the cell at row $i$, column $j$.  
* $x\_j$ is the voltage applied to row $j$.  
* $y\_i$ is the current collected at column $i$ (Kirchhoff’s Current Law).  
  This massive parallelism allows for the processing of entire neural network layers in a single clock cycle, independent of the layer size (up to the array dimensions), shattering the von Neumann bottleneck.

### **6.2 Transformer Acceleration and Attention Mechanisms**

Transformers (e.g., BERT, GPT) are dominated by the **Attention Mechanism**, which involves intensive MVM operations ($Q \\cdot K^T$ and $A \\cdot V$).

* **The Challenge:** Attention matrices are dynamic (computed at runtime), whereas PCM is efficient for static weights.  
* **The Solution:** Hybrid architectures. PCM arrays store the large, static Projection Weights ($W\_Q, W\_K, W\_V, W\_O$) and Feed-Forward Network (FFN) weights. The dynamic Attention Map computation is handled by a dedicated digital unit or a high-endurance capacitor-based analog unit.  
* **Performance:** In the **Long Range Arena (LRA)** benchmark, which tests performance on long sequences, PCM-based analog accelerators achieved inference accuracy within **2%** of full floating-point precision. This proves that the analog noise floor is low enough to support the subtle correlations captured by attention heads.34

### **6.3 Mixture-of-Experts (MoE) Implementation**

MoE models are the frontier of efficient LLMs. They use a "Router" to select a sparse subset of "Expert" networks for each token.

* **The GPU Problem:** MoE models have massive parameter counts but sparse activation. GPUs struggle because they must load massive weights from VRAM for only a few computations, leading to poor memory bandwidth utilization.  
* **The PCM Advantage:** In a 3D PCM architecture, *all* experts reside on-chip in non-volatile arrays. The "routing" simply involves activating the specific tiles corresponding to the chosen expert. There is no weight loading penalty.  
* **Result:** IBM Research (2025) demonstrated that PCM-based MoE accelerators outperform NVIDIA A100/H100 GPUs in energy efficiency (TOPS/W) and throughput for inference tasks, leveraging the "store-and-compute" capability to eliminate the memory bandwidth bottleneck.34

### **6.4 Accuracy Analysis: Floating Point vs. Analog**

A critical question is the "Equivalent Bit Precision" of analog PCM.

* **Storage Precision:** A Pro-PCM cell can reliably distinguish \~16-32 levels, equivalent to 4-5 bits.  
* **Computing Precision:** Due to the averaging effect of summing currents from thousands of synapses (Law of Large Numbers), the effective precision of the MVM output can approach 8-bit equivalent.  
* **Quantization:** Benchmarks on MobileBERT show that analog PCM achieves accuracy comparable to 8-bit integer (INT8) quantized digital models. The drift-induced error, when managed with Pro-PCM and compensation, remains below the quantization error floor of INT8 models for extended periods.35

## **Table 1: Comparative Benchmarks (2025 Projections)**

| Metric | Digital GPU (H100 class) | Analog PCM Accelerator (Pro-PCM) |
| :---- | :---- | :---- |
| **Compute Density** | High | Ultra-High (3D Stackable) |
| **Energy Efficiency** | \~1-10 TOPS/W | \~10-100 TOPS/W |
| **Data Movement** | High (HBM to Core) | Near Zero (In-Memory) |
| **Weight Precision** | FP16 / FP8 / INT4 (Exact) | \~4-8 bit (Stochastic) |
| **Drift Impact** | None | Requires Compensation |
| **MoE Efficiency** | Bandwidth Limited | Latency Optimized |
| **Transformer Accuracy** | Baseline (100%) | \>98% of Baseline 34 |

## ---

**7\. The 2025 Industrial Landscape and Roadmap**

The commercialization of PCM for AI is transitioning from exploratory research to roadmap integration.

### **7.1 The IRDS 2024/2025 Outlook**

The **International Roadmap for Devices and Systems (IRDS)** has solidified the role of emerging memory in "Autonomous Machine Computing."

* **The Split:** The roadmap bifurcates emerging memory into "Storage Class Memory" (latency reduction) and "Analog/Neuromorphic Compute" (energy reduction). PCM is the frontrunner for the latter, specifically for **inference** at the edge.  
* **Metrology Standards:** New protocols for measuring "Analog Retention" (drift stability) and "Linearity" are being standardized to allow apples-to-apples comparison between different PCM stacks and ReRAM devices.36

### **7.2 Competitive Analysis: PCM vs. ReRAM vs. Flash**

* **PCM:** Best for precision. Pro-PCM offers the most stable analog states. Mechanism (volume/melt) is robust but power-hungry during write.  
* **ReRAM (RRAM):** Best for density and read power. Filamentary switching is inherently noisier (Telegraph noise) and harder to control for \>2 bits of precision. IRDS positions ReRAM for binary/ternary weights or high-density storage, while PCM claims the high-precision analog niche.38  
* **Flash (NOR):** Existing technology. Can store analog values (floating gate charge) with high precision. However, high voltage programming and limited endurance prevent it from being used for *training* or dynamic weight updates. PCM offers superior endurance ($10^9$ cycles vs $10^5$ for Flash).30

### **7.3 Future Directions: 3D Integration and Edge Deployment**

The future lies in the vertical dimension.

* **3D Crosspoint:** Moving from 2D arrays to 3D vertical integration (similar to 3D NAND) is essential to achieve the synaptic density of the human brain ($10^{14}$ synapses) in a compact footprint.  
* **Edge AI:** Companies like Intel and STMicroelectronics are integrating PCM accelerators into "AI PC" chipsets and mobile SoCs. The goal is to enable "Always-On" AI (voice, gesture recognition) that operates within a milliwatt power budget, unachievable with digital logic.39

## ---

**8\. Conclusion**

The utilization of Phase-Change Memory for analog AI weights represents a convergence of condensed matter physics and computational neuroscience. The challenges are formidable: the very physics that enables non-volatility—the meta-stability of the amorphous phase—introduces conductance drift, a time-dependent error source that fundamentally antagonizes the precision required for deep learning inference.  
However, the field has transitioned from understanding these deficits to effectively engineering around them. The physics of structural relaxation is now well-quantified by power-law models, allowing for predictive compensation. More importantly, the invention of **Projected PCM** and **Interface-Confined Superlattices** has introduced a hardware abstraction layer that hides the messy glassy dynamics from the readout circuitry.  
As of 2025, the successful benchmarking of PCM-based analog accelerators on complex Transformer and Mixture-of-Experts workloads signals a maturation of the technology. With inference accuracy approaching floating-point standards and energy efficiency exceeding GPUs by orders of magnitude for specific workloads, PCM is poised to become the dominant substrate for the next generation of energy-efficient, edge-native AI.

### **Citations**

36

#### **Works cited**

1. Microstructure characterization, phase transition, and device application of phase-change memory materials \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10512918/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10512918/)  
2. Suppressing Structural Relaxation in Nanoscale Antimony to Enable Ultralow‐Drift Phase‐Change Memory Applications \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10477879/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10477879/)  
3. Principles of Crystal Nucleation and Growth, accessed December 24, 2025, [https://www.eps.mcgill.ca/\~jeannep/eps/EPSC644/Principles%20Of%20Nucleation%20And%20Growth.pdf](https://www.eps.mcgill.ca/~jeannep/eps/EPSC644/Principles%20Of%20Nucleation%20And%20Growth.pdf)  
4. Robust Simulations of Nanoscale Phase Change Memory: Dynamics and Retention \- PMC, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC8619026/](https://pmc.ncbi.nlm.nih.gov/articles/PMC8619026/)  
5. Molecular Dynamics Simulation of the Crystallization Behavior of Octadecane on a Homogeneous Nucleus \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2073-4352/12/7/987](https://www.mdpi.com/2073-4352/12/7/987)  
6. Phase-change properties of GeSbTe thin films deposited by plasma-enchanced atomic layer depositon \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC4385138/](https://pmc.ncbi.nlm.nih.gov/articles/PMC4385138/)  
7. Structural Assessment of Interfaces in Projected Phase-Change Memory \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-4991/12/10/1702](https://www.mdpi.com/2079-4991/12/10/1702)  
8. Modeling of Phase Change Memory: Nucleation, Growth and Amorphization Dynamics during Set and Reset: Part II – Discrete Grains \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/ielaam/16/8076920/8031275-aam.pdf](https://ieeexplore.ieee.org/ielaam/16/8076920/8031275-aam.pdf)  
9. (PDF) A Percolation Model of Thermal Conductivity for Filled Polymer Composites, accessed December 24, 2025, [https://www.researchgate.net/publication/249354498\_A\_Percolation\_Model\_of\_Thermal\_Conductivity\_for\_Filled\_Polymer\_Composites](https://www.researchgate.net/publication/249354498_A_Percolation_Model_of_Thermal_Conductivity_for_Filled_Polymer_Composites)  
10. Modeling Piezoresistive Behavior of Conductive Composite Sensors via Multi-State Percolation Theory \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2504-477X/9/7/354](https://www.mdpi.com/2504-477X/9/7/354)  
11. Impact of conductance drift on multi-PCM synaptic architectures ..., accessed December 24, 2025, [https://www.researchgate.net/publication/330581957\_Impact\_of\_conductance\_drift\_on\_multi-PCM\_synaptic\_architectures](https://www.researchgate.net/publication/330581957_Impact_of_conductance_drift_on_multi-PCM_synaptic_architectures)  
12. Statistics of Resistance Drift Due to Structural Relaxation in Phase-Change Memory Arrays | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/224169328\_Statistics\_of\_Resistance\_Drift\_Due\_to\_Structural\_Relaxation\_in\_Phase-Change\_Memory\_Arrays](https://www.researchgate.net/publication/224169328_Statistics_of_Resistance_Drift_Due_to_Structural_Relaxation_in_Phase-Change_Memory_Arrays)  
13. Resistance and Threshold Switching Voltage Drift Behavior in PhaseChange Memory and Their Temperature Dependence at Microsecond Time Scales Studied Using a MicroThermal Stage | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/238523142\_Resistance\_and\_Threshold\_Switching\_Voltage\_Drift\_Behavior\_in\_PhaseChange\_Memory\_and\_Their\_Temperature\_Dependence\_at\_Microsecond\_Time\_Scales\_Studied\_Using\_a\_MicroThermal\_Stage](https://www.researchgate.net/publication/238523142_Resistance_and_Threshold_Switching_Voltage_Drift_Behavior_in_PhaseChange_Memory_and_Their_Temperature_Dependence_at_Microsecond_Time_Scales_Studied_Using_a_MicroThermal_Stage)  
14. Phase Change Memory Drift Compensation in Spiking Neural Networks Using a Non-Linear Current Scaling Strategy \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9268/14/4/50](https://www.mdpi.com/2079-9268/14/4/50)  
15. A Readout Scheme for PCM-Based Analog In-Memory Computing With Drift Compensation Through Reference Conductance Tracking \- Unibo, accessed December 24, 2025, [https://cris.unibo.it/bitstream/11585/976134/3/A\_Readout\_Scheme\_for\_PCM-Based\_Analog\_In-Memory\_Computing\_With\_Drift\_Compensation\_Through\_Reference\_Conductance\_Tracking.pdf](https://cris.unibo.it/bitstream/11585/976134/3/A_Readout_Scheme_for_PCM-Based_Analog_In-Memory_Computing_With_Drift_Compensation_Through_Reference_Conductance_Tracking.pdf)  
16. Unsupervised Learning by Spike Timing Dependent ... \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2016.00056/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2016.00056/full)  
17. Unsupervised Learning by Spike Timing Dependent Plasticity in Phase Change Memory (PCM) Synapses \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/297662715\_Unsupervised\_Learning\_by\_Spike\_Timing\_Dependent\_Plasticity\_in\_Phase\_Change\_Memory\_PCM\_Synapses](https://www.researchgate.net/publication/297662715_Unsupervised_Learning_by_Spike_Timing_Dependent_Plasticity_in_Phase_Change_Memory_PCM_Synapses)  
18. Differential Phase Change Memory (PCM) Cell for Drift-Compensated In-Memory Computing \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/iel8/16/10778112/10736424.pdf](https://ieeexplore.ieee.org/iel8/16/10778112/10736424.pdf)  
19. Optimization of Projected Phase Change Memory for Analog In ..., accessed December 24, 2025, [https://research.ibm.com/publications/optimization-of-projected-phase-change-memory-for-analog-in-memory-computing-inference](https://research.ibm.com/publications/optimization-of-projected-phase-change-memory-for-analog-in-memory-computing-inference)  
20. Optimization of Projected Phase Change Memory for Application in Analog Neuromorphic Computing \- IBM Research, accessed December 24, 2025, [https://research.ibm.com/publications/optimization-of-projected-phase-change-memory-for-application-in-analog-neuromorphic-computing](https://research.ibm.com/publications/optimization-of-projected-phase-change-memory-for-application-in-analog-neuromorphic-computing)  
21. Structural Assessment of Interfaces in Projected Phase-Change Memory \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/360657381\_Structural\_Assessment\_of\_Interfaces\_in\_Projected\_Phase-Change\_Memory](https://www.researchgate.net/publication/360657381_Structural_Assessment_of_Interfaces_in_Projected_Phase-Change_Memory)  
22. Projected phase-change memory devices \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC4569800/](https://pmc.ncbi.nlm.nih.gov/articles/PMC4569800/)  
23. Projected memory device with reduced minimum conductance state \- Google Patents, accessed December 24, 2025, [https://patents.google.com/patent/US11665985/en](https://patents.google.com/patent/US11665985/en)  
24. Novel nanocomposite-superlattices for low energy and high stability ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10803317/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10803317/)  
25. National Institute of Standards and Technology \- Semiconductor Engineering, accessed December 24, 2025, [https://semiengineering.com/tag/national-institute-of-standards-and-technology/](https://semiengineering.com/tag/national-institute-of-standards-and-technology/)  
26. Pulse-frequency modulation \- Wikipedia, accessed December 24, 2025, [https://en.wikipedia.org/wiki/Pulse-frequency\_modulation](https://en.wikipedia.org/wiki/Pulse-frequency_modulation)  
27. Pulse Frequency Modulation (PFM): It's Not PWM, But Related \- Control.com, accessed December 24, 2025, [https://control.com/technical-articles/pulse-frequency-modulation-pfm-its-not-pwm-but-related/](https://control.com/technical-articles/pulse-frequency-modulation-pfm-its-not-pwm-but-related/)  
28. US11805713B2 \- Drift mitigation for resistive memory devices \- Google Patents, accessed December 24, 2025, [https://patents.google.com/patent/US11805713B2/en](https://patents.google.com/patent/US11805713B2/en)  
29. Dynamic pass bias control for temperature-resilient neural networks using vertical NAND flash memory \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12484496/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12484496/)  
30. A Noise-Resilient Neuromorphic Digit Classifier Based on NOR Flash Memories with Pulse–Width Modulation Scheme \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9292/10/22/2784](https://www.mdpi.com/2079-9292/10/22/2784)  
31. Smart Sensing of Multi-bit Resistive Memory using a Single Reference \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/396387464\_Smart\_Sensing\_of\_Multi-bit\_Resistive\_Memory\_using\_a\_Single\_Reference](https://www.researchgate.net/publication/396387464_Smart_Sensing_of_Multi-bit_Resistive_Memory_using_a_Single_Reference)  
32. Impact of conductance drift on multi-PCM synaptic architectures \- Semantic Scholar, accessed December 24, 2025, [https://www.semanticscholar.org/paper/Impact-of-conductance-drift-on-multi-PCM-synaptic-Boybat-Nandakumar/f17e3c0f610bb30a3ed496220ef1cf221522a5b4](https://www.semanticscholar.org/paper/Impact-of-conductance-drift-on-multi-PCM-synaptic-Boybat-Nandakumar/f17e3c0f610bb30a3ed496220ef1cf221522a5b4)  
33. Combined HW/SW Drift and Variability Mitigation for PCM-Based Analog In-Memory Computing for Neural Network Applications \- Unibo, accessed December 24, 2025, [https://cris.unibo.it/bitstream/11585/913711/5/Combined\_HW\_SW\_Drift\_and\_Variability\_Mitigation\_for\_PCM-based\_Analog\_In-memory\_Computing\_for\_Neural\_Network\_Applications\_IRIS.pdf](https://cris.unibo.it/bitstream/11585/913711/5/Combined_HW_SW_Drift_and_Variability_Mitigation_for_PCM-based_Analog_In-memory_Computing_for_Neural_Network_Applications_IRIS.pdf)  
34. Analog in-memory computing could power tomorrow's AI models ..., accessed December 24, 2025, [https://research.ibm.com/blog/how-can-analog-in-memory-computing-power-transformer-models](https://research.ibm.com/blog/how-can-analog-in-memory-computing-power-transformer-models)  
35. Analog Foundation Models \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2505.09663v3](https://arxiv.org/html/2505.09663v3)  
36. 2024 EDITION \- IRDS \- IEEE, accessed December 24, 2025, [https://irds.ieee.org/images/files/pdf/2024/2024IRDS\_WP\_AMC.pdf](https://irds.ieee.org/images/files/pdf/2024/2024IRDS_WP_AMC.pdf)  
37. 2024 IRDS Metrology \- IEEE, accessed December 24, 2025, [https://irds.ieee.org/images/files/pdf/2024/2024IRDS\_MET.pdf](https://irds.ieee.org/images/files/pdf/2024/2024IRDS_MET.pdf)  
38. IEDM+2024+Archive.pdf \- Squarespace, accessed December 24, 2025, [https://static1.squarespace.com/static/67a3eee4385dfb3390804f02/t/67d4b9026674b64dfd6d035b/1741994245226/IEDM+2024+Archive.pdf](https://static1.squarespace.com/static/67a3eee4385dfb3390804f02/t/67d4b9026674b64dfd6d035b/1741994245226/IEDM+2024+Archive.pdf)  
39. Intel Extends Leadership in AI PCs and Edge Computing at CES 2025 \- Intel Newsroom, accessed December 24, 2025, [https://newsroom.intel.com/client-computing/2025-ces-client-computing-news](https://newsroom.intel.com/client-computing/2025-ces-client-computing-news)  
40. The Promise of Analog AI: Could In-Memory Computing Revolutionize Edge Devices?, accessed December 24, 2025, [https://runtimerec.com/the-promise-of-analog-ai-could-in-memory-computing-revolutionize-edge-devices/](https://runtimerec.com/the-promise-of-analog-ai-could-in-memory-computing-revolutionize-edge-devices/)  
41. PCM-Based Analog Compute-In-Memory: Impact of Device Non-Idealities on Inference Accuracy \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/354886874\_PCM-Based\_Analog\_Compute-In-Memory\_Impact\_of\_Device\_Non-Idealities\_on\_Inference\_Accuracy](https://www.researchgate.net/publication/354886874_PCM-Based_Analog_Compute-In-Memory_Impact_of_Device_Non-Idealities_on_Inference_Accuracy)  
42. Neuromorphic Computing Materials and Electronics Industry Standards \- Patsnap Eureka, accessed December 24, 2025, [https://eureka.patsnap.com/report-neuromorphic-computing-materials-and-electronics-industry-standards](https://eureka.patsnap.com/report-neuromorphic-computing-materials-and-electronics-industry-standards)  
43. 저작자표시-동일조건변경허락 2.0 대한민국 이용자는 아래의 조건을 따르는 경우에 한하여 자유 \- Scholarworks@UNIST, accessed December 24, 2025, [https://scholarworks.unist.ac.kr/bitstream/201301/82234/2/200000743619.pdf](https://scholarworks.unist.ac.kr/bitstream/201301/82234/2/200000743619.pdf)  
44. Effect of Resistance Drift on the Activation Energy for Crystallization in Phase Change Memory | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/313527115\_Effect\_of\_Resistance\_Drift\_on\_the\_Activation\_Energy\_for\_Crystallization\_in\_Phase\_Change\_Memory](https://www.researchgate.net/publication/313527115_Effect_of_Resistance_Drift_on_the_Activation_Energy_for_Crystallization_in_Phase_Change_Memory)  
45. Demonstration of the Systematic Evaluation of an Optical Lattice Clock Using the Drift-Insensitive Self-Comparison Method \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2076-3417/11/3/1206](https://www.mdpi.com/2076-3417/11/3/1206)  
46. Technical Performance | The 2025 AI Index Report | Stanford HAI, accessed December 24, 2025, [https://hai.stanford.edu/ai-index/2025-ai-index-report/technical-performance](https://hai.stanford.edu/ai-index/2025-ai-index-report/technical-performance)  
47. Growth dominated crystallization of GeTe mushroom cells during partial SET operation, accessed December 24, 2025, [https://www.researchgate.net/publication/367379442\_Growth\_dominated\_crystallization\_of\_GeTe\_mushroom\_cells\_during\_partial\_SET\_operation](https://www.researchgate.net/publication/367379442_Growth_dominated_crystallization_of_GeTe_mushroom_cells_during_partial_SET_operation)  
48. When in-memory computing meets spiking neural networks—A perspective on device-circuit-system \- AIP Publishing, accessed December 24, 2025, [https://pubs.aip.org/aip/apr/article-pdf/doi/10.1063/5.0211040/20169400/031325\_1\_5.0211040.pdf](https://pubs.aip.org/aip/apr/article-pdf/doi/10.1063/5.0211040/20169400/031325_1_5.0211040.pdf)