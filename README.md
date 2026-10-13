<div align="center">

# ⚡ EIGRP Successor – 2 Paths (Equal-Cost Load Balancing)

### DUAL · Multiple Successors · Redundancy · ECMP — Cisco EIGRP AS 1

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP-blue?style=for-the-badge)
![AS](https://img.shields.io/badge/AS-1-orange?style=for-the-badge)
![Successors](https://img.shields.io/badge/Successors-2-brightgreen?style=for-the-badge)
![ECMP](https://img.shields.io/badge/Load%20Balancing-Equal--Cost-purple?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

</div>

---

## 📌 Project Overview

This lab is the **advanced follow-up to the EIGRP Single-Path Successor lab**. A second serial link is added between **R1** and **R2**, and both links have the **same metric**. As a result, EIGRP's **DUAL algorithm** finds **two equal-cost best paths** and installs **both** in the routing table:

> 🎯 Topology table: **`2 successors`** → Routing table: **two next hops for one prefix**

Traffic can now be **load-balanced** across both links, and if one link fails the other **already carries traffic** — the route never leaves the Passive state and no Query process is needed.

| | 🔹 Previous Lab | 🔸 This Lab |
|:--|:--:|:--:|
| Links between routers | 1 | **2** |
| Successors | 1 | **2** |
| Next hops in routing table | 1 | **2** |
| Load balancing | ❌ | ✅ Equal-cost |
| Link failure impact | Route goes **Active** | Route stays **Passive** |

---

## 🎯 Objectives

- ✅ Configure EIGRP AS 1 over **two parallel serial links**
- ✅ Understand how **equal metrics** create **multiple successors**
- ✅ Read the topology table with **2 successors**
- ✅ Verify **two equal-cost EIGRP routes** in the routing table
- ✅ Understand **ECMP / equal-cost load balancing**
- ✅ Compare **Successor vs Feasible Successor**
- ✅ Test **redundancy and failover**
- ✅ Learn advanced tuning: `maximum-paths` and `variance`

---

## 🗺️ Network Topology

```
   192.168.1.0/24                                                  192.168.2.0/24
                          Link 1 : 1.1.1.0/24  (Serial0/0/0)
                   ┌───────────────────────────────────────────┐
                   │ .1                                     .2 │
PC1 ─┐          ┌───┴──┐                                    ┌───┴──┐          ┌─ PC4
PC2 ─┼─ SW0 ────┤  R1  │                                    │  R2  ├──── SW1 ─┼─ PC5
PC3 ─┘   Gi0/0  │ 2911 │                                    │ 2911 │  Gi0/0   └─ PC6
         .1     └───┬──┘                                    └───┬──┘   .1
                   │ .1                                     .2 │
                   └───────────────────────────────────────────┘
                          Link 2 : 2.1.1.0/24  (Serial0/0/1)

        Router ID: 192.168.1.1                      Router ID: 192.168.2.1
                              EIGRP AS 1 on both routers
```

> 📷 Add your screenshot: `![Topology](images/topology.png)`

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Mask | Connected To |
|:------:|:---------:|:----------:|:----:|:------------:|
| **R1** | Gi0/0 | 192.168.1.1 | 255.255.255.0 | Switch0 (LAN 1) |
| **R1** | Se0/0/0 | 1.1.1.1 | 255.255.255.0 | R2 Se0/0/0 (**Link 1**) |
| **R1** | Se0/0/1 | 2.1.1.1 | 255.255.255.0 | R2 Se0/0/1 (**Link 2**) |
| **R2** | Gi0/0 | 192.168.2.1 | 255.255.255.0 | Switch1 (LAN 2) |
| **R2** | Se0/0/0 | 1.1.1.2 | 255.255.255.0 | R1 Se0/0/0 (**Link 1**) |
| **R2** | Se0/0/1 | 2.1.1.2 | 255.255.255.0 | R1 Se0/0/1 (**Link 2**) |
| PC1 / PC2 / PC3 | NIC | 192.168.1.2 / .3 / .4 | 255.255.255.0 | Switch0 |
| PC4 / PC5 / PC6 | NIC | 192.168.2.2 / .3 / .4 | 255.255.255.0 | Switch1 |

---

## ⚙️ Configuration

> Standard configuration for this topology — adjust to match your own running-config if different.

### 🔹 R1

```bash
enable
configure terminal
hostname R1

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 ip address 1.1.1.1 255.255.255.0
 no shutdown

interface Serial0/0/1
 ip address 2.1.1.1 255.255.255.0
 no shutdown

router eigrp 1
 network 1.1.1.0 0.0.0.255
 network 2.1.1.0 0.0.0.255
 network 192.168.1.0 0.0.0.255
 no auto-summary

end
write memory
```

### 🔹 R2

```bash
enable
configure terminal
hostname R2

interface GigabitEthernet0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 ip address 1.1.1.2 255.255.255.0
 no shutdown

interface Serial0/0/1
 ip address 2.1.1.2 255.255.255.0
 no shutdown

router eigrp 1
 network 1.1.1.0 0.0.0.255
 network 2.1.1.0 0.0.0.255
 network 192.168.2.0 0.0.0.255
 no auto-summary

end
write memory
```

---

## ✅ Verification

### 1️⃣ EIGRP Topology Table ⭐ (Main Focus)

**R1 → `show ip eigrp topology`**

```
IP-EIGRP Topology Table for AS 1/ID(192.168.1.1)

P 1.1.1.0/24, 1 successors, FD is 2169856
        via Connected, Serial0/0/0
P 2.1.1.0/24, 1 successors, FD is 2169856
        via Connected, Serial0/0/1
P 192.168.1.0/24, 1 successors, FD is 28160
        via Connected, GigabitEthernet0/0
P 192.168.2.0/24, 2 successors, FD is 2172416      ◄── 🔥 2 SUCCESSORS
        via 1.1.1.2 (2172416/28160), Serial0/0/0
        via 2.1.1.2 (2172416/28160), Serial0/0/1
```

**R2 → `show ip eigrp topology`**

```
IP-EIGRP Topology Table for AS 1/ID(192.168.2.1)

P 1.1.1.0/24, 1 successors, FD is 2169856
        via Connected, Serial0/0/0
P 2.1.1.0/24, 1 successors, FD is 2169856
        via Connected, Serial0/0/1
P 192.168.1.0/24, 2 successors, FD is 2172416      ◄── 🔥 2 SUCCESSORS
        via 1.1.1.1 (2172416/28160), Serial0/0/0
        via 2.1.1.1 (2172416/28160), Serial0/0/1
P 192.168.2.0/24, 1 successors, FD is 28160
        via Connected, GigabitEthernet0/0
```

#### 🔍 What to notice

| Observation | Explanation |
|:------------|:------------|
| `2 successors` | Two next hops share the **same lowest FD** |
| Both paths show `(2172416/28160)` | **FD = 2172416**, **RD = 28160** — identical on both paths |
| Both links are Serial (same bandwidth & delay) | Identical metric → **equal-cost paths** |
| State = **P** (Passive) | Route is stable and usable |
| Directly connected nets have 1 successor | Only one way to reach a local network |

#### 🧮 Metric Breakdown (per path)

```
EIGRP metric = 256 × ( 10⁷ / lowest BW(Kbps) + total delay (tens of µs) )

Lowest bandwidth : Serial = 1544 Kbps  → 10,000,000 / 1544 = 6476
Total delay      : Serial (2000) + remote Gi0/0 (10) = 2010

Metric = 256 × (6476 + 2010) = 2,172,416   ← identical on Link 1 and Link 2 ✅
```

---

### 2️⃣ Routing Table

**R1 → `show ip route`**

```
C    1.1.1.0/24   is directly connected, Serial0/0/0
C    2.1.1.0/24   is directly connected, Serial0/0/1
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
D    192.168.2.0/24 [90/2172416] via 1.1.1.2, 00:05:40, Serial0/0/0
                    [90/2172416] via 2.1.1.2, 00:05:38, Serial0/0/1
```

**R2 → `show ip route`**

```
C    1.1.1.0/24   is directly connected, Serial0/0/0
C    2.1.1.0/24   is directly connected, Serial0/0/1
D    192.168.1.0/24 [90/2172416] via 1.1.1.1, 00:07:19, Serial0/0/0
                    [90/2172416] via 2.1.1.1, 00:07:17, Serial0/0/1
C    192.168.2.0/24 is directly connected, GigabitEthernet0/0
```

| Field | Meaning |
|:------|:--------|
| **D** | Learned via EIGRP (internal) |
| **[90/2172416]** | AD **90** / metric 2172416 — equals the FD |
| **Two "via" lines** | **Both successors installed** → equal-cost load balancing active |

📌 **Every successor in the topology table becomes a route in the routing table.**

---

### 3️⃣ Recommended Extra Captures (for a stronger portfolio)

```bash
show ip eigrp neighbors        # expect 2 neighbors: 1.1.1.x via Se0/0/0 and 2.1.1.x via Se0/0/1
show ip protocols              # maximum-paths 4, variance 1, AD 90/170
show ip eigrp topology all-links
traceroute 192.168.2.1         # run several times from R1
```

---

## 🧠 Core Concepts

### 🔹 Successor
Next hop with the **lowest metric** to a destination — installed in the routing table. With **equal metrics, there can be multiple successors** (up to `maximum-paths`, default **4**).

### 🔹 Feasible Distance (FD) / Reported Distance (RD)
- **FD** — best total metric from *this* router to the destination
- **RD** — metric the *neighbor* reports for the same destination

### 🔹 Feasible Successor (FS)
A **backup** path with **RD < FD of the successor**. It is *not* in the routing table until needed.

### 🔹 Why aren't these paths Feasible Successors?
Because they're both **successors** — their metrics are *equal to* the FD, so both are already **actively forwarding**. A Feasible Successor only exists when a path is **worse than** the best path but still loop-free.

```
Successor            → best path(s), IN routing table
Feasible Successor   → loop-free backup (RD < FD), only in topology table
```

---

## ⚖️ Equal-Cost Load Balancing (ECMP)

When the routing table holds multiple equal-cost routes, the router **shares traffic** across them:

- ✅ Better **bandwidth utilization** — both links carry traffic
- ✅ **Redundancy** — one link failure doesn't stop traffic
- ✅ No extra configuration needed — EIGRP does it by default (up to 4 paths)

```bash
router eigrp 1
 maximum-paths 2      # limit/raise number of equal-cost paths (1–32 depending on IOS)
```

---

## 🔄 Failure Scenario — Why 2 Successors Matter

| Event | 1 Successor (previous lab) | 2 Successors (this lab) |
|:------|:--------------------------:|:-----------------------:|
| One link fails | Route lost → goes **Active** | Remaining successor **continues forwarding** |
| Queries sent | ✅ Yes (DUAL recalculates) | ❌ **No** — a successor still exists |
| Convergence | Slower | **Near-instant** |
| Route state | **A** (Active) | **P** (Passive) |

### 🧪 Try it yourself

```bash
R1(config)# interface Serial0/0/0
R1(config-if)# shutdown
R1# show ip route eigrp          ← only the 2.1.1.2 path remains
R1# show ip eigrp topology       ← 1 successors, route still Passive
```

---

## 🚀 Advanced: Unequal-Cost Load Balancing (`variance`)

EIGRP is unique among IGPs — it can load-balance across **unequal** paths if they are **feasible successors**:

```bash
router eigrp 1
 variance 2
```

**Rule:** a path is used if **its metric ≤ variance × best FD** **and** it passes the **feasibility condition (RD < FD)**.

> 💡 Challenge: change the bandwidth on one serial link (`bandwidth 512`) and use `variance` to use both links again.

---

## 🎓 Key Takeaways

1. 🔸 **Equal metrics → multiple successors** (`2 successors` in the topology table)
2. 🔸 Both successors are **installed in the routing table** as two `D` routes
3. 🔸 **FD = 2172416**, **RD = 28160** on both paths
4. 🔸 Traffic is **load-balanced** across equal-cost links
5. 🔸 Losing one link **does not make the route Active** — a successor remains
6. 🔸 Successor ≠ Feasible Successor: successors forward **now**, FS are **standby**
7. 🔸 Default `maximum-paths` is **4**; `variance` enables **unequal-cost** balancing
8. 🔸 Redundant design = faster convergence and a more stable network

---

## 🛠️ Useful Commands

```bash
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp topology all-links
show ip route eigrp
show ip protocols
show ip eigrp interfaces
show interfaces serial 0/0/0       # bandwidth / delay values
traceroute 192.168.2.1
```

---

## 📁 Repository Structure

```
📦 EIGRP-Successor-2-Paths
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ r1-topology-and-route.png
 ┃ ┗ 🖼️ r2-topology-and-route.png
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-Successor-2-Paths.pkt
```

---

## 🔗 Series Navigation

⬅️ **Previous:** EIGRP Successor – Single Path
➡️ **Next:** EIGRP Feasible Successor (unequal-cost paths)

`EIGRP` · `DUAL` · `Successor` · `Feasible Successor` · `ECMP` · `Load Balancing` · `Variance` · `Redundancy` · `Convergence`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If this lab helped you, star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>
