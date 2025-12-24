# Registry of Signal Encoding Options for Analog AI Computing

## Overview

This registry catalogs the full spectrum of signal encoding mechanisms used in analog AI and neuromorphic computing systems. Encodings are organized into three functional categories: **Input Encoding** (how digital activations enter the analog domain), **Weight Encoding** (how synaptic weights are physically stored), and **Computational/Output Encoding** (how information flows through and exits the system).

---

## 1. Input Encoding (Digital-to-Analog Interface)

These methods convert digital activation values into analog signals that drive memory arrays.

### 1.1 Pulse Amplitude Modulation (PAM)

| Attribute | Description |
|-----------|-------------|
| **Physical Domain** | Voltage magnitude |
| **Mechanism** | Input value encoded directly as voltage level applied to wordline. For k-bit precision, DAC generates 2^k distinct voltage levels. |
| **Computation** | I = V × G (single-cycle operation) |
| **Advantages** | Maximum throughput (single-cycle MVM), simple timing |
| **Challenges** | High SNR requirements, IR-drop sensitivity (voltage corruption over distance), device I-V nonlinearity sensitivity, limited voltage headroom in advanced nodes |
| **Best Suited For** | Flash-based architectures, high-precision cells, latency-critical applications |
| **Key Standards Needs** | DAC linearity specs, voltage headroom, IR-drop compensation schemes |

### 1.2 Pulse Width Modulation (PWM)

| Attribute | Description |
|-----------|-------------|
| **Physical Domain** | Time / Pulse duration |
| **Mechanism** | Fixed voltage applied for variable duration. T_on proportional to input value. Charge Q = I(V_read) × T_on accumulated on capacitor. |
| **Computation** | Integration over time window |
| **Advantages** | Inherent linearity (fixed operating point bypasses device nonlinearity), high noise immunity, robust to IR-drop (affects gain, not value) |
| **Challenges** | Latency penalty—exponential with bit precision (2^N cycles for N-bit), timing precision requirements |
| **Best Suited For** | ReRAM, PCM, non-ideal emerging memories |
| **Key Standards Needs** | Timing precision protocols, integration window definitions |

### 1.3 Bit-Serial / Hybrid Encoding

| Attribute | Description |
|-----------|-------------|
| **Physical Domain** | Time (sequential binary) |
| **Mechanism** | Inputs fed one bit at a time (LSB to MSB). Binary multiplication per bit, results shifted and added digitally. |
| **Computation** | N cycles for N-bit input (linear complexity) |
| **Advantages** | Reduces 2^N to N cycles, uses simple 1-bit DACs, maintains PWM linearity benefits |
| **Challenges** | Shift-and-add overhead in digital domain |
| **Best Suited For** | Balanced SRAM/NVM designs, moderate precision requirements |

### 1.4 Pulse Density Modulation (PDM) / Rate Coding

| Attribute | Description |
|-----------|-------------|
| **Physical Domain** | Spike frequency / firing rate |
| **Mechanism** | Magnitude encoded as frequency of fixed-width pulses. Count pulses over observation window. |
| **Computation** | Event-driven accumulation |
| **Advantages** | High noise immunity, mimics biological neural coding, ultra-low power for sparse events |
| **Challenges** | Poor information density, long observation windows needed for precision |
| **Best Suited For** | Spiking Neural Networks (SNNs), ultra-low-power event-driven applications |

### 1.5 Spatial Bit-Parallelism

| Attribute | Description |
|-----------|-------------|
| **Physical Domain** | Multiple parallel voltage lines |
| **Mechanism** | Each bit of input drives separate wordline simultaneously. N-bit input requires N wordlines with weighted devices. |
| **Computation** | Single-cycle with spatial expansion |
| **Advantages** | Maximum temporal speed, avoids latency penalty of PWM |
| **Challenges** | Increased wire count, more DACs, reduced array density |
| **Best Suited For** | Ultra-low latency requirements where area is available |

---

## 2. Weight Encoding (Physical State Storage)

These methods define how synaptic weights are physically instantiated in memory devices.

### 2.1 Resistive / Conductance Encoding

