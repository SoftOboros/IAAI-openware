# **Comprehensive Report on Ferroelectric (FeFET) and Flash/Floating-Gate Weight Encoding for Analog AI: Physics, Reliability, and 3D Architectures**

## **1\. Executive Summary**

The proliferation of data-centric computing, driven primarily by deep neural network (DNN) inference and training workloads, has exposed the fundamental latency and energy limitations of the von Neumann architecture. The "memory wall"—the bottleneck caused by the physical separation of processing units and memory arrays—necessitates a paradigm shift toward In-Memory Computing (IMC). Among the various implementations of IMC, Analog In-Memory Computing (AIMC) utilizing non-volatile memory (NVM) devices to perform Matrix-Vector Multiplication (MVM) in the analog domain offers the potential for orders-of-magnitude improvements in energy efficiency (TOPS/W) and compute density (TOPS/mm²).  
This report presents an exhaustive technical analysis of the two primary transistor-based candidates for analog weight storage: the emerging Ferroelectric Field-Effect Transistor (FeFET) based on hafnium oxide ($HfO\_2$), and the mature Flash memory technology utilizing Charge Trap (CT) or Floating Gate (FG) mechanisms. The analysis synthesizes data from recent developments in material science, device physics, and circuit design to provide a definitive comparison of their efficacy for next-generation AI accelerators.  
The divergence in weight encoding physics between these technologies is profound. Flash memory relies on the electrostatic modulation of the channel via charge stored in a potential well ($Si\_3N\_4$ or polysilicon). This mechanism, refined over decades of commercial scaling, offers high-precision analog states through Incremental Step Pulse Programming (ISPP) and unique subthreshold capabilities that enable log-domain arithmetic.1 However, the fundamental physics of charge injection—requiring high-energy tunneling carriers—imposes severe constraints on operating voltage (\>15V) and endurance ($10^4$-$10^5$ cycles), limiting its utility for training applications.4  
Conversely, FeFETs encode weights via the polarization orientation of ferroelectric domains within the gate dielectric. This field-driven switching mechanism allows for low-voltage (\<5V), high-speed (\<10ns), and high-endurance operation.6 However, the stochastic nature of domain nucleation in polycrystalline films, described by the Nucleation-Limited Switching (NLS) model, introduces non-linearities and history-dependent behavior that complicate precise analog tuning.8 Furthermore, the stability of the intermediate polarization states required for multi-bit storage is perpetually threatened by the depolarization field ($E\_{dep}$), a thermodynamic restoring force that drives weight drift.10  
A specific focus of this report is the transition to 3D architectures (Section 4 Deep Dive). As planar scaling saturates, both technologies are moving toward vertical integration. 3D NAND Flash faces challenges with channel mobility and lateral charge migration in charge-trap layers.5 The Vertical FeFET (V-FeFET) promises higher density but encounters severe manufacturing hurdles regarding the uniform crystallization of high-aspect-ratio ferroelectric films and the integration of alternative channel materials like Indium Gallium Zinc Oxide (IGZO) to mitigate interfacial instability.11  
The findings suggest a bifurcated future: Flash memory, with its superior static precision and established 3D manufacturing base, remains the dominant solution for inference-heavy edge applications where weights are static. FeFETs, offering speed and endurance, are poised to revolutionize on-chip training and dynamic weight applications, provided the challenges of linearity and 3D process integration can be overcome through material engineering and novel charge-domain architectural schemes.13

## ---

**2\. Fundamental Physics of Analog Weight Encoding**

The core requirement for analog AI is the ability to modulate the conductance ($G$) of a memory cell into a continuum of states, $G \\in \[G\_{min}, G\_{max}\]$, representing synaptic weights $w$. The physical mechanisms governing this modulation determine the device's linearity, symmetry, precision, and energy consumption.

### **2.1 Ferroelectric FeFET Physics: Polarization Dynamics**

The storage element in a FeFET is the ferroelectric gate dielectric, typically doped hafnium oxide ($HfO\_2$). Unlike traditional perovskite ferroelectrics (e.g., PZT), which are incompatible with CMOS processing, doped $HfO\_2$ (such as HZO) can be integrated into standard high-k metal gate stacks. The analog weight is encoded by the **Remnant Polarization ($P\_r$)**, which modulates the transistor's threshold voltage ($V\_{th}$) via the ferroelectric field effect.

#### **2.1.1 Crystallographic Origins and Phase Stability**

The ferroelectricity in $HfO\_2$ arises from the non-centrosymmetric orthorhombic phase (space group $Pca2\_1$). This phase is metastable under standard conditions; bulk $HfO\_2$ prefers the stable monoclinic phase ($P2\_1/c$), which is dielectric (non-ferroelectric). Stabilizing the $Pca2\_1$ phase requires precise control over dopants, strain, and thermal annealing.15

