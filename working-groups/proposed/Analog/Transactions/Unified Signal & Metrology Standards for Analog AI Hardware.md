# **Unified Signal & Metrology Standards for Analog AI Hardware**

**Consensus Draft v2.0 | Incorporating Broadened Scope & Statistical Safety Models**

## **1.0 Executive Summary: The "Layered" Standardization Strategy**

To resolve the tension between fostering innovation and ensuring interoperability, this standard adopts a **Two-Layer Architecture**:

1. **Layer 1 (The Descriptive Standard):** A mandatory metrology framework that standardizes *how* device characteristics (noise, drift, variance) are measured and reported. This ensures all devices—regardless of physics—speak the same language of reliability.  
2. **Layer 2 (The Prescriptive Standard):** A set of optional "Interop Profiles" (e.g., "UCIe-Compatible," "Automotive Grade") that mandate specific encodings for commercial integration.

This approach creates a **Compliance Ladder**, allowing novel research devices to be "Standard Compliant" (Level 1\) without being forced into a rigid commercial mold (Level 2).

---

## **2.0 Layer 1: The "Capability Declaration" Standard (Mandatory)**

This layer standardizes the *description* of the device, not the device itself. All compliant hardware must provide a machine-readable **Capability Declaration** (JSON/YAML) derived from standardized test protocols.

### **2.1 The "Noise-Induced Failure" Metric (MTBF)**

Recognizing that analog failure is probabilistic rather than binary, we replace simple "Endurance" metrics with **Mean Operations Between Failure (MOBF)** derived from the Noise Floor.

* **The Principle:** An "Operational Failure" occurs when instantaneous noise ($V\_{noise}$) pushes a read result across a decision boundary defined by the target precision.  
* **The Standard Metric:** Devices must report the **Noise Power Spectral Density (PSD)** and the **Gaussian Tail Probability** ($P(x \> \\sigma)$) for their read mechanism.  
* The Integrator's Formula: This allows system integrators (e.g., medical device designers) to calculate the failure rate for their specific use case:  
  MTBF(precision)  \= 1 /  f\_clock \* P(Noise \> 1/2 LSB\_Precision)  
  ![][image1]  
  * *Example:* A device might have an MTBF of 1 second at 8-bit precision (useless for medical) but an MTBF of 100 years at 4-bit precision (highly reliable).

### **2.2 Reporting Completeness Levels**

Conformance is defined by the transparency of the data provided:

* **Level 1 (Research):** Basic Envelope.  
  * Reports: Dynamic Range, Supply Voltage, Temperature Limits.  
  * Goal: "I exist and can be powered on safely."  
* **Level 2 (Engineering):** Statistical Distributions.  
  * Reports: Conductance PDFs, Quantiles (P50/P90/P99), worst-case envelopes.  
  * Goal: "I can be simulated in software."  
* **Level 3 (Safety/Commercial):** Time-Dependent Metrology.  
  * Reports: Drift coefficients ($\\nu$), Aging curves, Noise PSD, and certified MOBF calculations.  
  * Goal: "I can be deployed in a product."

---

## **3.0 Layer 2: The "Interop Profiles" Standard (Optional)**

This layer defines rigid specifications for devices intended to plug into standard ecosystems (like UCIe chiplets).

### **3.1 Logical Interface Profile: "Standard Command Set"**

To prevent software fragmentation, devices seeking "Logical Interop Certification" must support the **Abstract Command Set** (mapped to their internal physics):

* SET\_Target(Address, Value, Precision\_Goal): The host specifies the *goal*, the device handles the iteration/verify loops internally.  
* GET\_Status(Address): Returns the current confidence interval of the weight.  
* CALIBRATE\_Background(): Triggers internal drift compensation routines.

### **3.2 Physical Interface Profile: "UCIe-Analog Extension"**

For chiplet interoperability, we standardize two **Physical Encodings**. Devices must support at least one to be "UCIe-Analog Certified":

* **Profile A (Differential PAM):** For high-throughput. Uses differential pairs ($G^+ \- G^-$) for common-mode noise rejection.  
* **Profile B (Time-Domain PWM):** For high-linearity/low-voltage. Encodes value in pulse width.

---

## **4.0 Safety & Regulatory Alignment: The "Mitigation" Approach**

