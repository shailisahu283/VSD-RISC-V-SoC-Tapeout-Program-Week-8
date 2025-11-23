# VSD-RISC-V-SoC-Tapeout-Program-Week-8
# 🌟 **Week 8 – Post-Layout STA & Timing Graphs Across PVT Corners**

**VSDBabySoC – Post-Route Timing Closure**

---

# 🧭 **Objective**

In Week 8, you perform **Post-Layout Static Timing Analysis (STA)** using:

* The **post-route gate-level netlist**
* The **extracted parasitics (SPEF)**
* The **multi-corner Liberty (.lib) files**
* The **SDC constraints**

This validates the true timing **after layout parasitics are included**, making this the most accurate representation of silicon behavior before tape-out.

---

# 🚀 **Why This Task Is Important**

This week, you **close the full ASIC flow:**

RTL → Synthesis → Floorplan → Placement → CTS → Routing → **Parasitic Extraction → STA → Tapeout readiness**

You will now see:

### 💡 *What changes when SPEF is added?*

✔ Delay increases because of **wire RC**
✔ Paths behave differently at **TT, SS, FF** corners
✔ Setup can worsen due to higher wire delays
✔ Hold may degrade at fast corners

### 🏁 *Why this stage is essential for tape-out?*

* Only post-layout STA predicts **real silicon timing**
* Ensures the chip meets timing across **PVT variation**
* Exposes corner-specific timing violations
* Allows ECO fixes before fabrication

---

# 📚 **Reference Material**