* **Zirconium (Zr) Doping:** Since $Zr^{4+}$ and $Hf^{4+}$ have nearly identical ionic radii (due to lanthanide contraction), $Hf\_{0.5}Zr\_{0.5}O\_2$ (HZO) forms a solid solution that lowers the crystallization temperature and creates a morphotropic phase boundary (MPB) between the tetragonal and orthorhombic phases. This proximity to the MPB enhances polarizability, yielding high $P\_r$ values (\~60 $\\mu C/cm^2$) robust enough for memory windows.15  
* **Silicon (Si) Doping:** Si acts as a crucial stabilizer for the tetragonal phase, the precursor to the ferroelectric orthorhombic phase. Because Si prefers $sp^3$ hybridization, it distorts the lattice, effectively lowering the energy barrier for the transformation. This results in a reduction of the Coercive Field ($E\_c$), which is vital for reducing the write voltage in low-power analog accelerators.17  
* **Lanthanum (La) and Yttrium (Y) Doping:** These larger ions introduce tensile strain and oxygen vacancies. While this might seem detrimental, for analog applications, it is beneficial. The local strain variations broaden the distribution of coercive fields across the film. A broader $E\_c$ distribution "smears out" the typically abrupt switching of domains, allowing for a more gradual, linear modulation of polarization with applied voltage—a key requirement for analog weight updates.18

#### **2.1.2 Nucleation-Limited Switching (NLS) Kinetics**

Understanding the time-dependent switching of polarization is critical for controlling analog weights. Early models utilized the Kolmogorov-Avrami-Ishibashi (KAI) theory, which assumes switching is driven by the unrestricted growth of domains from pre-existing nuclei. However, KAI fails to describe the behavior of polycrystalline $HfO\_2$ thin films used in modern FeFETs.8  
The **Nucleation-Limited Switching (NLS)** model provides the correct physical description.8 In polycrystalline films, grain boundaries act as pinning sites that impede domain wall propagation. Consequently:

1. **Granularity:** The film behaves as an ensemble of independent grains (or sub-grains).  
2. **Stochastic Wait Times:** Each grain has a specific "waiting time" for nucleation that depends exponentially on the local electric field and the activation energy barrier.  
3. **Implications for Analog Weights:** When a write pulse is applied, it does not essentially "grow" a single domain; rather, it statistically nucleates a subset of grains. The analog state is the aggregate effect of these flipped grains. This explains the "step-like" or granular nature of conductance changes in scaled FeFETs. As device dimensions shrink, the number of grains decreases, increasing the discreteness of the weight update and degrading analog precision.8

Recent atomistic simulations on similar nitride ferroelectrics (AlScN) reveal that switching at low fields (the regime used for fine-tuning weights) is dominated by **creep-like domain wall motion**, whereas high fields trigger collective **nucleation events**.9 This dual mechanism complicates the control circuitry, as the voltage-time relationship is highly non-linear.

#### **2.1.3 The History Effect and Minor Loops**

In analog AI training, weights are updated incrementally (e.g., $w\_{new} \= w\_{old} \+ \\Delta w$). This requires operating the FeFET in **minor hysteresis loops**, where the polarization is never fully saturated. A major physical challenge here is the **History Effect**.19  
The voltage required to achieve a target polarization state depends on the device's prior state trajectory. This is accurately modeled by the **Preisach Model**, which visualizes the material as a collection of hysterons with distinct up/down switching thresholds. If a device is in a specific conductance state reached from "all-up," its internal domain configuration differs from the same conductance reached from "all-down." This path dependence introduces errors in the weight update, effectively acting as noise in the training gradient. To mitigate this, complex "reset-before-write" schemes are often proposed, but these introduce latency and power penalties.20

### **2.2 Flash Memory Physics: Charge Trapping and Transport**

Flash memory relies on the storage of electrons in a potential well separated from the channel by a tunnel oxide. The physics of Flash is governed by electrostatics and quantum tunneling, offering a fundamentally different set of trade-offs compared to the kinetic switching of FeFETs.

#### **2.2.1 Charge Storage Mechanisms: Floating Gate vs. Charge Trap**

While traditional Floating Gate (FG) cells use a conductive polysilicon layer, modern analog AI accelerators predominantly use **Charge Trap (CT)** technology (e.g., SONOS, TANOS) due to its superior scalability and reduced cell-to-cell interference.1

* **SONOS (Silicon-Oxide-Nitride-Oxide-Silicon):** Electrons are stored in discrete trap sites (defects) within an insulating silicon nitride ($Si\_3N\_4$) layer. This spatial localization of charge makes the cell more robust against localized oxide defects; a single leakage path drains only the local charge, not the entire cell.  
* **TANOS (TaN-Alumina-Nitride-Oxide-Silicon):** This evolution replaces the top oxide with high-k Alumina ($Al\_2O\_3$) and the control gate with metal (TaN). The high-k blocking layer prevents electron back-tunneling from the gate during erase, enabling higher erase saturation levels and faster operation.4

#### **2.2.2 Precision Programming: ISPP**

To achieve the high-precision states (e.g., \>8 bits) required for analog inference, Flash employs **Incremental Step Pulse Programming (ISPP)**.2

1. **Mechanism:** A programming voltage pulse $V\_{pgm}$ is applied to inject electrons via Fowler-Nordheim (FN) tunneling.  
2. **Verification:** The cell is read. If the target $V\_{th}$ is not reached, $V\_{pgm}$ is incremented by a small step $\\Delta V\_{step}$ (e.g., 0.1V).  
3. **Self-Limiting Physics:** As the trap layer fills, the stored negative charge creates a Coulombic potential that repels subsequent incoming electrons. This negative feedback linearizes the relationship between the applied pulse number and the $V\_{th}$ shift, enabling extremely tight control over the weight distribution.  
4. **Trade-off:** While ISPP provides exquisite precision, the iterative read-verify loop dominates the write latency, making Flash writes ($10-100 \\mu s$) orders of magnitude slower than FeFET writes ($10-100 ns$).2