#### 2.1.1 Filamentary Conductance (ReRAM/RRAM)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Filament geometry / conductance |
| **Physical Mechanism** | Oxygen vacancy migration forms/ruptures conductive filament in metal-oxide (HfO₂, TaO₂). SET: vacancies cluster → filament grows. RESET: vacancies disperse → filament ruptures. |
| **Analog Continuum** | Intermediate resistance states via partial filament modulation |
| **Challenges** | Stochastic ion migration, cycle-to-cycle variability, Random Telegraph Noise (RTN), asymmetric SET/RESET |
| **Precision** | Typically 2-4 bits reliable per cell |
| **Efficiency** | 70-195 TOPS/W reported |

#### 2.1.2 Electrochemical Metallization (CBRAM)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Metal filament conductance |
| **Physical Mechanism** | Active metal anode (Ag, Cu) releases cations into electrolyte. Cations deposit at cathode forming metallic bridge. |
| **Analog Continuum** | Filament thickness/density modulation |
| **Challenges** | Similar stochasticity to ReRAM, dual-filament switching behaviors observed |

#### 2.1.3 Phase-Change Memory (PCM)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Crystalline-to-amorphous ratio |
| **Physical Mechanism** | Chalcogenide (GST) phase transition via Joule heating. Crystalline = low resistance, Amorphous = high resistance. Partial crystallization creates continuum. |
| **Analog Continuum** | Volume fraction of crystalline region |
| **Challenges** | **Conductance drift** (power-law: G(t) = G₀·t^(-ν)), abrupt amorphization, asymmetric SET/RESET |
| **Precision** | 4-5 bits typical, requires drift compensation |
| **Drift Mitigation** | Multi-device encoding, global drift compensation, noise-aware training |

### 2.2 Charge-Based Encoding

#### 2.2.1 Floating-Gate Flash

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Trapped electron count on floating gate |
| **Physical Mechanism** | Fowler-Nordheim tunneling or hot-carrier injection places/removes electrons. Charge shifts threshold voltage V_th. |
| **Analog Continuum** | Continuous V_th modulation |
| **Operation** | Typically subthreshold: I_ds ∝ exp(κ(V_gs - V_th)) |
| **Precision** | Up to 8 bits (256 levels) demonstrated |
| **Challenges** | Limited write endurance (10⁴-10⁵ cycles), high programming voltages, exponential I-V sensitivity to temperature |
| **Best Suited For** | Inference-only applications (write once, read many) |
| **Commercial Example** | Mythic M1076 (~80M weights per chip) |

#### 2.2.2 Charge-Trap (SONOS)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Charge trapped in silicon nitride layer |
| **Physical Mechanism** | Discrete trap sites in Si₃N₄ store charge. Better reliability and lower programming voltage than floating gate. |
| **Advantages** | Excellent symmetric write nonlinearities, demonstrated "ideal accuracy" training |

#### 2.2.3 Switched Capacitor (CMOS)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Capacitance ratio (C_in / C_int) |
| **Physical Mechanism** | Charge sampled on input capacitor (Q = C·V), transferred to integrator. Weight = capacitor ratio. |
| **Precision** | Up to 10 bits (MIM capacitors match precisely) |
| **Advantages** | Excellent linearity, no static power consumption |
| **Challenges** | Larger area per synapse, discrete-time operation |

#### 2.2.4 Gain Cells (2T1C/3T1C with IGZO)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Charge on storage capacitor |
| **Physical Mechanism** | DRAM-like storage with IGZO transistor for amplification. Ultra-low leakage enables seconds-to-minutes retention without refresh. |
| **Advantages** | Density between SRAM and DRAM, efficient volatile analog compute |

### 2.3 Ferroelectric Encoding

#### 2.3.1 Ferroelectric FET (FeFET)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Remnant polarization (P_r) |
| **Physical Mechanism** | Doped HfO₂ gate insulator exhibits ferroelectric hysteresis. Polarization shifts threshold voltage. Positive P_r → low V_th, Negative P_r → high V_th. |
| **Analog Continuum** | Partial domain switching via minor hysteresis loops |
| **Challenges** | Wake-up effect, fatigue, depolarization over time, dopant engineering critical |
| **Advantages** | CMOS-compatible, scalable, three-terminal control |

### 2.4 Spintronic / Magnetic Encoding

#### 2.4.1 Magnetic Tunnel Junction (MTJ) - Probabilistic

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Switching probability / magnetization state |
| **Physical Mechanism** | Superparamagnetic MTJ with energy barrier ~ k_B·T. Magnetization fluctuates between Parallel/Anti-Parallel states. |
| **Analog Representation** | Weight = probability P of being in P-state, tunable via bias current |
| **Advantages** | Native stochasticity for Boltzmann machines, probabilistic neural networks |
| **Challenges** | Not deterministic—statistical inference only |

