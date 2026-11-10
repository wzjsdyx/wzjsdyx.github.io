---
title: 培训笔记-9-Features-Master specific parameters
date: 2026-11-09 10:27:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

**Master specific parameters**

- Protocol – AXI4/3, AXI3, AHB-Lite Mirrored Master or AHB-Lite Slave
- Data Width – 32, 64, 128 or 256 bits
- Clock domain
- Address Width – 32 to 64 bits
- Read issuing capability – 1 to 127 transactions
- Write issuing capability – 1 to 32 transactions
- TrustZone (security) – Secure/Non-secure/Per-access Secure
- ID width – 0 to 16 bits
- QoS Priority Type
  - Fixed – Specify a value (0 – 15)
  - Programmable – Specify a post reset value (0 – 15)
  - From Master (Master supplies 4-bit priority value on AWQOS and ARQOS)
- USER Signals – 0 to 256 bits