#### **2.2.3 Subthreshold Physics and Log-Domain Arithmetic**

A unique and powerful attribute of Flash for analog computing is its behavior in the subthreshold regime. The drain current ($I\_{ds}$) of a MOSFET in weak inversion is exponentially related to the gate voltage ($V\_{gs}$) and threshold voltage ($V\_{th}$):

$$I\_{ds} \= I\_0 \\cdot \\exp\\left(\\frac{\\kappa(V\_{gs} \- V\_{th})}{V\_T}\\right)$$  
where $V\_T \= kT/q$ is the thermal voltage. By encoding the weight $w$ linearly into the threshold voltage ($V\_{th} \\propto \-\\ln(w)$), the transistor effectively stores the logarithm of the weight. When a voltage representing the log of the input ($\\ln(x)$) is added to the gate, the resulting current represents the product $x \\cdot w$.  
This **Log-Domain Arithmetic** capability allows Flash-based accelerators to perform multiplication and complex non-linear functions (like neural activations) with minimal circuitry and femtojoule-level energy consumption.3 However, the dependence on $V\_T$ makes this approach highly sensitive to temperature, necessitating sophisticated temperature-compensation circuits or novel "geometric" encoding schemes that rely on transistor width ratios rather than absolute charge.26

### **2.3 Comparative Physics Summary**

| Feature | FeFET (Ferroelectric) | Flash (Charge Trap) |
| :---- | :---- | :---- |
| **Physical State** | Domain Orientation ($P$) | Trapped Charge ($Q$) |
| **Switching Mechanism** | Field-driven Nucleation (NLS Model) | Tunneling (FN) / Hot Carrier Injection |
| **Intrinsic Linearity** | Low (Sigmoidal, Stochastic) | High (Self-limiting, Linearized via ISPP) |
| **Write Speed** | Fast ($\< 10 ns$) | Slow ($\> 10 \\mu s$) due to ISPP loop |
| **Voltage Requirement** | Low ($\< 5 V$) | High ($\> 15 V$) |
| **Endurance** | High ($10^{10} \- 10^{12}$) | Limited ($10^4 \- 10^5$) |
| **Compute Regime** | Strong Inversion (typically) | Subthreshold (Log-domain capable) |

## ---

**3\. Reliability and Stability Mechanisms**

In digital memory, stability is binary: a "1" must stay a "1". In analog memory, *any* drift in the stored state constitutes a computation error. The physical mechanisms driving this drift are distinct for charge-based and polarization-based devices.

### **3.1 FeFET Instabilities: The Depolarization Challenge**

The Achilles' heel of analog FeFETs is the **Depolarization Field ($E\_{dep}$)**. In a FeFET structure, the ferroelectric layer is stacked in series with a dielectric Interfacial Layer (IL) and the semiconductor channel. Electrostatics dictates that the polarization charge $P$ induces an electric field in the dielectric layers that opposes the polarization direction.27

$$E\_{dep} \\approx \-\\frac{P}{\\epsilon\_{fe} \+ \\epsilon\_{IL} (t\_{fe}/t\_{IL})}$$

#### **3.1.1 Mechanisms of Analog Weight Drift**

In digital states (saturated $P$), the coercive field is usually large enough to resist $E\_{dep}$. However, analog weights rely on **intermediate polarization states**, which physically consist of a complex texture of up and down domains.

1. **Domain Relaxation:** The domain walls in these intermediate states are high-energy interfaces. Under the constant pressure of $E\_{dep}$, these walls tend to move to minimize the system's free energy, causing the programmed weight to "relax" or drift toward a ground state over time. This drift is often logarithmic with time.10  
2. **Screening Charge Dynamics:** To stabilize the polarization, charge carriers from the channel or gate often become trapped at the ferroelectric/IL interface. These charges screen $E\_{dep}$. However, these trapped charges are not permanent; they can detrap (tunnel back to the channel) over timescales ranging from milliseconds to years. When they detrap, the unscreened $E\_{dep}$ re-emerges, causing sudden shifts in polarization and $V\_{th}$.10

#### **3.1.2 Endurance: Wake-up and Fatigue**

FeFETs exhibit time-variant behavior during cycling:

* **Wake-up:** In pristine devices, domains are often pinned by oxygen vacancies. Initial cycling redistributes these defects, unpinning domains and increasing the switchable polarization ($P\_r$). For an analog accelerator, this is problematic because the "dynamic range" of the weights changes during operation, requiring the system to constantly recalibrate its read/write pulses.28  
* **Fatigue:** After extensive cycling ($\>10^{10}$), new defects are generated that pin domains or create leakage paths, eventually collapsing the memory window.

### **3.2 Flash Reliability: Lateral Migration and Disturb**

While Flash cells are physically robust, the scaling to dense 3D arrays introduces specific charge loss mechanisms that degrade analog precision.

#### **3.2.1 Lateral Charge Migration (LCM)**

In 3D Charge Trap NAND, the silicon nitride storage layer is often a continuous tube running vertically along the string. The charge stored in a specific cell (wordline) is not physically isolated from its neighbors.

