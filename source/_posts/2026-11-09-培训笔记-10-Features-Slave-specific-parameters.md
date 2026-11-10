---
title: 培训笔记-10-Features-Slave specific parameters
date: 2026-11-09 10:28:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

**Slave specific parameters**

- Protocol – AXI4/3, AHB-Lite Master, AHB-Lite Mirrored Slave, APB2/3/4
- Data Width – 32, 64, 128 or 256 bits
- Clock domain
- Address Width – 32 to 64 bits
- Read acceptance capability – 1 to 127 (or unlimited) transactions
- Write acceptance capability – 1 to 32 transactions
- TrustZone (security) – Secure/Non-secure/Boot-secure
- Multi-region
- Brings out AWREGION and ARREGION signals at the top level
- Output Transaction QoS Value
- USER Signals – 0 to 256 bits