#### 2.4.2 Domain Wall Racetracks

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Domain wall position along ferromagnetic wire |
| **Physical Mechanism** | Spin-Transfer Torque or Spin-Orbit Torque moves domain wall. Position encodes continuous analog weight. |
| **Advantages** | Deterministic analog, infinite endurance, radiation hard |

#### 2.4.3 Skyrmion-Based Synapses

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Number/presence of magnetic skyrmions |
| **Physical Mechanism** | Topologically protected spin textures behave like particles. Count encodes weight. |
| **Advantages** | Extremely low driving currents, high stability |

#### 2.4.4 Spin-Torque Oscillators (STOs) - Frequency Domain

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Oscillation frequency / phase |
| **Physical Mechanism** | DC current sustains steady-state magnetization precession. Coupled STOs synchronize frequencies. |
| **Analog Representation** | Weight encoded in frequency or phase relationships |
| **Application** | Oscillatory neural networks mimicking biological assemblies |

### 2.5 Photonic Encoding

#### 2.5.1 Mach-Zehnder Interferometer (MZI)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Optical phase (φ) |
| **Physical Mechanism** | Thermo-optic or electro-optic phase shifter creates relative delay between split beams. Interference determines transmission T(φ). |
| **Computation** | Matrix multiplication at speed of light via MZI meshes |
| **Advantages** | Massive bandwidth, low latency |
| **Challenges** | Thermal management, phase stability |

#### 2.5.2 Microring Resonators (MRR) + WDM

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Resonance frequency / coupling coefficient |
| **Physical Mechanism** | Ring resonance tuned relative to signal wavelength determines light coupling. WDM enables multiple weights per waveguide. |
| **Challenges** | Thermal crosstalk between adjacent rings |
| **Advantages** | High integration density |

#### 2.5.3 Optical Intensity/Amplitude

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Light intensity |
| **Physical Mechanism** | Amplitude modulators control optical power. Direct multiplication via intensity. |

### 2.6 Superconducting Encoding

#### 2.6.1 Flux Quantization / Josephson Junctions

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Magnetic flux quanta in superconducting loops |
| **Physical Mechanism** | Weights stored as quantized flux states. Single-photon detectors trigger Josephson junction threshold logic. |
| **Advantages** | Massive fan-out (thousands of connections), femtojoule per event, approaching biological efficiency |
| **Commercial Example** | NIST superconducting optoelectronic networks |

### 2.7 Subthreshold CMOS (Current-Mode)

| Attribute | Description |
|-----------|-------------|
| **State Variable** | Transistor threshold voltage / bias point |
| **Physical Mechanism** | Transistors operated in weak inversion where I_ds ∝ exp(V_gs). Exponential I-V matches biological ion channels. |
| **Advantages** | Picoampere currents, Hodgkin-Huxley model emulation, ultra-low power |
| **Challenges** | High sensitivity to process mismatch—exponential current variations from small V_th variations |

---

## 3. Architectural Encoding Strategies

These are system-level techniques that combine multiple devices or encodings.

### 3.1 Differential Pair Encoding

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Weight W = G⁺ − G⁻ using two physical devices |
| **Purpose** | Enables signed (positive/negative) weights from positive-only conductances |
| **Key Benefit** | Common-mode rejection of temperature, supply noise, global variations |
| **Status** | Industry standard for all major analog AI architectures |

### 3.2 Bit-Slicing / Significance Encoding

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Decompose high-precision weight into multiple low-precision slices (e.g., 8-bit → 2×4-bit MSP+LSP) |
| **Recombination** | W_total = F·(G_MSP⁺ - G_MSP⁻) + (G_LSP⁺ - G_LSP⁻) where F is significance factor |
| **Purpose** | Achieves 8-16 bit system precision from 2-4 bit devices |
| **Trade-off** | Area/energy cost vs. reliability; decouples system precision from device limitations |
| **Variants** | Unbalanced bit-slicing allocates more resources to MSP |

### 3.3 Multi-Level Cell (MLC)

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Single device stores multiple bits via discrete conductance levels |
| **Precision Target** | 16+ levels (4-bit) in single cell |
| **Challenge** | State margin shrinks with levels; RTN and drift cause level confusion |
| **Status** | Flash achieves 8-bit MLC; ReRAM/PCM typically limited to 2-3 bits reliable |