* **Mechanism:** Driven by the concentration gradient and the large internal electric fields between cells (especially if adjacent cells hold very different weight values, e.g., "1.0" next to "0.0"), electrons can hop or diffuse laterally through the trap sites.10  
* **Analog Impact:** This diffusion acts as a "blurring" filter on the stored weight vector. Sharp contrasts in weights are smoothed out over time, degrading the inference accuracy of the neural network. This effect is strongly temperature-dependent (Arrhenius behavior), posing risks for edge AI devices operating in uncontrolled thermal environments.26

#### **3.2.2 Read Disturb**

In inference applications, weights are read continuously but rarely written. To read a specific cell in a NAND string, all other unselected cells must be turned on by applying a high **Pass Voltage ($V\_{pass}$)** (typically \~6V).

* **Mechanism:** This $V\_{pass}$ acts as a weak programming pulse. Over millions of read cycles (inference operations), it can inject small amounts of charge into the unselected cells.  
* **Drift:** This causes a systematic upward drift in the threshold voltage of all cells in the string. For analog weights, this manifests as a gradual corruption of the model, requiring periodic "refresh" (rewrite) operations to restore accuracy.10

### **3.3 Comparative Reliability Analysis**

| Mechanism | FeFET Effect | Flash Effect |
| :---- | :---- | :---- |
| **Primary Drift Source** | Depolarization Field ($E\_{dep}$) | Lateral Charge Migration (Diffusion) |
| **Time Dependence** | Logarithmic (Fast initial decay) | Power-law / Logarithmic (Slow) |
| **Temperature Sensitivity** | High (affects nucleation/$E\_c$) | High (accelerates diffusion) |
| **Endurance Limit** | $10^{10}$+ (Fatigue limited) | $10^5$ (Oxide breakdown limited) |
| **Read Disturb** | Low (Field-effect read) | High (Pass voltage stress) |
| **Mitigation Strategy** | Increase Capacitance Ratio (1T-1C) | Charge Domain Computing / Refresh |

## ---

**4\. Deep Dive: 3D Architectures and Vertical Scaling**

As the demand for AI model size explodes, planar scaling of memory arrays has become insufficient. The industry has pivoted to 3D vertical integration. This section analyzes the architectural evolution, manufacturing physics, and specific challenges of adapting 3D structures for analog weight storage.

### **4.1 3D NAND Flash: The Incumbent Architecture**

3D NAND utilizes a "Gate-All-Around" (GAA) cylindrical topology. The process involves depositing alternating layers of oxide and nitride (ONON stack), etching a high-aspect-ratio channel hole, and depositing the ONO dielectric and polysilicon channel.

#### **4.1.1 Architectural Constraints for Analog AI**

While 3D NAND offers immense density ($\>100 Gb/mm^2$), adapting it for analog compute presents physical bottlenecks:

1. **Channel Mobility and Current:** The vertical channel is typically **Polysilicon (poly-Si)**. Unlike single-crystal silicon, poly-Si suffers from grain boundary scattering, resulting in low carrier mobility ($\\mu \\approx 10-30 cm^2/V\\cdot s$).  
   * **Impact:** Low mobility limits the "Read Current" ($I\_{cell}$) available for the analog multiplication ($I \= G \\cdot V$). Low currents degrade the Signal-to-Noise Ratio (SNR) and limit the inference speed.5  
2. **Series Resistance (IR Drop):** The vertical string behaves as a resistive wire. As current flows through the string during a summation operation, the voltage drops along the channel. This means the "Source" voltage of a cell at the top of the string differs from one at the bottom.  
   * **Impact:** This introduces a systematic, position-dependent error in the VMM computation. The effective $V\_{gs}$ seen by the cell is $V\_{gate} \- V\_{channel}(z)$, where $V\_{channel}(z)$ varies with height $z$. Compensating for this requires complex training-aware calibration.  
3. **Dielectric Breakdown:** To increase density, the vertical pitch (distance between wordlines) is aggressively scaled. This increases the electric field stress in the inter-poly dielectrics, heightening the risk of breakdown during the high-voltage operations ($\>20V$) required for Flash programming.5

#### **4.1.2 The Hybrid FeNAND Proposal**

To mitigate the high voltage and low speed of Flash, researchers propose **FeNAND**: replacing the charge-trap layer in a standard 3D NAND string with a ferroelectric layer.5

* **Physics:** This creates a 3D string of FeFETs.  
* **Advantage:** It retains the cost-effective, high-density manufacturing flow of 3D NAND (using the same etching and deposition tools) while enabling low-voltage (\<7V) operation.  
* **Challenge:** The cylindrical geometry exacerbates the depolarization field (as discussed in Section 3.1), as the field lines concentrate in the inner dielectric layers.

### **4.2 Vertical FeFET (V-FeFET): Manufacturing Physics**

The V-FeFET is the dedicated ferroelectric successor to 3D NAND, aiming to fix the mobility and voltage issues. However, fabricating high-quality ferroelectric transistors on the vertical walls of a deep trench introduces severe material science challenges.

#### **4.2.1 The Crystallization Conundrum**

HZO must be crystallized into the orthorhombic phase via a thermal anneal (typically 400-600°C).

* **Thermal Budget:** In a high-aspect-ratio stack, thermal gradients can occur.  
* **Stress Distribution:** The crystallization of HZO is strain-dependent. In a vertical stack, the mechanical stress exerted by the alternating wordline and insulating layers varies along the z-axis (depth).  
* **Non-Uniformity:** This leads to **Phase Non-Uniformity**. The top of the string might crystallize into the ferroelectric phase, while the bottom (under different stress/thermal conditions) might remain monoclinic (dielectric) or become tetragonal (anti-ferroelectric). This results in massive **Device-to-Device (D2D) variability** within a single string, making accurate analog weight assignment impossible without individual cell calibration.11

