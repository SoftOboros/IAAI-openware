# **The Neurotrophic Structure: Plasticity and Adaptation in Analog AI Weight Encoding**

## **1\. Introduction: The Imperative for Structural Plasticity in Analog Computing**

The trajectory of artificial intelligence hardware is currently colliding with a fundamental barrier: the physical limits of static connectivity. In the dominant paradigm of deep learning, neural networks are conceptualized as fixed graphs—dense matrices of synaptic weights where the topology is predetermined and immutable, and learning is restricted to the adjustment of scalar values within this rigid lattice. While this approach has flourished on high-precision digital substrates like GPUs, it faces existential challenges when translated to the energy-efficient domain of Analog AI.  
Analog computing, which leverages the intrinsic physical properties of devices—such as Ohm’s law for multiplication and Kirchhoff’s current law for accumulation—promises orders-of-magnitude improvements in energy efficiency. However, this promise is compromised by the physical realities of the substrate: limited fan-in/fan-out capabilities, area constraints, leakage currents in large crossbar arrays, and the inevitability of device failure. A static analog topology is brittle; it cannot route around defects, it wastes power on redundant connections, and it scales quadratically ($O(N^2)$) in resource usage, making brain-scale emulation physically impossible with current integration densities.  
The "Neurotrophic Structure" proposes a paradigm shift inspired by the vertebrate brain's solution to these same constraints. Biological neural networks are not static; they are dynamic, adaptive substrates where the connectivity itself—the topology—is a fluid variable. Driven by neurotrophic factors like Brain-Derived Neurotrophic Factor (BDNF), biological systems engage in a continuous cycle of synaptic growth (synaptogenesis) and pruning. This structural plasticity allows the brain to maintain a sparse, efficient connectivity ($O(N)$) that dynamically reallocates metabolic resources to where they are most information-rich.  
This report provides a comprehensive analysis of neurotrophic structures in Analog AI. It explores the translation of biological mechanisms—such as competitive Hebbian learning and homeostatic scaling—into algorithmic frameworks like Deep Rewiring (DEEP R) and Rigged Lottery (RigL). It details the hardware implementations on platforms ranging from the digital SpiNNaker system to mixed-signal BrainScaleS-2 and emerging memristive crossbars. Furthermore, it examines the profound theoretical implications of "topology as memory," arguing that in the regime of low-precision analog devices, the structure of the network encodes more intelligence than the precise value of its weights.

### **1.1 The Physical Constraints of Fixed-Topology Analog Systems**

To understand the necessity of structural plasticity, one must first appreciate the limitations of fixed-topology analog cores. The standard building block of Analog AI is the crossbar array, where synaptic weights are stored as conductances in non-volatile memory (NVM) devices like Resistive RAM (RRAM) or Phase Change Memory (PCM). In a fully connected layer, an input vector is converted to voltages applied to rows, and the resulting currents are summed along columns.  
While elegant in theory, this architecture encounters severe scaling limits:

1. **Fan-in and Fan-out Limitations:** Unlike a software variable which can be read infinite times, a physical neuron has a limited drive capability. The current supplied by a presynaptic driver must charge the capacitance of the entire row. As the number of connections grows, the RC delay increases, degrading speed. Similarly, the postsynaptic neuron must sink the summed current from all active synapses. In a dense array, this accumulated current can saturate the analog-to-digital converters (ADCs) or the neuron's integration capacitor, forcing the use of aggressive (and precision-destroying) scaling factors.1  
2. **Parasitic Leakage and Sneak Paths:** In large, dense arrays, "sneak paths"—unintended current loops through unselected devices—degrade the read margin. Even with selector devices (1T1R configurations), the cumulative leakage from thousands of "off" state devices contributes to static power consumption, eroding the energy advantage of the analog approach.2  
3. **Defect Intolerance:** Nanoscale memristors suffer from high defect rates and variability. In a fixed topology, a "stuck-on" defect (a short circuit) in a critical weight position can render an entire neuron or column useless. A fixed architecture lacks the mechanism to "disconnect" this fault and reroute the signal through a healthy path.

Structural plasticity addresses these issues by fundamentally changing the scaling law. Instead of a dense matrix where every element is a physical device, neurotrophic architectures utilize sparse connectivity where resources are allocated dynamically. This allows the system to operate within the fan-in limits of the hardware, minimize leakage by physically disconnecting unused paths, and inherently tolerate defects by pruning broken links and growing new ones.

## ---

**2\. Biological Foundations: Neurotrophic Factors and Synaptic Competition**

The engineering of structural plasticity is not a de novo invention but a translation of biological principles. The mammalian brain serves as the ultimate proof-of-concept for high-capacity, low-energy computing, achieving this through a sophisticated interplay of chemical signaling and structural remodeling. The central protagonist in this process is the family of proteins known as neurotrophins, with Brain-Derived Neurotrophic Factor (BDNF) being the most critical for activity-dependent plasticity.

### **2.1 The BDNF/TrkB Signaling Pathway: A Resource Allocation Algorithm**

BDNF is not merely a "growth juice" for neurons; it is a highly specific, activity-dependent modulator of synaptic efficacy and structure. Its synthesis and secretion are triggered by neuronal activity (depolarization and calcium influx), creating a direct feedback loop between function and structure. BDNF operates through two distinct receptor systems that drive opposing effects, a mechanism that provides the fine-grained control necessary for network refinement.

#### **2.1.1 Mature BDNF and TrkB: The Potentiation Signal**

When mature BDNF (mBDNF) binds to the Tropomyosin receptor kinase B (TrkB) receptor, it initiates phosphorylation cascades (such as the MAPK/ERK and PI3K/Akt pathways) that result in synaptic strengthening and stabilization.

