# Hardware-Assisted Cmplog via Intel PEBS

A high-performance, non-intrusive comparison logging engine for closed-source binary fuzzing. This project replaces dynamic binary translation (AFL++ QEMU mode) with hardware-enforced **Intel Processor Event-Based Sampling (PEBS)** and a 1D sequential DS Save Area buffer, eliminating emulation overhead and hash table cache thrashing in Input-to-State (I2S / RedQueen) analysis.

---

## Overview

AFL++ Cmplog fundamentally improves magic-byte and checksum solving by capturing operand values at conditional branches. However, applying Cmplog to closed-source, proprietary binaries currently relies on Dynamic Binary Translation (DBT) engines like QEMU or runtime instrumentation like Frida:

* **Emulation Tax:** QEMU user-mode translation incurs a 3x–10x execution speed penalty.
* **Cache Pollution & Collisions:** Standard AFL++ `cmp_map` uses a fixed-size 2D hash structure (`CMP_MAP_W * CMP_MAP_H`). Under tight loops, constant hash collisions cause slot dropouts and heavy L1/L2 data cache thrashing.

This project leverages **Intel PEBS** hardware execution to capture register contents at comparison instructions directly into an OS-managed linear memory buffer (DS Save Area). By writing operands sequentially via CPU silicon rather than software trampoline hooks or DBT blocks, this architecture achieves native-speed comparison extraction with zero binary modification.