#### **4.2.2 Channel Material Innovation: The Shift to IGZO**

To solve the low mobility of Poly-Si and the interfacial issues of FeNAND, V-FeFETs are increasingly integrating **Oxide Semiconductors (OS)**, specifically **Indium Gallium Zinc Oxide (IGZO)**, as the channel material.12

* **Why IGZO?**  
  1. **Low Temperature:** IGZO can be deposited via Atomic Layer Deposition (ALD) at low temperatures (\<400°C), compatible with Back-End-of-Line (BEOL) processing.  
  2. **High Mobility:** ALD IGZO offers mobility ($\>30 cm^2/V\\cdot s$) superior to poly-Si and comparable to bulk silicon in thin films.  
  3. **Low Leakage:** IGZO is a wide-bandgap semiconductor with extremely low off-currents ($I\_{off}$), crucial for minimizing leakage power in massive idle arrays.  
  4. **Interface Physics:** Crucially, IGZO allows for the elimination of the $SiO\_2$ interfacial buffer layer. HZO can be deposited directly onto IGZO or with a minimal alumina buffer. This removal of the low-k $SiO\_2$ layer significantly reduces the depolarization field (by increasing the effective series capacitance), thereby improving the stability of analog weights.12

#### **4.2.3 The Interfacial Buffer Layer (IFBL) Conflict**

A central manufacturing paradox in 3D V-FeFETs is the **IFBL Conflict**.27

* **For Mobility:** A high-quality interface (like $SiO\_2$) is needed between the channel and the gate dielectric to minimize trap states and scattering, ensuring high mobility and reliability.  
* **For Retention:** An IFBL acts as a low-capacitance layer in series with the ferroelectric. This "capacitive divider" absorbs a large portion of the voltage drop and generates a strong depolarization field. To maximize retention, the IFBL should be eliminated.  
* **The Trade-off:** A thick IFBL gives good mobility but poor analog retention. No IFBL gives good retention but poor mobility and high interface trapping (which causes drift). The transition to IGZO channels is largely driven by the potential to resolve this conflict, as IGZO forms a reasonably good electrical interface with high-k dielectrics without needing a thick $SiO\_2$ layer.

### **4.3 Variability Sources in Vertical Scaling**

In addition to crystallization, 3D V-FeFETs suffer from geometric variability.

* **Grain Orientation:** A typical FeFET gate length in a vertical string might be 20-30nm. The grain size of HZO is often 10-20nm. This means a single cell might contain only 1-3 grains. The specific crystallographic orientation of these few grains relative to the electric field dominates the device performance.  
* **Percolation Paths:** Current in the channel does not flow uniformly; it follows the path of least resistance. In a multi-grain FeFET, the current will "percolate" through the regions controlled by grains that switch first. This **Percolation Physics** means that the threshold voltage is determined by the "weakest link" (easiest switching grain), leading to significant random telegraph noise and threshold variability.11

### **4.4 Table: 3D Architecture Comparison**

| Feature | 3D NAND Flash (Charge Trap) | 3D V-FeFET (Ferroelectric) | Hybrid FeNAND |
| :---- | :---- | :---- | :---- |
| **Storage Physics** | Charge Trapping in SiN (Continuous) | Domain Switching in HZO (Granular) | Domain Switching in HZO |
| **Channel Material** | Poly-Silicon (Grainy, Low $\\mu$) | **IGZO** / Oxide Semi (Uniform, High $\\mu$) | Poly-Silicon |
| **Programming Voltage** | High (\> 20 V) | Low (\< 5-7 V) | Medium (\~10 V) |
| **Vertical Scaling Limit** | Dielectric Breakdown / IR Drop | Depolarization / Crystallization Uniformity | Interface Quality |
| **Analog Stability** | Limited by Lateral Charge Migration | Limited by Depolarization Field | Limited by Depolarization |
| **Endurance** | $10^4 \- 10^5$ | $10^{10} \- 10^{12}$ | High |
| **Primary Mfg. Challenge** | High Aspect Ratio Etch / ONO scaling | **Uniform Crystallization** in deep trenches | Interface Integration |

## ---

**5\. System-Level Implications for Analog AI**

The physical properties of the memory device dictate the architecture of the AI accelerator. The industry is witnessing a divergence into **Current-Domain** and **Charge-Domain** computing to address the limitations of FeFETs.

### **5.1 Charge-Domain vs. Current-Domain Computing**

Traditional analog accelerators operate in the **Current Domain**: inputs are applied as voltages to the gates, conductance represents weight, and the output is the summation of currents ($I\_{sum} \= \\Sigma G\_{ij} V\_j$).

* **Flash Suitability:** Flash is excellent for this because its subthreshold $I-V$ characteristics can be tuned precisely.  
* **FeFET Limitation:** FeFETs suffer from $V\_{th}$ variability and drift. In the current domain, a small drift in $V\_{th}$ results in an exponential error in current (if operating subthreshold) or significant linear error (strong inversion).

To mitigate this, recent architectures utilize **Charge-Domain Computing** for FeFETs.13