* **Structural Effect:** TrkB activation promotes actin polymerization in the cytoskeleton, leading to the enlargement of dendritic spines and the stabilization of the synaptic contact. This is the biological equivalent of increasing the bit-precision or conductance limit of a memristor.4  
* **Functional Effect:** It facilitates the recruitment of AMPA receptors to the postsynaptic membrane and enhances presynaptic neurotransmitter release. In hardware terms, this is analogous to increasing the gain of the synapse.  
* **Local Specificity:** Crucially, while BDNF is a diffusible molecule, its release and capture are spatially restricted to active synaptic sites. This "synaptic tagging" ensures that neurotrophic support is allocated only to connections that are causally involved in firing the postsynaptic neuron, fulfilling the Hebbian requirement of "cells that fire together, wire together".6

#### **2.1.2 Pro-BDNF and p75NTR: The Pruning Signal**

Biology balances growth with decay. BDNF is synthesized as a precursor protein, pro-BDNF. If not cleaved into mature BDNF, pro-BDNF binds to the p75 neurotrophin receptor (p75NTR). This binding activates signaling pathways (such as RhoA) that lead to synaptic depression (LTD), spine shrinkage, and eventually, synapse elimination.

* **The Yin-Yang of Plasticity:** This dual mechanism—mBDNF/TrkB promoting growth and pro-BDNF/p75NTR promoting pruning—creates a push-pull dynamic. High-frequency stimulation typically promotes the conversion of pro-BDNF to mBDNF (via extracellular proteases like tPA), favoring LTP. Low-frequency stimulation favors the accumulation of pro-BDNF, driving LTD and pruning. This biological logic is mirrored in neuromorphic algorithms that prune weights falling below a certain activity threshold.7

### **2.2 Synaptic Competition: The Winner-Take-All Dynamics of Wiring**

A defining feature of neurotrophic regulation is the principle of limited resources. A neuron does not have unlimited metabolic energy or protein synthesis capacity to maintain an infinite number of connections. This scarcity induces a fierce competition among synapses for the limited pool of neurotrophic factors available at the postsynaptic dendrite.

* **Mechanism of Competition:** When a subset of synapses drives the postsynaptic neuron to fire, they trigger the local release of BDNF. These active synapses capture the available BDNF, strengthening themselves. Neighboring synapses that were inactive or weakly active during the event are starved of neurotrophic support. Furthermore, the strengthening of the "winner" synapses can actively suppress the "losers" through heterosynaptic depression—a process where the potentiation of one input causes the depression of inactive inputs on the same dendrite.9  
* **Topological Consequence:** This competitive dynamic naturally leads to a sparse, polarized topology. Instead of a neuron maintaining weak connections to thousands of inputs (a "dense" state), synaptic competition forces it to select a small subset of highly correlated inputs and form strong connections with them, while pruning the rest. This results in the emergence of "feature detectives"—neurons that are highly selective for specific patterns.  
* **Relevance to Analog AI:** In hardware, this biological competition is mimicked by normalization constraints (e.g., L1/L2 weight regularization or hard constraints on the total synaptic weight vector). Implementing such "Winner-Take-All" (WTA) dynamics in analog circuits—using shared inhibitory lines or current limiters—prevents runaway potentiation and forces the network to learn efficient, sparse representations.11

### **2.3 Critical Periods and Inhibitory Maturation**

Neurotrophic factors also regulate the temporal window of plasticity, known as the "critical period." BDNF drives the maturation of inhibitory interneurons, specifically parvalbumin-positive (PV+) fast-spiking cells. As these inhibitory circuits mature, they tighten the control over excitability, effectively "closing" the window of massive structural plasticity and locking in the learned topology.

* **Hardware Parallel:** This concept of a "critical period" is valuable for Analog AI training schedules. A system can be designed to have high structural plasticity (high learning rates, aggressive rewiring) during the initial "developmental" phase to explore the topology space, followed by a "maturation" phase where the topology freezes and only fine-grained weight adjustments occur. This approach, often called "annealing" in optimization, stabilizes the network and protects learned memories from catastrophic forgetting.8

## ---

**3\. Algorithmic Frameworks for Dynamic Topology**

Translating biological plasticity into executable code for neuromorphic systems has led to the development of distinct algorithmic families. These algorithms—Structural Plasticity Modules (SPMs)—govern the birth and death of connections in artificial networks.

### **3.1 Deep Rewiring (DEEP R): Stochastic Optimization under Constraints**

**Deep Rewiring (DEEP R)** is arguably the most prominent algorithm designed explicitly for the constraints of neuromorphic hardware. Unlike pruning methods that sparsify a network *after* training (which requires training a massive dense model first), DEEP R maintains a strict sparsity constraint *during* the entire training process. This allows it to run on hardware with extremely limited memory, such as the SpiNNaker system.

#### **3.1.1 The Mathematical Engine of DEEP R**

DEEP R treats the connectivity of the network not as a fixed graph but as a realization of a stochastic process. The algorithm operates on a sparse weight matrix $W$ and an associated connectivity matrix $C$.

1. **Transmission:** The effective weight is $W\_{eff} \= W \\odot C$.  
2. **Regularization as Metabolic Cost:** The objective function includes an $L\_1$ regularization term ($\\lambda ||W||\_1$). This term acts as a constant pressure, pushing all weights toward zero—simulating the metabolic cost of maintaining a synapse.  
3. **The Pruning Rule:** If a weight $w\_{ij}$ crosses zero (or falls below a threshold $\\theta$) due to the gradient update or regularization pressure, the connection is severed: $c\_{ij} \\leftarrow 0$. This mimics the retraction of a spine that fails to transmit significant signals.  
4. **The Growth Rule:** To prevent the network from dissolving, the algorithm re-activates a fixed number of connections at every step to maintain the target sparsity level. New connections are initialized at random locations with zero weight: $c\_{mn} \\leftarrow 1, w\_{mn} \= 0$. These "candidate synapses" are then subject to gradient updates; if the gradient indicates they are useful, they grow (potentiate). If not, they drift back to zero and are pruned again in the next cycle.14