---

## 4. Output / Readout Encoding

### 4.1 Current-Domain (Direct Analog)

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Currents sum on bitlines via Kirchhoff's law |
| **Readout** | ADC converts accumulated current to digital |
| **Challenges** | ADC is primary bottleneck (50-85% of tile energy/area) |

### 4.2 Charge-Domain

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Charge redistribution on capacitors encodes result |
| **Advantages** | Better linearity than current-mode, capacitors match well |

### 4.3 Time-Domain (TDC-based)

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Result encoded as delay or pulse width. Time-to-Digital Converter (TDC) measures using delay lines or counters. |
| **Advantages** | TDCs improve with process scaling (faster transistors = better resolution), inherently digital-friendly |
| **Efficiency** | Superior to SAR ADCs at 4-6 bit precision |

### 4.4 Analog Shift-and-Add (Pre-ADC)

| Attribute | Description |
|-----------|-------------|
| **Mechanism** | Switched-capacitor networks combine MSP/LSP bit-slices before digitization |
| **Advantage** | Single ADC conversion for full-precision result instead of per-slice |

---

## 5. Temporal / Spike-Based Encoding

Used primarily in neuromorphic and spiking systems.

### 5.1 Rate Coding

| Attribute | Description |
|-----------|-------------|
| **Encoding** | Information in spike firing frequency |
| **Metric** | Spikes per unit time |

### 5.2 Temporal Coding

| Attribute | Description |
|-----------|-------------|
| **Encoding** | Information in precise spike timing or inter-spike intervals |
| **Advantages** | Higher information density than rate coding |

### 5.3 Population Coding

| Attribute | Description |
|-----------|-------------|
| **Encoding** | Information distributed across ensemble of neurons |

### 5.4 Rank-Order Coding

| Attribute | Description |
|-----------|-------------|
| **Encoding** | Information in relative ordering of spike arrivals |

---

## 6. Comparative Summary Matrix

| Encoding Type | Domain | Precision | Speed | Linearity | Noise Immunity | Maturity |
|--------------|--------|-----------|-------|-----------|----------------|----------|
| PAM Input | Voltage | High | Fast (1-cycle) | Device-dependent | Low | High |
| PWM Input | Time | High | Slow (2^N cycles) | Inherent | High | High |
| Bit-Serial Input | Time | High | Medium (N cycles) | Inherent | High | High |
| PDM/Rate Input | Frequency | Medium | Slow | High | Very High | Medium |
| ReRAM Weight | Conductance | 2-4 bit | - | Moderate | Moderate | Medium |
| PCM Weight | Phase | 4-5 bit | - | Moderate | Low (drift) | High |
| Flash Weight | Charge | 8 bit | - | High | High | Very High |
| FeFET Weight | Polarization | 3-4 bit | - | Moderate | Moderate | Low |
| MTJ Probabilistic | Probability | Stochastic | - | N/A | Intrinsic | Low |
| Photonic | Phase/Intensity | High | Ultra-fast | High | High | Low |
| Differential Pair | Architecture | +1-2 bits effective | N/A | Improved | Improved (CMRR) | Standard |
| Bit-Slicing | Architecture | System > Device | N/A | Improved | Improved | Standard |

---

## 7. Standards Alignment

| Standards Body | Scope | Relevant Encodings |
|----------------|-------|-------------------|
| JEDEC | Memory specs, commands, retention | Weight encoding (all NVM types) |
| IEEE P2415 | Energy-proportional hardware abstraction | Input/output encoding efficiency |
| IEEE P2851 | Functional safety data exchange | Reliability metrics across encodings |
| UCIe Consortium | Chiplet interfaces | Digital-analog interface standards |
| NeuroBench | Benchmarking metrics | SynOps for spike-based, Accuracy-per-Watt |

---

## References

Compiled from:
- "The Analog Manifold: Physical Encoding of Neural Parametrics in Next-Generation Neuromorphic Architectures"
- "Comparative Analysis of Analog Encoding Techniques in In-Memory Computing Architectures"  
- "Advanced Architectures for Analog AI Weight Encoding: Programming, Precision, and Reliability"
- "Proposal for Unified Signal and Metrology Standards for Analog AI Hardware"

---

*Document Version: 1.0*  
*Generated: December 2025*