We reject the notion that analog hardware must be "perfect." Instead, we align with **ISO 26262 Part 5**(Hardware Architectural Metrics), treating analog noise like soft errors in digital RAM.

### **4.1 The "Class A" Redefinition**

Instead of a rigid "\<1% Drift" rule, **Class A** is now defined by its **Mitigation Capability**:

* **Old Definition:** Device must not drift more than 1%.  
* **New Definition:** Device must provide **Observable Reliability**.  
  * Must have built-in **Drift Warning Triggers** (e.g., a reference cell that flags an interrupt when drift \> 0.5%).  
  * Must have characterized **Noise Tails** to $6\\sigma$.  
  * *Why:* This allows a medical host system to say, "The analog chip has flagged a drift warning; I will pause inference and recalibrate." This *detectability* is what makes a probabilistic system safe.

### **4.2 Application to Medical/Automotive**

The standard provides the "datasheet inputs" for the System-Level Safety Case:

* **Medical Integrator:** "We use a 'Level 3' reported device. We run it at 4-bit precision where the noise-induced MTBF is $10^9$ hours. If the background calibration fails, the device asserts a HARD\_STOP pin within 10ms."

---

## **5.0 Revised Roadmap: "Measure First, Standardize Second"**

* **Phase 1 (2025-2026): The Metrology Era.**  
  * Publish the **Capability Declaration Schema** (JSON).  
  * Establish the **Round-Robin Labs** to verify Noise PSD and Drift measurements.  
  * *Goal:* We stop arguing about "which encoding is best" and start agreeing on "how to measure noise."  
* **Phase 2 (2026-2027): The Interop Era.**  
  * Finalize the **UCIe Analog Extensions**.  
  * First commercial "Level 3" datasheets appear.  
* **Phase 3 (2028+): The Safety Era.**  
  * Regulatory bodies (FDA/ISO) accept **Analog MTBF** calculations as valid safety evidence, provided the inputs come from Phase 1 certified measurements.

---

## **Appendix A: Capability Declaration Schema (Draft Snippet)**