#### **3.1.2 Efficiency Gains on Neuromorphic Hardware**

The impact of DEEP R on hardware efficiency is profound. On the **SpiNNaker 2** prototype, DEEP R allowed the training of a deep neural network using only **36 KB** of local SRAM per core. A standard fully connected version of the same network would require over **1 MB** of memory, necessitating energy-intensive access to off-chip DRAM.

* **Memory Reduction:** The weight matrix footprint was reduced by **96%**.  
* **Energy Reduction:** By keeping the model within the local SRAM (which consumes far less energy per access than DRAM), the overall system energy consumption was reduced by **62%**.  
* **Hardware Acceleration:** SpiNNaker 2 includes dedicated hardware accelerators for random number generation (RNG) and exponential functions. These are critical for DEEP R, which requires frequent stochastic sampling to decide which connections to rewire. The presence of these accelerators sped up the plasticity updates by a factor of 2\.14

### **3.2 Sparse Evolutionary Training (SET) and RigL**

While DEEP R relies on stochastic sampling, other algorithms take a more heuristic approach to topology search.

#### **3.2.1 Sparse Evolutionary Training (SET)**

**SET** draws inspiration from network science and the properties of scale-free networks. It employs a simple magnitude-based pruning cycle:

* At the end of each training epoch, a fraction $\\zeta$ (e.g., 30%) of the weights with the smallest magnitudes are pruned (set to zero).  
* An equal number of new connections are grown randomly across the topology.  
* **Biological Analogy:** This mimics the "use it or lose it" principle. Strong weights represent consolidated memory; weak weights represent tentative connections that, if not reinforced, are eliminated to free up resources for new exploration.17

#### **3.2.2 RigL: Rigged Lottery and the Gradient Problem**

Random growth (as used in SET and DEEP R) is inefficient because the search space for possible connections is vast ($N^2$). **RigL (Rigged Lottery)** improves upon this by using gradient information to guide growth.

* **Mechanism:** RigL calculates the gradients not just for active weights, but also for "non-existent" weights (connections that are currently zero). It effectively asks: "If a connection existed here, how much would it reduce the error?"  
* **Directed Growth:** New connections are grown specifically at locations with the highest gradient magnitude. This "rigs" the lottery, ensuring that new synapses are placed where they are mathematically most needed.  
* **The Hardware Challenge:** Calculating gradients for non-existent connections is trivial in software (using dense matrix multiplication) but problematic in analog hardware. In a physical crossbar, a non-existent connection is an open circuit; you cannot easily measure a gradient across it without temporarily activating it or performing a digital simulation. Hybrid architectures (Analog Core \+ Digital DSP) attempt to solve this by having the digital core periodically "probe" the potential gradients using sparse backpropagation techniques.19

### **3.3 The Lottery Ticket Hypothesis in Spiking Networks**

The success of structural plasticity algorithms provides empirical support for the **Lottery Ticket Hypothesis (LTH)**: the idea that dense networks contain sparse subnetworks ("winning tickets") that, when trained in isolation, can match the accuracy of the full network.

* **SNN Adaptation:** Research has extended LTH to Spiking Neural Networks (SNNs). "winning tickets" in SNNs are subnetworks that not only have the right weights but also the right *temporal* connectivity to propagate spikes efficiently.  
* **Multi-Prize Tickets:** Recent work suggests the existence of "Multi-Prize Tickets"—sparse binary subnetworks that perform well *without any weight training at all*, simply by having the correct topology. This reinforces the concept that in low-precision regimes, topology is the primary carrier of information.21

## ---

**4\. Hardware Implementations: Digital and Mixed-Signal Architectures**

The implementation of structural plasticity varies significantly depending on whether the underlying substrate is digital, analog, or a hybrid of both.

### **4.1 SpiNNaker 2: The Digital Neurotrophic Engine**

**SpiNNaker (Spiking Neural Network Architecture)** represents the massive-parallel digital approach. The system is composed of millions of ARM cores connected via a lightweight packet-switching network.

* **Architecture:** Unlike GPUs that rely on synchronous global memory, SpiNNaker uses distributed local memory (SRAM) attached to each core. This mimics the brain's distributed memory storage.  
* **Plasticity Implementation:** Structural plasticity is implemented purely in software/firmware running on the ARM cores. The flexibility of the general-purpose processors allows researchers to define arbitrary rewiring rules (like DEEP R or SET).  
* **Optimization:** To support the heavy computational load of generating random numbers for stochastic rewiring, SpiNNaker 2 integrates True Random Number Generators (TRNG) and accelerators for exponential and logarithmic functions directly into the silicon. This allows it to perform the complex probability calculations required for synaptic sampling in real-time with minimal energy overhead.14

### **4.2 BrainScaleS-2: The Mixed-Signal Adaptor**

**BrainScaleS-2** takes a radically different approach, using physical analog circuits to emulate neurons and synapses.

* **Accelerated Emulation:** The analog circuits operate at 1000x real-time speed (compared to biology). This means a day of biological learning can be simulated in seconds.  
* **The Plasticity Processing Unit (PPU):** Because the analog core is too fast for standard computers to control in real-time, BrainScaleS-2 embeds a dedicated digital processor (the PPU) directly next to the analog array.  
* **Virtual Rewiring:** Physical synapses in a crossbar cannot be moved. However, BrainScaleS-2 implements structural plasticity through "address rewiring." The system maintains a mapping table that assigns physical synapse circuits to logical neuron addresses. When the algorithm decides to "move" a synapse, the PPU updates this mapping table. This allows the network to virtually restructure itself, effectively moving synaptic resources from low-correlation inputs to high-correlation inputs without changing the physical silicon layout.23