🔗 **Timing Graphs:** Day 26 Reference
🔗 **PVT STA TCL Script:**
[https://github.com/arunkpv/vsd-hdp/blob/main/code/openlane/designs/riscv_core/sta_across_pvt_route.tcl](https://github.com/arunkpv/vsd-hdp/blob/main/code/openlane/designs/riscv_core/sta_across_pvt_route.tcl)

---

# 🧩 **Task Components (Fully Explained)**

---

## ✅ **1. Load Post-Route Design into OpenSTA**

You load the following:

### **a. Gate-Level Netlist**

The gate-level netlist produced after routing includes:

* Buffer insertions
* CTS clock tree
* Optimizations
* Tie cells

🔍 *This is the actual silicon logic.*

---

### **b. Liberty (.lib) Files for All PVT Corners**

Each `.lib` file corresponds to a **PVT corner**:

| Corner | Meaning                                       |
| ------ | --------------------------------------------- |
| **TT** | Typical Process, 25°C, 1.80V                  |
| **SS** | Slow NMOS + Slow PMOS, Low Voltage, High Temp |
| **FF** | Fast NMOS + Fast PMOS, High Voltage, Low Temp |

### ⭐ Special Note

**TT ≠ guaranteed working corner. Real failures often appear in SS (setup) and FF (hold).**

---

### **c. Parasitic SPEF File**

SPEF = **Standard Parasitic Exchange Format**

Contains:

* Resistance (R)
* Capacitance (C)
* Coupling caps

🎯 *This is crucial — routing parasitics directly affect timing.*

---

### **d. SDC Constraints**

Includes:

* Clock definition
* Input/output delays
* Load constraints
* Timing exceptions

---

### 💻 **Commands (Typical Flow)**

```tcl
read_liberty lib/ss.lib
read_liberty lib/tt.lib
read_liberty lib/ff.lib

read_verilog soc_postroute.v
read_sdc constraints.sdc
read_spef extracted.spef

update_timing
report_checks -path_delay max -fields {slew cap input_pin} -digits 5
```

---

### ❓ **Q&A – Section 1**

**Q1: Why do we need SPEF for STA?**
✔ Because **70–80% of delays in 130nm-45nm nodes come from wires**, not gates.

**Q2: Which corner is worst for setup?**
✔ **SS corner** (slow transistors → max delay → worst setup slack)

**Q3: Which corner is worst for hold?**
✔ **FF corner** (signals move too fast → hold time violation)

---

---

## ✅ **2. Generate Post-Route Timing Reports & Graphs**

You analyze:

* **Setup timing** (max path delay)
* **Hold timing** (min path delay)
* **Slack values**
* **Capacitance, resistance (from SPEF)**
* **Timing paths with annotation**

---

### 🌈 **Timing Graphs**

Using Day 26 graph format:

* Nodes = pins, cells
* Edges = delays
* Annotated with **RC parasitics**

You generate graphs for:

* **TT**
* **SS**
* **FF**
* (Optional: additional corners if available)

---

### 📊 **Key Metrics**

| Metric  | Meaning                                       |
| ------- | --------------------------------------------- |
| **WNS** | Worst Negative Slack (max delay path)         |
| **TNS** | Total Negative Slack (sum of violating paths) |
| **WHS** | Worst Hold Slack                              |
| **THS** | Total Hold Slack                              |

---

### ❓ **Q&A – Section 2**

**Q1: What if WNS is negative?**
🚨 Design fails setup timing.

**Q2: Why does WNS worsen from Week 3 to Week 8?**
✔ SPEF adds real wire delay → timing becomes slower.

**Q3: What makes hold slack negative?**
✔ Extremely fast data path in FF corner.

---

---

## ✅ **3. Compare Week 3 vs Week 8 Timing**

This is one of the most important deliverables.

You show:

### 🏁 **Before parasitics (Week 3)**

* Ideal wire delay
* Optimistic timing

### 🏁 **After parasitics (Week 8)**

* Actual wire load
* Realistic timing

---

### 📋 **Comparison Table Template**

| Corner | WNS (W3) | WNS (W8) | TNS (W3) | TNS (W8) | WHS (W3) | WHS (W8) | THS (W3) | THS (W8) |
| ------ | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| TT     |          |          |          |          |          |          |          |          |
| SS     |          |          |          |          |          |          |          |          |
| FF     |          |          |          |          |          |          |          |          |

---

### 📌 Expected Observation Patterns

* **WNS becomes more negative after routing**
  → Due to RC delay

* **WHS becomes more negative in FF**
  → Paths too fast

* **TNS increases**
  → More paths fail after adding parasitics

---

### ⭐ Special Fact

In real tapouts, **post-layout STA is the single most important step for timing closure.**

---

---

## ✅ **4. Interpret the Results (Very Detailed)**

### ✔ Why is Post-Route Timing Worse?

1. **Routing adds long wires → more RC → more delay**
2. **Coupling capacitance (C_coup)** introduces additional delays
3. **Clock tree insertion delays modify path timing**
4. **Buffer insertion during optimization** changes path structure
5. **Load on nets** increases significantly after layout

---

### ✔ How SPEF Affects Path Delays

* **R × C products** increase delay
* Coupling causes **crosstalk delay**
* Net capacitances slow down transitions
* Rise/fall delays become unbalanced

---

### ✔ Why Different Corners Show Different Failures

| Corner | Failure Type   | Reason                        |
| ------ | -------------- | ----------------------------- |
| **SS** | Setup Failures | Slow transistors + slow paths |
| **FF** | Hold Failures  | Very fast transitions         |
| **TT** | Balanced       | Normal reference corner       |

---

### ❓ Q&A – Section 4

**Q1: Can a design pass TT but fail SS?**
✔ Yes — very common. SS is worst for setup.

**Q2: What is a typical hold fix after Week 8?**
✔ Inserting delay buffers in fast paths.

**Q3: Why is PVT analysis mandatory?**
✔ Silicon never behaves at only one corner.

---

---

# 📦 **Deliverables (What You Will Submit)**

### **1. Timing Reports & Screenshots**

* report_checks outputs
* SPEF-annotated timing paths
* Slack summary for each corner

---

### **2. Comparison Table**

Week 3 vs Week 8 for:

* WNS
* TNS
* WHS
* THS

---

### **3. Documentation on GitHub**

* Step-by-step commands
* Timing graphs
* Interpretation
* Screenshots
* Comparison plots

---

# 🌟 **Extra Special Notes to Make README Stand Out**

### 🟣 Meaning of Timing Slack

**Slack = Required Time – Arrival Time**
Positive slack = good
Negative slack = violation

---

### 🔵 What is Coupling Capacitance (Cc)?

Capacitance between adjacent wires causing **crosstalk delay**.

---

### 🔴 Why is SPEF accurate?

Because every extracted segment of every wire has:

* R (resistance)
* C_total (full capacitance)
* C_coup (coupling cap)

This is the **real physical layout**.

---

### 🟢 Why Does FF Cause Hold Violations?

Because fast process + high voltage + low temperature =
**Signals move too fast, violating hold.**

---

### 🟡 Why STA Instead of Simulation?

STA checks *all possible paths simultaneously*, whereas simulation checks *one pattern*.