JSON  
{  
  "device\_id": "Softoboros\_Gen1\_Analog\_Core",  
  "metrology\_level": 3,  
  "physics": {  
    "type": "Continuous\_Manifold",  
    "encoding": "Time\_Domain\_PWM"  
  },  
  "reliability\_metrics": {  
    "noise\_floor\_RMS": "1.2 mV",  
    "noise\_distribution": "Gaussian",  
    "drift\_coefficient": 0.04,  
    "mtbf\_table": {  
      "4\_bit": "1.5e9 hours",  
      "8\_bit": "2.4e3 hours"  
    }  
  },  
  "safety\_features": {  
    "drift\_monitor": true,  
    "ecc\_compatible": false  
  }  
}

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnAAAABkCAYAAAAGyw0HAAAcI0lEQVR4Xu2dZ7sU1dKGz3845oiKWfCcg4lkIKgoYBbBiIKCgmJCEBPBnFDBS1RQQTBgQBQRURADgoiYMGd/yLzX3bw1rqlePXHP7N3D8+G+9p5Va3X3dM90P1Orqta//v3vHQpCCCGEECI//Ms3CCGEEEKIro0EnBBCCCFEzpCAE0IIIYTIGRJwQgghhBA5QwJOCCGEECJnSMAJIYQQQuQMCTghhBBCiJwhASeEEEIIkTMk4IQQQgghcoYEnBBCCCFEzpCAE0IIIYTIGRJwQgghhBA5QwJOCCGEECJnSMAJIYQQQuQMCTghhBBCiJwhASeEEEIIkTMk4IQQQgghcoYEnBBCCCFEzpCAE0IIIYTIGRJwQgghhBA5QwJOCCGEECJnSMAJIYQQQuQMCTghhBBCiJwhASeEEEIIkTMk4IQQQgghcoYEnBBCCCFEzpCAE0IIIYTIGRJwQgghmsK4ceML69evLzz//PMpmxCiMSTghBBCdBiDB59YGDNmTOGZZ54p/P333wkScEJ0PBJwQgghOoxLLhmdsMsuuxaWLVsmASdEk5CAE0II0RQk4IRoHhJwQgghmoIEnBDNQwJOCCFEU5CAE6J5SMAJIYRoChJwQjQPCTghhBBNQQJOiOYhASeEEKIpmIBbvHhxyiaEaAwJOCGEEE3BBNySJUtSNiFEY0jACSGEaAom4F544cWUTQjRGBJwQgghmoIEnBDNQwJOCCFEUzAB99JLL6dsQojGkIDrwhxyyKGF5557LtUuWsuiRYsK69atS7ULIcpjAu7llyXghOhoJOC6MIiGAw88KNUuWgvX4Kuvvip07949ZRNCZLN27dpEwK1e/V7KJoRojJYJOL7Enh122DHVL4YfB6NGnV/WXg02/uuvv07ZsrjrrrtTx7frrrul+lWD307IrbfeVpg+fXpJmx+fxfr16wvjxo1PbbOzoRZUI1MpjY5vFM7t00/PT7ULIf7hoYceKvz000+FL7/8svD5558XNm3alPzl9ddff1OYMWNGaowQonZaJuAMvsh4M3gYzps3L2X39O9/bOHhhx/OFD3duu2TtF900cWFXXbZNdUOO++8S8mYDRs2RNuBdgSdb4e99tq7uM05c+ak7DY+61gBsffnn39m2uGHH34oa//ll18T+19//ZWywcqVKxM72/G2zmLo0KHF83Lsscel7JVodHxHMGrUqGT/++67X8omhBBCtJJOEXB4sXgQ/vLLL4W99+6W6hPy4IMPJuIsSxT95z//TQSRbw/F1o477lRiO+aYY5L22PQk7Ryjbzf4BZl1LDYeYsdkXHvtdUmfLA8kti1btqTaDRNwv/76a8oGhx/+n2KfQw89LGXvDPbf/4DiueHaeHslGh3fEey0086Fzz77LOUZFUIIIVpNpwg4/q5atSp5GL/99tupPsbIkaMSIVROwJ155lmFa66ZlGrfY489M8fA1KlTC0OGnJJqp/+GDRtT7cZZZ51d3C7eNG832x9//JGyhcyaNatwwgkDUu28n99++61svJWJsx9//DFlMzgn9GHaIuZpFPXBD46sz5QQQgjRKjpNwI0ff2VZgQUvvvhSkr1UTsCNGTOmcNppp6Xad999j8wxQAwdAtG3VxJwp576z1QeXiFvN9vvv/+esoUg1GL75/2uWLEi1R5iAq7cFOnYsWOLx9KjR8+UXdRP1mdKCCGEaBWdJuDABEaWh+i7775LvFzlBNw999ybmiKF3XbbPXMMEMc0bdq0VPs2Abch1W68+eabZbebJeCOPPKokjFMn/qEiCuuGJf0GTFiRGq7IdUIuNtvvyPp8+2335bEBgLn1OK4ss69gdAcNGhwqt3D+xk5cmQipvF+ervBtYrtk2nR008/o9Cz5zaxiVD2fcqNN/r27Vfo3btPqt3DNrwAR/T7fjE4r5dddlmqXQghhGgVnSrg+J+HYazW2cCBAwvHH39C8n85AZdFJQGXRZaAQ2AsWLAgsT/66GPJ9n0fGx8TcCQ9VDqW9957v2IfqCTgiHtj//Qx79uAAQOLxwavvPJKMj1NVievX3311eJ4Yr0QxrQjpP73v17JtO7y5W8WDjro4JJ9IfAQ2t98800i3gYPPjE5Ljj44EOK/cJ9H3HEkcV2MtJoYzqb/RK/9/PPP6fi+7LG2zGwv/nzFxTOP/+CwsknD0m2QZJHeJ1i2cIIMcZNmjSp8NFHHyVtiPRw+55PPvmkbIyiEEII0Ww6VcCZl8iLHZg9+5Hi/60WcFu3bi3cdNNNCXjpKOlB8Dq2p556OjXGj7f3hOhDTA0aNKiqYzHR5ds9lQTcG28sT+z0szY8ThyHZahu3ry5MHny5MLSpUtT14CFp2m7/PIrim0XX3xJ0ubLaNj7OuCAA1NtM2fOLLaRTWztoQBDZHmxhiDzbVnjbRv+vOGJQ2RxLsJ2Eld4D7YtMqLNhgfOSsqEYzx2fst5AkP22697kjlbL357QgghRKcKOMCr4x+YF154UUkSQKsFXMwDx8P6ggsuTOwLFy5Msl99HxsPiAri2UjSQEhUcyzV9AETcFkgMLLqwCEIwv0wJYnnzKYP99xzr8zj8O0m/kLvnbUjFBEusfGhAOO1F2sQa4uNJ16QttGjL031N48b09dhe7n3aKVCfHvI3Llzo9vN4rDDeiTbLQfTz+edd17h3HNHFM4559zC2WefkyTMIGb99oQQQohOF3D2MH3ggQeKbYifMA6sKwg4gylJ226vXkek7GbzXkUrHeL7x8b6dk8o4LytEubJyqohR9kW7ExBepER7nP06NHF14gOv50Y5mEMBRhC3bbDFDJFQPlM+LFZ43kdetE82NesWVPSFk6l+v6WZezbQ/DK1vK+hRBCiI6m0wUc8DAMg+2JpwoD77uSgLM+ECvmazYv4IjxqnQs1R5vRwi4mIcLXn/99eK2s8AbaTF90K9f/9R2YsQE2M0331ycAjWIqfNjs8bzOvaZCu2MCz9P/G/78v3xeMXaQ66//oakD9PK3iaEEEK0gi4h4D799NPkgUjZkBNPPKnQp0/fEntXFXCxbVu7F3B4fQiu9/1jY327pyMEHEvdeBtYfFdWfJ0RxpFRGNnbY5AIQX8fwwYkSzAVy3HRB1Hnpyhj43ldrh6eHSOJGNZGskTW+WM6OdYeYl5KPqveJoQQQrSCLiHgpkyZkjwQmU6LebVaL+Cy68BZn6xtW3u5lRiyyNqmpyME3Pfff5+yQblly0ImTpxY7Dd8+PCUPUZMgB19dKn4w7uHB5Z+Tz31VMXxdgyIMr8/szMuzEZtVMBxXPSpplwJUO+PdVzrxW9PCCGE6BICDuyBysPW21ov4LI9cOEKD2SnervZat0vsLRYNeMaEXCWxJAl4MpNLwLLSJlYIiOXfpTf8P04N94zljUFSnydH0/7O++8U3E8K03QRuKL34bFV1JaJGwv99mgFl2sPYTEFDyFWaJRCCGEaDYtFXCU1ODhiJeN/0MbcU/YTjvt9NS4G264MfOBm0WYbXnUUUen7DFYvor+lJKIrVMalgNBOHg7mL2WYzUQR4zr3bt3yhZiQqaefZAcwjg/xRtiU4RebJNV66dWLQmBDEprQ9iQBOGnVu2Yw8LAvPYJFWSv0u4zMGPjs7xplBHhGsWuE8WCY2PAiil3775/ymZg90WYhRBdG6oIEHNr9yxvFyJvtEzAhYvAG7Nnzy7ayT6kLfRqUCLi3XffLRmzfv36xAMSq9R/yy23JpmM1GxjCtPG8IXFS4SN2md+HLF3a9euLcZfAbXgPvjgg8L7769Jshh5bbZ58+aVVPHHa4W3aN26dSXHiiePY6Wsht9nDFumK6sECPvgmMJ9bNy4MTlHWYWFDcQUBWhD8cfxhrXaDK7BXXfdVXyvY8denpTpwEPoV2XAY4WnjXNMZirikyLBVoTZ4PzafjnPVrzZ2qj7163bPkmh4NWrVxceeeTRqsYDYop2riNrwFKGw7yZYTFhrhOfKbMBnyemlXlfTJ3bzR0Byj7DYzCwsxSbbxdCdF14LixevLh4D/V2IfJGywScqA4C+Unq8O2dBR5R1lXNqntnIJRYlxYvpbeVg5UX+EsdOmLFhg4dWnWBXA+i+qSTTo56cTsS3fxFvWjavfNhJkHf4fbGLx/ZrkjAdTGYujXPl7eJzodl1Kr1qHZluMHZMmuiNbAyS7mahaI1tFrAEYtrazwbVFqoNGsi6ocVd3CEEBblbe2EBFwXhJtLuRg10TkQF0cCiY/Nyxt4OJn2JvZw2bJlKbvYBjd/lpMj4/qaa65J1ssFXl911YQkXhKvczXlZPhhtmjRosKwYcOKbUz3e7IeOBwHiTq2agdT+ByDPHq1U0nAcX2Jl2NGgNhe4LyTTMf19v2zQLATQmNhK4RvcO2wkazHbIP19Z8DPmdXX311co259qzZfNxxx0djs0UcwqgqrWuddyTguiAnnDAgEQrykHQtKDZNLKdvzxOWZWt/YeDAgal+YlvSFUuaEZNq54r1kclq5oF+ySWjkwes2XyCTwhimXjasI2HOSKBh7VtI0tYTJgwofD4448X+yBCOA6WwvN9q8FWI8EjWG/IQl6pJOC4Lgg1KxcETzzxRCLqYhnzMSzmO4yV5jwTp00RdWyhgONzQPKYFTWnWDifL9oR7sQgf/HFF4kNYVLvdd+e4BytWrWqcOON6bj3dkECrovCjeKtt95KtYvOgZsvv6SJsfO2PGFJQcQc2sOJh7nvJ/6h3Nq5QJFos8cy3vfeu1tiy1q5g9JElPWxbfTo0SPVxyBpCW+gb68VxIPtj4x9b29nKgk4IywZVavIZUwsA95sEAo4gx+JWceGR48YaeyxeqkiDc9Rfljtu+9+KVs7IAHXhaGoLoVlfbtoLXirfKmTvGIPD/6nnh4PBd9HlELpifC8xTB7bIk6stHx3Pl2A6EwZ87cRLjZdrJEGv0os+Pb64EqAL7Uz/ZAPQKulqlq4mQZQza8twErzNQj4Aw7JrxL3ibScK6yVh7KOxJwQmwnWIJMpQeEKMXKC+H98DbDzmtsBZZK59sEHP+/9NI2cZElGujXTt4ESnvwXlsZV1qPgKsl9szKlDDl7W0G9kYFXKV+Yhu2vrdvbwck4ITYTqAUDDeydvAm4sWIPQBjUP/Qt9WCPSynTp2asvk+S5YsKWm36VXfPyQUcJTjsW3FvHb022effVPteYWpScoU8X6pI1nrVGU91CPgvK0c1JpjTLl4Weyxz28tAm7Tpk0pm0hz++13JOerHWPKJeCEaHMoekwGpBWaZtUT1lidP39Brm9qJAXElt4zBg8+MfGeUVLA22qBc0bx56zaUmeddXbmg56C1LH2kFDAAdfLtucTH+hHTJ3fhnHvvfcVxzKdGyt4/sADDyTxtbbmcLg0HRAniScRoU+SAwH8salhwGMYFgcn+N73qRYSRihWjvBh2trbG4XjJNaQ7QNiibYsj2a9Aq5XryOK4/iuMaV6wAEHpvrFqCTgyFDFTva4Ehmqw1b2efLJJ1O2vCMBJ0SbYw8TD6tOhFlyeYOyLqyWES7jFoJ4a9T7hkeIc8XqIt5msOQcfWIrd7D6SyVPiRdwXBOW84uJB/plxS0yDUl/slMREWQ8IsLuv//+kn5+xZhQwM2YMSMpfTFkyCnJFO6UKVOSVUliAu6ee+5Nxt933/2Jp/G2225PXrOaiu9bCxao39nL1dUr4CA8vwbn6Ywzzkz1DSkn4BYsWFDclmrI1Qb3gS1btqTa844EnBDbCXbzt9UvqoVl7nh4TJs2LWWrFx5mTGUx3eRttULMEeUWwjbEW7iMWr3YOsz8PfnkIUnNN6a+8LrhLUIg4dXx4wDvJmNZ4s3bQrYJuDmpdrteLANobVkCLmt9z0svvTRpv/ba61I2234o4GLbAC/gmO6M9V25cmXSTikkb6sVMnrxkOEN7Nevf8rebBoRcGQuL1y4sDjekzVVbAKOOo0GJaXMy0lds9i5YBoaAQ233npbAgKeafhW/kjjnPm2rgAzDvVcx66OBJwQ2wncwN54Y3mqvRLclBEqtS6TVgmOp6OmyhBxiBgEFuLtsMOyS3FUixU85jgpnGuce+6IxNtVyQvCsTCW+l7eFsL5nTv3Hw+cMXz48OID38rXIOD8Q5LjoQ/v328DLDHC78O27QUcIsKXPOnbt1/x/7Aunt8X09a0I+S8rRHuvPPO///8vpGyNYtGBFwInszrrrs+KWdh24PY9HY5DxwQ/4idz7gXyewHoccUeNhupYMa9YxWgqlo9oPH1ts89OWeUk3fjsC8xb4970jACbGdwA0Mz5dvrwQPGsZmxYDVg9Wh66i6engZEHE8ECjb4e31MGDAwIYe4BYbRxC1t4UgFLIyFmfPfiTZBu8Jz1tMwM2cOTPpk7VMFzFs2NesKY2nM69OKODMkwdM/1JEFm9SOA5Bag9q6myFXHnlVYkNIeKPo1EQ+8Tt4flkitfbO5pqBRzfi2rWX0ZghfGSsVVQKgk4IO6TPlu3bo3afP1Qpre51suXN3dVAstynzVrVsrmoS+xntX07QiYPah0XvOIBJwQ2wE8ZLiB9e7dJ2WrBNM2HX3zmz59el3ewCzwiPHwwvMTK+VRD5atWO97N+GL98jbQhAKiCzfblh1fogJOMQdNir1+7FAPcnY+zAh4JMYKEBr/Q2m5Pz+iP1DVMbwx9goeEOZSma/1MjjB4Dv09FUK+CYUg/LoFTyuOLdzNpuNQLOpsWhW7d9Smxc05hQs6ltplq9bXvAMlF9e96RgBNiO8BKNfj2EKZIeTiTOceyQdbOOLIw0/0HJwHyTA0RwO/jengAMnXENOS2haX3L9rwCJgXBe8Z3pVKsWJZ8PBcv359MeaNLM2s6cRasIdkpQdyFizFxfhnn302ZQvhPJXLkLNlrwAx58VRuMSWHwtW1sJn7MYE3NFHlxb2ZTkny1ZlaSnasgRhM5g8eXLiWSQbN1YXr5lUK+CWLl1aEm/Jd6iStzpru9UIOAoE23gvvrmm/ocRYhdvKbGaTPuTSMH5RPAzHiH+wgsvlowha53vPNfex9DxPUbk80OJ75197/nBw3sPl5Rj++z3448/Tqa/yYC2vkzt+uXnCFEgiYX9knQQ7pvzTN1A7iV47gkJ+PDDD6uK6bVl0Xx73pGAE2I7wKa9fLvBNBnigLiaQw45tKQv//situbRY4qvZ8+ehWnTbim89tprJX242fKQIJif6cgwLoqxFkOG+Bs//sq6p914aPmEBTIpfWJDrdhDMivLtRI2TezPnQehYOIoC5vuBC/gWPQ8SxDA6tWrE5tPGIkJuNi0HHb6WTYt63KW2x94MV8P5jWJeZQaoVohWI2A69nz8OTahO93w4aNJTGDMbK2W42As2xf8Oc5JuBsJRH7PvDDiR9ptCGsvIedLGamYbkPsCYsGdHh92vz5s3JDzy+v08/Pb/oPX7ssTmp98X7sbI3iHETcPS1Is7Wl/fCPYiEIV4z9cu+zW4/VBCPfKf4wYln2IcGxODHYaXzmkck4IRoc6gXZasJeBvg/cFm1eYRW9zg+d9EiE9giG0rbONBQtamveYXd+jJoi9TTytWrEhec7Pmhu63WQ7ijthPVnA22+bh6turoZqHdzWw3JH3MngI6PYeEA8PNwtG9wIO7Fj9igEm7iiD4m0W7+aTGPy2rT0sdWIe3dhSXGRNskyXb68GjoVrxrH16dM3ZW8EHuL8WGB5QouJKlefrZrPQMyOgKMtqyaeZfBeccW4lK2SgLOyNhBmJxt8HwDPFOVi+CETK12Cp8uuPfeH0PPq989r894iHsM1XvHCDxw4sKRvOJ7/w3p1EyZMKP7fv/+xJX353PjPE97eefPmZW7f2iqVnMGj54VtOyABJ0SbY2sv+ik0I3ZTNO/YKaecmky/hFNClmkY207sfwgD4W1xeJa4CWN4vDehEpS2IHHBtxvEYiHiyNL0tkrgeYidl1oh1q/cNhBVPGTpU6lMi13HmIBjOgnb3XffU9Ju8WwxkWvvj4dw2DZp0qSSflYI1S93RRvTb3hqrI3pXgRr2FYtTDXjgcGry6oh3t4oq1e/V/J54/tAQkRWQVxbRQN8xvGIESMSz2DsM2ICDiEaK7rMjyk8X/7z3r1792LWsxfbwI8opiGxc13x/vk+Jt58uwdvHNvx7x3v5LbPxKAiFoOJHW+X97SH+PPB/4gnfiT640Wg+75+e5a9a+fDb9/ayiVn2XWsJPLyiAScEG2OpdCTrehtgC0rBotfrRRzDdvwEoS/ioEHld1YyUb0N9kQRA0PP7w09MMj4vtUggdxVgV9Ty2lShCrPESZSuR9Er+DUOR1rP5aJSx+zbcDD1tWBEAEMT3J1FRWTTmDaVAvJkJYXcMecnDVVRNS8VgIC94b+wPEAHFF2Ig3o3ZYWPKC48uadsSzGmauEqfk+5SDHwhkY3IemrlEGOKI4wtjOUeOHJm02bSeEZ6/agnHI+D4PiDQzKMWEss+9X0A4cS14pgRfVwbxD6eKz/eqFXA+XY8cbF2sBImeMW8zYidD35c8PmiPTzXvXv3Lunrx4FlNTMrkLV9Xvti1SGWyR0T03lHAk6INgdRwg0s68GPzcd54eUhVgYbAgAIyucvUx0EUof9mQKk2C//M70W84zhBeIYeChZ+RCmk6wvxTb9mHaAcxgrviq2XX/viWoWZAUjqO0103lcm0qezzyBgKtm9RGbQvXtEGun9h9/+THk13gdO/byYrKBF1h+bWBsFh9IRnzYl/99UgbT9sTohX388fHai3Bv92PaBQk4IdoQboQsi8T/lW5g2AgwDtvIUps4cWJxXDh9hhco9OYR0Ew/ptps33ix/H6IcUO40dcepJSEMOFmcXftBnFX3mMpOh9+cJBJ2avXESlbXkHA4QH07R7q+2XdE0gcwDNqrwmZsB9Zllkd9idb1Ly8/l7js8HxKlo4hZ9C5Ueg95ZzTwiPxW/f2splimOv1TOcFyTghGhD7EZH3Al/iTfzfQymZBBsTIcxhccSQGYjW4wsRp/BSPkA4oeY+kPohZ4N4Bcx8TJsk5s/03K2PX8DZrqOgGu/jXbBCpzecUf5gr6itXBNWuX9azaEJNi0P+B1zyoOzbJq9MWTxl/7boZY3TjKgjDVHNr4gYYXntACpuCt3cqOgLVzXEz/ksyD593CEOjLvsO+QHgFbewXsTds2LDo9tk3U7Dcr3gfvGcKA/v3gXeV+5eP9WsXJOCEaEO4+XJT27hxY+Giiy5O2UVrIS6QB1K7CIY8Q/wX1yIrrk+0B4jHerPQ84IEnBBtCFOeTNvFfpWKzuHGGydnJpKI1oH3xlaWwCNEgoPvI/IPP2LDsiXtiAScEEK0CB9rKFoHpU18rUGm+KmV5/uKfINAj5ViaTck4IQQQrQ9xGISD0WgPLFRxCQSZB/GWQmRJyTghBBCtDWWzOMhGL9dA9xF+yMBJ4QQQgiRMyTghBBCCCFyhgScEEIIIUTOkIATQgghhMgZEnBCCCGEEDlDAk4IIYQQImdIwAkhhBBC5AwJOCGEEEKInCEBJ4QQQgiRMyTghBBCCCFyhgScEEIIIUTOkIATQgghhMgZEnBCCCGEEDlDAk4IIYQQImf8H+p50flwNvO0AAAAAElFTkSuQmCC>