### **4.3 Intel Loihi and Sparse SNNs**

Intel's **Loihi** architecture is a digital asynchronous chip optimized for sparse spike communication.

* **Synaptic Plasticity:** Loihi's learning engine is microcode-programmable. It supports trace-based learning rules that can implement 3-factor plasticity (Pre-synaptic x Post-synaptic x Reward).  
* **Structural Capability:** While Loihi doesn't physically "grow" wires, its architecture supports the dynamic enabling/disabling of synapses in its adjacency lists. Algorithms running on the embedded x86 Lakemont cores can periodically groom these lists, removing unused connections to free up memory for new ones, effectively implementing a form of structural plasticity that optimizes the limited on-chip SRAM.25

### **4.4 Hybrid Architectures: Xpikeformer and Netcast**

Emerging architectures are blending analog and digital to overcome specific bottlenecks.

* **Xpikeformer:** This architecture is designed for Spiking Transformers. It uses analog in-memory computing (AIMC) for the dense feed-forward layers but employs a digital "Stochastic Spiking Attention" engine. Structural plasticity could be applied here to sparsify the attention matrix, dynamically pruning the "tokens" that the model attends to, thereby reducing the computational load on the analog array.26  
* **Netcast (Optical):** In optical neural networks, weights are encoded in the transmission of light. Structural plasticity in this domain might involve dynamically tuning Mach-Zehnder Interferometers (MZIs) to "switch off" light paths (zero transmission), effectively pruning connections at the speed of light.27

## ---

**5\. Analog Substrates and Material Intelligence**

The ultimate realization of the neurotrophic structure lies in materials that possess intrinsic plasticity—where the physics of the material *is* the learning algorithm.

### **5.1 Memristors: The Filamentary Synapse**

Memristors (Resistive Random Access Memory, or RRAM) are two-terminal devices that change resistance based on the history of applied voltage.

* **Filament Formation as Growth:** The switching mechanism typically involves the migration of oxygen vacancies (in oxides like $HfO\_2$) or metal ions (in conductive bridge RAM, CBRAM) to form a conductive filament. This process is physically analogous to the growth of a dendritic spine. A formed filament is a "connection"; a ruptured filament is a "pruned" synapse.  
* **Stochasticity and Exploration:** The formation of filaments is inherently stochastic. In digital memory, this variability is a nuisance. In neurotrophic AI, it is a feature. The noise in the switching process provides the "exploration" term needed for reinforcement learning and structural optimization, allowing the network to sample different connectivity configurations spontaneously.28

### **5.2 High-Entropy Oxides (HEOs): Tunable Plasticity**

To tame the unruly nature of standard memristors, materials scientists are developing **High-Entropy Oxides (HEOs)**. These materials are composed of five or more cations (e.g., $(Co,Cr,Fe,Mn,Ni)\_3O\_4$) mixed in a single crystal lattice.

* **Entropy Stabilization:** The high configurational entropy of the crystal lattice creates a complex energy landscape for ion migration. This stabilizes the intermediate conductance states, allowing for more precise analog weights.  
* **Tunable Dynamics:** By adjusting the ratio of cations, researchers can fine-tune the mobility of oxygen vacancies. This effectively allows engineers to "program" the learning rate and the forgetting rate (drift) of the material. A material with high drift could be used for short-term memory (which naturally decays/prunes), while a stable HEO could be used for long-term structural storage.29

### **5.3 Mitigating Conductance Drift via Structural Adaptation**

A major plague of analog AI is **conductance drift**—the tendency of programmed weights to relax over time, losing their information.

* **The Structural Solution:** Instead of fighting drift with expensive periodic reprogramming, neurotrophic architectures adapt to it. Drift is treated as a natural "forgetting" process. If a weight drifts below a useful threshold, the structural plasticity algorithm (monitoring error or correlation) interprets this as a signal to prune the connection. The resources are then re-allocated to a new, fresh connection.  
* **Compensation:** Research has demonstrated that simple calibration parameters stored in digital memory can compensate for the drift of entire crossbar columns, working in tandem with structural pruning to maintain accuracy over time.30

### **5.4 Material Intelligence: Growing Dendrites with PEDOT:PSS**

Perhaps the most futuristic approach involves **Organic Electrochemical Transistors (OECTs)** using conductive polymers like PEDOT:PSS.

* **Electropolymerization:** Researchers have demonstrated systems where connections are not just switched, but physically grown. By applying bipolar AC voltage pulses to electrodes in a monomer solution, conductive polymer chains (dendrites) grow through the solution to bridge the gap.  
* **Hebbian Growth:** The growth rate depends on the correlation of the pulses between the electrodes. High correlation leads to denser, thicker dendritic trees (stronger weights). Low correlation leads to thin, fragile connections that can be electrochemically dissolved (pruned).  
* **Implication:** This represents **Material Intelligence**. The topology is not stored in a digital address table; it is physically embodied in the polymer structure. This could lead to self-assembling, self-repairing circuits that evolve their physical form to match the data structure.31

### **5.5 3D Integration: The Path to Brain Scale**

The human brain is a volumetric computer. Current chips are 2D surfaces.

* **Vertical RRAM (VRRAM):** To approach biological density, structural plasticity is moving to 3D. Vertical RRAM technology (adapted from 3D NAND flash) allows for the stacking of synapse layers.  
* **Sparse Utilization:** While the physical 3D structure is fixed during fabrication, the density is so high ($10^{12}$ devices/wafer) that the algorithm can treat the block as a "tabula rasa." It can sparsely activate paths through this 3D lattice, effectively "carving" a functional circuit out of the massive block of available memory, much like a sculptor carves a statue from stone.32