* **1FeFET-1C Structure:** The FeFET is used not as a variable resistor, but as a non-volatile switch to gate a capacitor.  
* **Mechanism:** The computation relies on charge redistribution between capacitors. The FeFET controls *whether* charge flows, or *how much* charge flows based on a dynamic switching event.  
* **Advantage:** This method is less sensitive to the absolute static $V\_{th}$ of the FeFET. It leverages the robust ON/OFF ratio and the speed of the device rather than its precise intermediate conductance stability. Benchmarks indicate such macros can achieve **111.6 TOPS/W**, outperforming many RRAM and Flash equivalents.13

### **5.2 Geometric Ratio Encoding: Bypassing Material Instability**

A radical alternative approach ignores the variable physics of the memory cell entirely for the computation itself, using the memory only to configure the circuit topology. This is the **Geometric Ratio of Transistors** method.26

* **Physics:** The weight is encoded in the effective Width/Length ($W/L$) ratio of the computing transistors.  
* **Implementation:** Digital memory (SRAM or Flash) turns on a specific parallel combination of unit transistors. The effective conductance is determined by the geometry (which is fixed by lithography) rather than a stored charge or polarization state.  
* **Reliability:** Since the geometry does not drift with time or temperature, this method achieves record-breaking precision (RMSE \< 0.1%) and thermal stability (-78°C to 180°C), far surpassing intrinsic analog memory storage. However, it compromises density, as multiple transistors are needed to represent one weight.

### **5.3 Benchmarking: Energy and Density**

Based on IEDM/ISSCC 2024-2025 data 34:

* **Flash (Subthreshold):** Unbeatable for density ($\>100$ layers). Energy efficiency is moderate (10-100 TOPS/W) due to the overhead of peripheral circuits needed to manage high-voltage writing and temperature compensation. Best for **Inference** of massive models (LLMs).  
* **FeFET (V-FeFET):** High potential for energy efficiency (\>100 TOPS/W) due to low voltage. The write energy is femtojoules per bit, making it the only viable candidate for **On-Chip Training** or learning-at-the-edge where weights are updated frequently.  
* **Hybrid/Geometric:** Offers the highest precision (equivalent to 8-bit digital) but lower density. Ideal for mission-critical analog compute layers where error tolerance is low.

## ---

**6\. Conclusion**

The landscape of Analog AI weight encoding is defined by a trade-off between **kinetic speed** and **static precision**, governed fundamentally by the physics of the storage medium.  
**Flash Memory** remains the incumbent champion for inference. Its physics—governed by the electrostatics of trapped charge and the quantum mechanics of tunneling—allows for exquisite control via ISPP and highly efficient log-domain arithmetic in the subthreshold regime. The maturity of 3D NAND manufacturing provides an immediate pathway to high-density storage. However, the high voltages and limited endurance inherent to tunneling physics render it unsuitable for dynamic training workloads.  
**Ferroelectric FeFETs**, specifically 3D V-FeFETs with IGZO channels, represent the future of energy-efficient, dynamic AI. Their physics—governed by the nucleation and growth of domains—offers the speed and endurance required to break the training energy bottleneck. However, the technology is currently limited by the stochastic nature of polycrystalline switching (granularity), the thermodynamic instability of intermediate states (depolarization), and the immense challenges of fabricating uniform ferroelectric layers in high-aspect-ratio vertical trenches.  
For the immediate future, **Flash-based architectures** utilizing subthreshold physics will dominate high-precision edge inference. As material engineering solves the challenges of HZO crystallization and interface stability, **3D V-FeFETs** are poised to enable the next frontier: true on-chip learning and adaptation.  
---

This report synthesizes research from sources 9 through 17 provided in the research materials.

#### **Works cited**

