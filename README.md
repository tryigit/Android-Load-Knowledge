# Android System Load & Performance Diagnostics

![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Platform](https://img.shields.io/badge/Platform-Android-green.svg)
![Status](https://img.shields.io/badge/Status-Maintained-brightgreen.svg)

**A professional, technical guide to diagnosing Android system performance, understanding the Linux kernel scheduler, and identifying the root causes of lag and battery drain.**

---

## Introduction

Most Android "optimization" guides focus on placebos: clearing cache, killing background apps, or installing "performance modules."

This repository takes a **Systems Engineering approach**. We analyze the Android OS at the kernel level to understand:
*   What "Load Average" actually means (hint: it's not just CPU usage).
*   Why your phone lags even when the CPU is idle (I/O Wait).
*   How to scientifically prove which app is causing battery drain using tracing tools.

## Documentation

### [1. Understanding Load Average](./docs/01-understanding-load-average.md)
Stop guessing. Learn the difference between **CPU Usage**, **Run Queues**, and **Uninterruptible Sleep (D-State)**. This is the foundation of all diagnostics.

### [2. CLI Diagnostic Tools](./docs/02-cli-diagnostic-tools.md)
Master the command line. A deep dive into `uptime`, `top`, `vmstat`, and `dumpsys cpuinfo`. Learn how to read the kernel's vitals with zero overhead.

### [3. Advanced Profiling & GUI Tools](./docs/03-advanced-profiling-and-gui.md)
From enthusiast apps like **Scene** and **Franco Kernel Manager** to professional engineering tools like **Perfetto** and **Systrace**. Visualize the stutter.

### [4. The Custom Kernel Dilemma](./docs/04-the-custom-kernel-dilemma.md)
Why flashing a custom kernel might be hurting your performance. An analysis of **Energy Aware Scheduling (EAS)**, out-of-tree drivers, and the stability risks of "overclocking."

---

## Quick Start

If you have root access (`su`), run this command to see your system's heartbeat immediately:

```bash
# View 1min, 5min, 15min load averages
su -c uptime
```

If the first number is higher than your CPU core count (e.g., > 8.0 on an octa-core device), your system is saturated.

To see *what* is causing it (sorted by CPU usage):

```bash
su -c top -m 10 -s cpu
```

---

## Community & Contact

Join our community for high-level discussions on Android internals, optimization, and development.

*   **Telegram Channel:** [Cleveres Tech](https://t.me/cleverestech)

---

## License

This repository is licensed under the MIT License. feel free to fork, contribute, and share.