## ---

**6\. Theoretical Implications: Topology as Memory**

The shift from optimizing weights to optimizing topology necessitates a re-evaluation of information storage in neural networks.

### **6.1 The Precision-Sparsity Trade-off**

There is a fundamental information-theoretic trade-off between the precision of synaptic weights and the sparsity of the topology.

* **Dense/High-Precision:** Traditional deep learning stores information in the precise values of a dense matrix (e.g., 32-bit floats).  
* **Sparse/Low-Precision:** Neurotrophic AI stores information in the *presence or absence* of connections (topology) and uses very low precision for the weights (e.g., 1-bit or 2-bit).  
* **Comparative Analysis:** Recent studies 22 have shown that sparse binary networks (where weights are $\\pm 1$ and topology is optimized) can match the accuracy of dense full-precision networks on complex tasks. This suggests that the *structure* of the network encodes more task-relevant information than the fine-grained *value* of the weights.  
* **The "3PXNet" Example:** The 3PXNet architecture combines binarization with structured pruning, achieving 38x model compression and 3x runtime reduction on microcontrollers with negligible accuracy loss. This proves that structural optimization can compensate for extreme quantization.33

### **6.2 Information Storage Capacity**

How much information can a synapse store?

* **Biological Estimates:** A biological synapse is estimated to store about 4.7 bits of information (approx. 26 distinguishable states).34  
* **Topological Information:** In a sparse network, the index of a connection (knowing *who* is connected to *whom*) carries significant information. In a network of $N$ neurons with $K$ connections, the number of possible topologies is $\\binom{N^2}{K}$. This combinatorial explosion allows sparse networks to possess a massive "potential" memory capacity. By dynamically selecting the optimal topology, the network maximizes its storage efficiency per bit of hardware.35  
* **Thermodynamic Perspective:** Pruning can be viewed as a thermodynamic process of minimizing free energy. The network evolves toward a configuration that minimizes the error (energy) while maximizing the entropy of the inactive states (sparsity). This "thermodynamic annealing" prevents overfitting and improves generalization.37

## ---

**7\. Future Directions: Autonomous Edge Adaptation and Repair**

The immediate future of neurotrophic structures lies in applications where energy is scarce and maintenance is impossible.

### **7.1 Autonomous Edge Learning**

For edge devices (e.g., nanosatellites, implantable biosensors, arctic buoys), communicating with the cloud for retraining is prohibitively expensive.

* **The Scenario:** A vision system trained in a factory is deployed to a forest. The input statistics change drastically (domain shift). A fixed network would fail.  
* **The Neurotrophic Solution:** A neurotrophic system would detect the surge in prediction error (homeostatic stress). This would trigger a "critical period"—a release of virtual neurotrophins—allowing the network to prune the now-useless "factory" features and grow new "forest" features. The device adapts its physical topology to the new environment autonomously.38

### **7.2 Self-Healing and Fault Tolerance**

Analog hardware degrades. Radiation in space or simple aging can destroy individual memristors.

* **Component Fusion:** Algorithms like **Component Fusion** use graph theory to repair fragmented networks. If a cluster of neurons becomes disconnected due to device failure, the algorithm identifies the optimal set of new connections needed to re-integrate them into the giant component, restoring function with minimal resource expenditure.40  
* **Regeneration:** In material-based systems (like PEDOT:PSS), a damaged connection could theoretically be "healed" by re-applying the electropolymerization voltage, growing a new polymer bridge to replace the broken one.

### **7.3 Conclusion: From Programming to Cultivation**

The Neurotrophic Structure represents a maturation of neuromorphic engineering. It moves beyond the naive imitation of neural *activity* (spiking) to the emulation of neural *construction* (plasticity). By integrating the biological principles of BDNF signaling, synaptic competition, and homeostatic scaling with the engineering capabilities of DEEP R, memristors, and digital accelerators, we are creating a new class of hardware.  
These are not just processors; they are substrates that *grow*. They solve the scalability crisis of analog AI by replacing quadratic waste with linear efficiency. They turn the noise and variability of the physical world from a bug into a feature. And most importantly, they offer a path to AI that is not rigidly programmed for a single task, but is cultivated to adapt, survive, and evolve in the unpredictable complexity of the real world.  
**Table 1: The Evolution of Weight Encoding**

| Feature | Standard Deep Learning | Static Analog AI | Neurotrophic Analog AI |
| :---- | :---- | :---- | :---- |
| **Weight Precision** | High (32-bit / 16-bit Float) | Medium (4-8 bit Fixed) | **Low (1-2 bit / Binary)** |
| **Topology** | Dense / Fixed ($O(N^2)$) | Dense / Fixed | **Sparse / Dynamic ($O(N)$)** |
| **Learning Mechanism** | Global Backpropagation | In-situ SGD / STDP | **Deep Rewiring / RigL / Material Growth** |
| **Information Carrier** | Weight Value | Weight Value | **Network Structure (Topology)** |
| **Fault Tolerance** | Low (Software dependent) | Low (Hardware vulnerable) | **High (Self-repairing)** |
| **Energy Efficiency** | Low | Medium (Leakage issues) | **High (Active sparsity)** |

The data is clear: the future of efficient AI is not dense and precise; it is sparse, structural, and plastic.  
References:

1

#### **Works cited**