1. Charge trap flash \- Wikipedia, accessed December 24, 2025, [https://en.wikipedia.org/wiki/Charge\_trap\_flash](https://en.wikipedia.org/wiki/Charge_trap_flash)  
2. US20150262675A1 \- Incremental step pulse programming (ISPP) scheme capable of determining a next starting pulse based on a current program-verify pulse for improving programming speed \- Google Patents, accessed December 24, 2025, [https://patents.google.com/patent/US20150262675A1/en](https://patents.google.com/patent/US20150262675A1/en)  
3. Analog In-Memory Subthreshold Deep Neural Network Accelerator, accessed December 24, 2025, [https://knowen-production.s3.amazonaws.com/uploads/attachment/file/5301/Fick-Analog-In-Memory-Subthreshold-Deep-Neural-Network-Accelerator.pdf](https://knowen-production.s3.amazonaws.com/uploads/attachment/file/5301/Fick-Analog-In-Memory-Subthreshold-Deep-Neural-Network-Accelerator.pdf)  
4. A new physics-based model for TANOS memories program/erase ..., accessed December 24, 2025, [https://www.researchgate.net/publication/224392480\_A\_new\_physics-based\_model\_for\_TANOS\_memories\_programerase](https://www.researchgate.net/publication/224392480_A_new_physics-based_model_for_TANOS_memories_programerase)  
5. Vertical NAND in a Ferroelectric-driven Paradigm Shift \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2512.15988v1](https://arxiv.org/html/2512.15988v1)  
6. Hafnium oxide-based ferroelectric field effect transistors: From materials and reliability to applications in storage-class memory and in-memory computing \- AIP Publishing, accessed December 24, 2025, [https://pubs.aip.org/aip/jap/article/138/1/010701/3351745/Hafnium-oxide-based-ferroelectric-field-effect](https://pubs.aip.org/aip/jap/article/138/1/010701/3351745/Hafnium-oxide-based-ferroelectric-field-effect)  
7. Polarization And Switching Dynamics Study Of Ferroelectric Hafnium Zirconium Oxide For FeRAM And FeFET Applications \- Purdue University Graduate School \- Figshare, accessed December 24, 2025, [https://hammer.purdue.edu/articles/thesis/Polarization\_And\_Switching\_Dynamics\_Study\_Of\_Ferroelectric\_Hafnium\_Zirconium\_Oxide\_For\_FeRAM\_And\_FeFET\_Applications/23514816](https://hammer.purdue.edu/articles/thesis/Polarization_And_Switching_Dynamics_Study_Of_Ferroelectric_Hafnium_Zirconium_Oxide_For_FeRAM_And_FeFET_Applications/23514816)  
8. Nucleation-Limited Switching Dynamics Model for Efficient ..., accessed December 24, 2025, [https://www.researchgate.net/publication/356860109\_Nucleation-Limited\_Switching\_Dynamics\_Model\_for\_Efficient\_Ferroelectrics\_Circuit\_Simulation](https://www.researchgate.net/publication/356860109_Nucleation-Limited_Switching_Dynamics_Model_for_Efficient_Ferroelectrics_Circuit_Simulation)  
9. Domain-Wall Mediated Polarization Switching in Ferroelectric AlScN: Strain Relief and Field-Dependent Dynamics \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2509.00705v1](https://arxiv.org/html/2509.00705v1)  
10. Variability and disturb sources in ferroelectric 3D ... \- UCL Discovery, accessed December 24, 2025, [https://discovery.ucl.ac.uk/10188322/1/Pesic\_Variability\_disturb\_sources\_in\_ferroelectric\_3D\_NANDs\_and\_comparison\_to\_Charge-Trap\_equivalents\_IEDM22.pdf](https://discovery.ucl.ac.uk/10188322/1/Pesic_Variability_disturb_sources_in_ferroelectric_3D_NANDs_and_comparison_to_Charge-Trap_equivalents_IEDM22.pdf)  
11. Variability in Planar FeFETs \- Channel Percolation Impact \- Lirias, accessed December 24, 2025, [https://lirias.kuleuven.be/retrieve/722840](https://lirias.kuleuven.be/retrieve/722840)  
12. (PDF) A 3D Vertical-Channel Ferroelectric/Anti-Ferroelectric FET with Indium Oxide, accessed December 24, 2025, [https://www.researchgate.net/publication/361446937\_A\_3D\_Vertical-Channel\_FerroelectricAnti-Ferroelectric\_FET\_with\_Indium\_Oxide](https://www.researchgate.net/publication/361446937_A_3D_Vertical-Channel_FerroelectricAnti-Ferroelectric_FET_with_Indium_Oxide)  
13. A High-Efficiency Charge-Domain Compute-in-Memory 1F1C Macro Using 2-bit FeFET Cells for DNN \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/iel8/6570653/10432974/10750057.pdf](https://ieeexplore.ieee.org/iel8/6570653/10432974/10750057.pdf)  
14. A Remedy to Compute-in-Memory with Dynamic Random Access Memory: 1FeFET-1C Technology for Neuro-Symbolic AI \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2410.15296v1](https://arxiv.org/html/2410.15296v1)  
15. Ferroelectric Phase Stabilization and Charge-Transport ... \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-6412/15/12/1396](https://www.mdpi.com/2079-6412/15/12/1396)  
16. Ferroelectrics Based on HfO 2 Film \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2079-9292/10/22/2759](https://www.mdpi.com/2079-9292/10/22/2759)  
17. Role of Si Doping in Reducing Coercive Fields for Ferroelectric ..., accessed December 24, 2025, [https://www.researchgate.net/publication/347335452\_Role\_of\_Si\_Doping\_in\_Reducing\_Coercive\_Fields\_for\_Ferroelectric\_Switching\_in\_Hf\_O\_2](https://www.researchgate.net/publication/347335452_Role_of_Si_Doping_in_Reducing_Coercive_Fields_for_Ferroelectric_Switching_in_Hf_O_2)  
18. Synaptic Characteristics of Atomic Layer-Deposited Ferroelectric Lanthanum-Doped HfO2 (La:HfO2) and TaN-Based Artificial Synapses \- PubMed, accessed December 24, 2025, [https://pubmed.ncbi.nlm.nih.gov/38041654/](https://pubmed.ncbi.nlm.nih.gov/38041654/)  
19. Investigating Ferroelectric Minor Loop Dynamics and History Effect--Part I: Device Characterization | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/343353967\_Investigating\_Ferroelectric\_Minor\_Loop\_Dynamics\_and\_History\_Effect--Part\_I\_Device\_Characterization](https://www.researchgate.net/publication/343353967_Investigating_Ferroelectric_Minor_Loop_Dynamics_and_History_Effect--Part_I_Device_Characterization)  
20. Ferroelectric Memory Based on Partial Polarization for Analog Weight Storage \- Curate ND, accessed December 24, 2025, [https://curate.nd.edu/articles/thesis/Ferroelectric\_Memory\_Based\_on\_Partial\_Polarization\_for\_Analog\_Weight\_Storage/24739677](https://curate.nd.edu/articles/thesis/Ferroelectric_Memory_Based_on_Partial_Polarization_for_Analog_Weight_Storage/24739677)  
21. TCAD Simulation of a SONOS Device with Silvaco's new FNONOS Model, accessed December 24, 2025, [https://silvaco.com/simulation-standard/tcad-simulation-of-a-sonos-device-with-silvacos-new-fnonos-model/](https://silvaco.com/simulation-standard/tcad-simulation-of-a-sonos-device-with-silvacos-new-fnonos-model/)  
22. TANOS Charge-Trapping Flash Memory Structures, accessed December 24, 2025, [https://repository.rit.edu/cgi/viewcontent.cgi?article=1019\&context=amec](https://repository.rit.edu/cgi/viewcontent.cgi?article=1019&context=amec)  
23. study of incremental step pulse programming (ISPP) and STI edge effect of BE-SONOS NAND Flash | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/4348287\_study\_of\_incremental\_step\_pulse\_programming\_ISPP\_and\_STI\_edge\_effect\_of\_BE-SONOS\_NAND\_Flash](https://www.researchgate.net/publication/4348287_study_of_incremental_step_pulse_programming_ISPP_and_STI_edge_effect_of_BE-SONOS_NAND_Flash)  
24. Subthreshold analog circuit design for large-scale, low-power, spiking neuromorphic systems | Stanford Digital Repository, accessed December 24, 2025, [https://purl.stanford.edu/wx067xs4423](https://purl.stanford.edu/wx067xs4423)  
25. Learning in Log-Domain: Subthreshold Analog AI Accelerator Based ..., accessed December 24, 2025, [https://arxiv.org/abs/2501.13181](https://arxiv.org/abs/2501.13181)  
26. Ultrahigh-precision analog computing using memory-switching ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12429057/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12429057/)  
27. Variability sources and reliability of 3D – FeFETs \- UCL Discovery, accessed December 24, 2025, [https://discovery.ucl.ac.uk/10188323/1/Pesic\_Variability\_sources\_and\_reliability\_of\_3D\_\_FeFETs\_IEDM21.pdf](https://discovery.ucl.ac.uk/10188323/1/Pesic_Variability_sources_and_reliability_of_3D__FeFETs_IEDM21.pdf)  
28. Reliability of HfO2-Based Ferroelectric FETs: A Critical Review of Current and Future Challenges | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/367965292\_Reliability\_of\_HfO2-Based\_Ferroelectric\_FETs\_A\_Critical\_Review\_of\_Current\_and\_Future\_Challenges](https://www.researchgate.net/publication/367965292_Reliability_of_HfO2-Based_Ferroelectric_FETs_A_Critical_Review_of_Current_and_Future_Challenges)  
29. (PDF) Ferroelectric HZO Thin Films for FEFETs: Crystal Structure‐Device Performance Relationship \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/398889771\_Ferroelectric\_HZO\_Thin\_Films\_for\_FEFETs\_Crystal\_Structure-Device\_Performance\_Relationship](https://www.researchgate.net/publication/398889771_Ferroelectric_HZO_Thin_Films_for_FEFETs_Crystal_Structure-Device_Performance_Relationship)  
30. 3D NAND | Applied Materials, accessed December 24, 2025, [https://www.appliedmaterials.com/us/en/semiconductor/markets-and-inflections/memory/3d-nand.html](https://www.appliedmaterials.com/us/en/semiconductor/markets-and-inflections/memory/3d-nand.html)  
31. Analog Computation in Ultra-High Density 3D FeNAND for TB-level ..., accessed December 24, 2025, [https://research.skhynix.com/blog/detail/analog-computation-in-ultra-high-density-3d-fenand-for-tb-level-hyperscale-ai-models](https://research.skhynix.com/blog/detail/analog-computation-in-ultra-high-density-3d-fenand-for-tb-level-hyperscale-ai-models)  
32. Hf0.4Zr0.6O2 Thickness-Dependent Transfer Characteristics of InxZn1–xOy Channel Ferroelectric FETs | The Journal of Physical Chemistry Letters \- ACS Publications, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acs.jpclett.4c02201](https://pubs.acs.org/doi/10.1021/acs.jpclett.4c02201)  
33. Variability Study of Ferroelectric Field-Effect Transistors towards 7nm Technology Node \- IEEE Xplore, accessed December 24, 2025, [https://ieeexplore.ieee.org/iel7/6245494/6423298/09499116.pdf](https://ieeexplore.ieee.org/iel7/6245494/6423298/09499116.pdf)  
34. Download Tip Sheet \- IEDM, accessed December 24, 2025, [https://www.ieee-iedm.org/s/Tip-Sheet-of-Technical-Highlights-for-IEDM-2025\_FINAL.docx](https://www.ieee-iedm.org/s/Tip-Sheet-of-Technical-Highlights-for-IEDM-2025_FINAL.docx)  
35. Energy Efficient Dual Designs of FeFET-Based Analog In-Memory Computing with Inherent Shift-Add Capability \- arXiv, accessed December 24, 2025, [https://arxiv.org/html/2410.19593v1](https://arxiv.org/html/2410.19593v1)  
36. ISSCC 2025 / SESSION 23 / AI-ACCELERATORS / 23.1 \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2503.00322](https://arxiv.org/pdf/2503.00322)