1. (PDF) Structural plasticity on an accelerated analog neuromorphic hardware system, accessed December 24, 2025, [https://www.researchgate.net/publication/346259786\_Structural\_plasticity\_on\_an\_accelerated\_analog\_neuromorphic\_hardware\_system](https://www.researchgate.net/publication/346259786_Structural_plasticity_on_an_accelerated_analog_neuromorphic_hardware_system)  
2. Hardware-efficient photonic tensor core: accelerating deep neural networks with structured compression \- Optica Publishing Group, accessed December 24, 2025, [https://opg.optica.org/optica/fulltext.cfm?uri=optica-12-7-1079](https://opg.optica.org/optica/fulltext.cfm?uri=optica-12-7-1079)  
3. SPIKA: an energy-efficient time-domain hybrid CMOS-RRAM compute-in-memory macro, accessed December 24, 2025, [https://www.frontiersin.org/journals/electronics/articles/10.3389/felec.2025.1567562/full](https://www.frontiersin.org/journals/electronics/articles/10.3389/felec.2025.1567562/full)  
4. Brain-derived neurotrophic factor is a regulator of synaptic transmission in the adult visual thalamus \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC9662800/](https://pmc.ncbi.nlm.nih.gov/articles/PMC9662800/)  
5. Features of Neural Network Formation and Their Functions in Primary Hippocampal Cultures in the Context of Chronic TrkB Receptor System Influence \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC6335358/](https://pmc.ncbi.nlm.nih.gov/articles/PMC6335358/)  
6. BDNF and Activity-Dependent Synaptic Modulation \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC5479144/](https://pmc.ncbi.nlm.nih.gov/articles/PMC5479144/)  
7. Brain-Derived Neurotrophic Factor and the Development of Structural Neuronal Connectivity \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC2893579/](https://pmc.ncbi.nlm.nih.gov/articles/PMC2893579/)  
8. Mechanism of BDNF Modulation in GABAergic Synaptic Transmission in Healthy and Disease Brains \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/cellular-neuroscience/articles/10.3389/fncel.2018.00273/full](https://www.frontiersin.org/journals/cellular-neuroscience/articles/10.3389/fncel.2018.00273/full)  
9. Competition on presynaptic resources enhances the discrimination of interfering memories | PNAS Nexus | Oxford Academic, accessed December 24, 2025, [https://academic.oup.com/pnasnexus/article/2/6/pgad161/7162092](https://academic.oup.com/pnasnexus/article/2/6/pgad161/7162092)  
10. Synaptic competition in structural plasticity and cognitive function | Philosophical Transactions of the Royal Society B \- Journals, accessed December 24, 2025, [https://royalsocietypublishing.org/rstb/article/369/1633/20130157/22509/Synaptic-competition-in-structural-plasticity-and](https://royalsocietypublishing.org/rstb/article/369/1633/20130157/22509/Synaptic-competition-in-structural-plasticity-and)  
11. Synapse-type-specific competitive Hebbian learning forms ... \- PNAS, accessed December 24, 2025, [https://www.pnas.org/doi/10.1073/pnas.2305326121](https://www.pnas.org/doi/10.1073/pnas.2305326121)  
12. An adaptable neuromorphic model of orientation selectivity based on floating gate dynamics \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC3980111/](https://pmc.ncbi.nlm.nih.gov/articles/PMC3980111/)  
13. Brain-Derived Neurotrophic Factor Regulates the Maturation of Layer 4 Fast-Spiking Cells after the Second Postnatal Week in the Developing Barrel Cortex | Journal of Neuroscience, accessed December 24, 2025, [https://www.jneurosci.org/content/27/9/2241](https://www.jneurosci.org/content/27/9/2241)  
14. Memory-Efficient Deep Learning on a SpiNNaker 2 ... \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2018.00840/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2018.00840/full)  
15. Memory-Efficient Deep Learning on a SpiNNaker 2 Prototype \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC6250847/](https://pmc.ncbi.nlm.nih.gov/articles/PMC6250847/)  
16. Efficient Reward-Based Structural Plasticity on a SpiNNaker 2 Prototype \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/331916326\_Efficient\_Reward-Based\_Structural\_Plasticity\_on\_a\_SpiNNaker\_2\_Prototype](https://www.researchgate.net/publication/331916326_Efficient_Reward-Based_Structural_Plasticity_on_a_SpiNNaker_2_Prototype)  
17. Sparse Networks from Scratch: Faster Training without Losing Performance \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/334388834\_Sparse\_Networks\_from\_Scratch\_Faster\_Training\_without\_Losing\_Performance](https://www.researchgate.net/publication/334388834_Sparse_Networks_from_Scratch_Faster_Training_without_Losing_Performance)  
18. Pruning and Sparsification of Neural Networks \- inovex GmbH, accessed December 24, 2025, [https://www.inovex.de/de/blog/neural-networks-pruning-sparsification/](https://www.inovex.de/de/blog/neural-networks-pruning-sparsification/)  
19. Neurogenesis Dynamics-inspired Spiking Neural Network Training Acceleration, accessed December 24, 2025, [https://par.nsf.gov/servlets/purl/10462378](https://par.nsf.gov/servlets/purl/10462378)  
20. Multilayer perceptron ensembles in a truly sparse training context, accessed December 24, 2025, [https://d-nb.info/1375911899/34](https://d-nb.info/1375911899/34)  
21. Exploring Lottery Ticket Hypothesis in Spiking Neural Networks \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/364649507\_Exploring\_Lottery\_Ticket\_Hypothesis\_in\_Spiking\_Neural\_Networks](https://www.researchgate.net/publication/364649507_Exploring_Lottery_Ticket_Hypothesis_in_Spiking_Neural_Networks)  
22. Multi-Prize Lottery Ticket Hypothesis \- arXiv, accessed December 24, 2025, [https://arxiv.org/pdf/2103.09377](https://arxiv.org/pdf/2103.09377)  
23. Structural plasticity on an accelerated analog neuromorphic hardware system \- Department of Physiology, accessed December 24, 2025, [https://physiologie.unibe.ch/PublicationPDF/Billaudelle2021Structural.pdf](https://physiologie.unibe.ch/PublicationPDF/Billaudelle2021Structural.pdf)  
24. Structural plasticity on an accelerated analog neuromorphic hardware system \- BORIS Portal, accessed December 24, 2025, [https://boris-portal.unibe.ch/bitstreams/1bc56439-1cd6-49e5-9912-1443a8f9cf81/download](https://boris-portal.unibe.ch/bitstreams/1bc56439-1cd6-49e5-9912-1443a8f9cf81/download)  
25. robust and real-time movement encoding in spiking neural networks using neuromorphic hardware \- eDiss, accessed December 24, 2025, [https://ediss.uni-goettingen.de/bitstream/handle/11858/13912/Thesis\_2021-11-25\_Carlo-Michaelis.pdf?sequence=1\&isAllowed=y](https://ediss.uni-goettingen.de/bitstream/handle/11858/13912/Thesis_2021-11-25_Carlo-Michaelis.pdf?sequence=1&isAllowed=y)  
26. Xpikeformer: Hybrid Analog-Digital Hardware Acceleration for Spiking Transformers | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/390614962\_Xpikeformer\_Hybrid\_Analog-Digital\_Hardware\_Acceleration\_for\_Spiking\_Transformers](https://www.researchgate.net/publication/390614962_Xpikeformer_Hybrid_Analog-Digital_Hardware_Acceleration_for_Spiking_Transformers)  
27. Netcast: Low-Power Edge Computing With WDM-Defined Optical Neural Networks \- People, accessed December 24, 2025, [https://people.csail.mit.edu/zhizhenzhong/papers/2024\_JLT\_Netcast.pdf](https://people.csail.mit.edu/zhizhenzhong/papers/2024_JLT_Netcast.pdf)  
28. Memristive Ion Dynamics to Enable Biorealistic Computing | Chemical Reviews, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acs.chemrev.4c00587](https://pubs.acs.org/doi/10.1021/acs.chemrev.4c00587)  
29. High-Entropy Oxide Memristors for Neuromorphic Computing: From Material Engineering to Functional Integration \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12378895/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12378895/)  
30. Reliability of Memristive Devices for High-Performance Neuromorphic Computing: (Invited Paper) | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/370804668\_Reliability\_of\_Memristive\_Devices\_for\_High-Performance\_Neuromorphic\_Computing\_Invited\_Paper](https://www.researchgate.net/publication/370804668_Reliability_of_Memristive_Devices_for_High-Performance_Neuromorphic_Computing_Invited_Paper)  
31. Structural plasticity for neuromorphic networks with ..., accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC10709651/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10709651/)  
32. Wafer-Scale Integration of Analog Neural Networks | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/4375708\_Wafer-Scale\_Integration\_of\_Analog\_Neural\_Networks](https://www.researchgate.net/publication/4375708_Wafer-Scale_Integration_of_Analog_Neural_Networks)  
33. 3PXNet: Pruned-Permuted-Packed XNOR Networks for Edge Machine Learning \- UCLA NanoCAD Lab, accessed December 24, 2025, [https://nanocad.ee.ucla.edu/wp-content/papercite-data/pdf/j63.pdf](https://nanocad.ee.ucla.edu/wp-content/papercite-data/pdf/j63.pdf)  
34. An In-Depth Look at Google's Tensor Processing Unit Architecture | Hacker News, accessed December 24, 2025, [https://news.ycombinator.com/item?id=14043059](https://news.ycombinator.com/item?id=14043059)  
35. Maximizing theoretical and practical storage capacity in single-layer feedforward neural networks \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12414968/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12414968/)  
36. Selective connectivity enhances storage capacity in attractor models of memory function \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/systems-neuroscience/articles/10.3389/fnsys.2022.983147/full](https://www.frontiersin.org/journals/systems-neuroscience/articles/10.3389/fnsys.2022.983147/full)  
37. PROBABILISTIC NEURAL PRUNING VIA SPARSITY EVOLUTIONARY FOKKER-PLANCK-KOLMOGOROV EQUATION \- ICLR Proceedings, accessed December 24, 2025, [https://proceedings.iclr.cc/paper\_files/paper/2025/file/b287f84b5354e1130c51b24d86e98be9-Paper-Conference.pdf](https://proceedings.iclr.cc/paper_files/paper/2025/file/b287f84b5354e1130c51b24d86e98be9-Paper-Conference.pdf)  
38. Structural Plasticity Module: Dynamic Neural Networks \- Emergent Mind, accessed December 24, 2025, [https://www.emergentmind.com/topics/structural-plasticity-module-spm](https://www.emergentmind.com/topics/structural-plasticity-module-spm)  
39. The Neuromorphic Revolution: Brain-Inspired Architectures for Energy-Efficient and Sustainable AI Artificial Intelligence | Uplatz Blog, accessed December 24, 2025, [https://uplatz.com/blog/the-neuromorphic-revolution-brain-inspired-architectures-for-energy-efficient-and-sustainable-artificial-intelligence/](https://uplatz.com/blog/the-neuromorphic-revolution-brain-inspired-architectures-for-energy-efficient-and-sustainable-artificial-intelligence/)  
40. (PDF) Optimal Neural Network Repair: A Principled Approach to Achieving the Theoretical Limits of Structural Restoration \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/396850397\_Optimal\_Neural\_Network\_Repair\_A\_Principled\_Approach\_to\_Achieving\_the\_Theoretical\_Limits\_of\_Structural\_Restoration](https://www.researchgate.net/publication/396850397_Optimal_Neural_Network_Repair_A_Principled_Approach_to_Achieving_the_Theoretical_Limits_of_Structural_Restoration)  
41. Integrating programmable plasticity in experiment descriptions for analog neuromorphic hardware | CoLab, accessed December 24, 2025, [https://colab.ws/articles/10.1109%2Fnice65350.2025.11065886](https://colab.ws/articles/10.1109%2Fnice65350.2025.11065886)  
42. The interplay between homeostatic synaptic scaling and homeostatic structural plasticity maintains the robust firing rate of neural networks \- PMC \- PubMed Central, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12227201/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12227201/)  
43. The interplay between homeostatic synaptic scaling and homeostatic structural plasticity maintains the robust firing rate of neural networks | bioRxiv, accessed December 24, 2025, [https://www.biorxiv.org/content/10.1101/2023.03.09.531681v3.full-text](https://www.biorxiv.org/content/10.1101/2023.03.09.531681v3.full-text)  
44. Attention via Synaptic Plasticity is All You Need: A Biologically Inspired Spiking Neuromorphic Transformer | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/397740006\_Attention\_via\_Synaptic\_Plasticity\_is\_All\_You\_Need\_A\_Biologically\_Inspired\_Spiking\_Neuromorphic\_Transformer](https://www.researchgate.net/publication/397740006_Attention_via_Synaptic_Plasticity_is_All_You_Need_A_Biologically_Inspired_Spiking_Neuromorphic_Transformer)  
45. Neural signals encoded by specific frequencies of closed-loop electrical stimulation structurally and functionally reconstruct spinal sensorimotor circuits after spinal cord injury | bioRxiv, accessed December 24, 2025, [https://www.biorxiv.org/content/10.1101/2022.05.10.491346v1.full-text](https://www.biorxiv.org/content/10.1101/2022.05.10.491346v1.full-text)  
46. A DENDRITIC-INSPIRED NETWORK SCIENCE GENER- ATIVE MODEL FOR TOPOLOGICAL INITIALIZATION OF CONNECTIVITY IN SPARSE ARTIFICIAL NEUR \- OpenReview, accessed December 24, 2025, [https://openreview.net/pdf/6609ee187679d12037c9e7c4de91b54d14690898.pdf](https://openreview.net/pdf/6609ee187679d12037c9e7c4de91b54d14690898.pdf)  
47. The information theory of developmental pruning: Optimizing global network architectures using local synaptic rules | PLOS Computational Biology \- Research journals, accessed December 24, 2025, [https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1009458](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1009458)  
48. Constraints of Metabolic Energy on the Number of Synaptic Connections of Neurons and the Density of Neuronal Networks \- Frontiers, accessed December 24, 2025, [https://www.frontiersin.org/journals/computational-neuroscience/articles/10.3389/fncom.2018.00091/full](https://www.frontiersin.org/journals/computational-neuroscience/articles/10.3389/fncom.2018.00091/full)  
49. Relevance of network topology for the dynamics of biological neuronal networks \- bioRxiv, accessed December 24, 2025, [https://www.biorxiv.org/content/10.1101/2021.02.19.431963.full](https://www.biorxiv.org/content/10.1101/2021.02.19.431963.full)  
50. Reinforcing Synaptic Plasticity of Defect-Tolerant States in Alloyed 2D Artificial Transistors, accessed December 24, 2025, [https://pubs.acs.org/doi/10.1021/acsami.3c07578](https://pubs.acs.org/doi/10.1021/acsami.3c07578)  
51. Brain-Derived Neurotrophic Factor, Nociception, and Pain \- MDPI, accessed December 24, 2025, [https://www.mdpi.com/2218-273X/14/5/539](https://www.mdpi.com/2218-273X/14/5/539)  
52. CoDG-ReRAM: An Algorithm-Hardware Co-design to Accelerate Semi-Structured GNNs on ReRAM | Request PDF \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/366431994\_CoDG-ReRAM\_An\_Algorithm-Hardware\_Co-design\_to\_Accelerate\_Semi-Structured\_GNNs\_on\_ReRAM](https://www.researchgate.net/publication/366431994_CoDG-ReRAM_An_Algorithm-Hardware_Co-design_to_Accelerate_Semi-Structured_GNNs_on_ReRAM)  
53. DenseShift: Towards Accurate and Efficient Low-Bit Power-of-Two Quantization \- CVF Open Access, accessed December 24, 2025, [https://openaccess.thecvf.com/content/ICCV2023/papers/Li\_DenseShift\_Towards\_Accurate\_and\_Efficient\_Low-Bit\_Power-of-Two\_Quantization\_ICCV\_2023\_paper.pdf](https://openaccess.thecvf.com/content/ICCV2023/papers/Li_DenseShift_Towards_Accurate_and_Efficient_Low-Bit_Power-of-Two_Quantization_ICCV_2023_paper.pdf)  
54. Adaptive Neuromorphic Circuit for Stereoscopic Disparity Using Ocular Dominance Map \- PMC \- NIH, accessed December 24, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC4868909/](https://pmc.ncbi.nlm.nih.gov/articles/PMC4868909/)  
55. Competition between recently potentiated synaptic inputs reveals a winner-take-all phase of synaptic tagging and capture \- ResearchGate, accessed December 24, 2025, [https://www.researchgate.net/publication/264501330\_Competition\_between\_recently\_potentiated\_synaptic\_inputs\_reveals\_a\_winner-take-all\_phase\_of\_synaptic\_tagging\_and\_capture](https://www.researchgate.net/publication/264501330_Competition_between_recently_potentiated_synaptic_inputs_reveals_a_winner-take-all_phase_of_synaptic_tagging_and_capture)  
56. Top 10 Neuromorphic Computing Engineer Interview Questions, accessed December 24, 2025, [https://www.interviews.chat/questions/neuromorphic-computing-engineer](https://www.interviews.chat/questions/neuromorphic-computing-engineer)