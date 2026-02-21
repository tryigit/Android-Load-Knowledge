# CLI Diagnostic Tools for Android

For a Systems Engineer, the command line interface (CLI) is the surgical theater. While GUI apps are convenient, they consume significant resources (CPU/GPU) to render their interfaces, which can skew your measurements (the "Observer Effect").

CLI tools have near-zero overhead, allowing you to observe the system in its natural state.

## 1. The Basics: `uptime`

The simplest way to check system health.

```bash
su -c uptime
```

**Output:**
`12:34:56 up 10 days, 2:30, load average: 4.12, 3.88, 3.50`

### Interpretation
*   **1min (4.12):** Immediate system state. If this is > (CPU Cores * 2), you have a bottleneck.
*   **5min (3.88):** A short-term trend.
*   **15min (3.50):** The long-term baseline.

> [!TIP]
> If `1min` > `15min`, the load is **increasing** (something just happened).
> If `1min` < `15min`, the load is **decreasing** (the system is recovering).

---

## 2. Process Analysis: `top`

The standard Linux process viewer. On Android, the `toybox` version is limited but powerful if you know the flags.

### The "Golden" Command
To see the top 10 processes consuming the most CPU right now:

```bash
su -c top -m 10 -s cpu
```

| Flag | Function |
| :--- | :--- |
| `-m 10` | Limit output to the top 10 entries (reduces clutter). |
| `-s cpu` | Sort by CPU usage (default is often PID or VSS). |
| `-d 1` | Update every 1 second. |

### Reading the Output
Look for the `S` (State) column:
*   `R`: Running (Healthy).
*   `S`: Sleeping (Healthy).
*   `D`: **Uninterruptible Sleep** (I/O Wait / Kernel Lock). **If you see many 'D' states, your storage or kernel is the problem, not the app.**
*   `Z`: Zombie (Parent process didn't clean up).

---

## 3. The Android Special: `dumpsys cpuinfo`

Unlike `top`, which shows an instantaneous snapshot, `dumpsys` aggregates data over a short window (usually the last few seconds/minutes). It is aware of Android packages.

```bash
su -c dumpsys cpuinfo
```

**Why it's useful:**
It breaks down usage by:
1.  **User Stack:** (App logic)
2.  **Kernel Stack:** (System calls)

If an app shows 5% User and 40% Kernel, that app is spamming system calls (e.g., trying to open a file 10,000 times a second).

---

## 4. Virtual Memory Statistics: `vmstat`

Essential for diagnosing **Memory Thrashing** and **Context Switching**.

```bash
su -c vmstat 1
```
*(Prints a new line every 1 second)*

### Key Columns:
*   **`r` (run):** Processes waiting for CPU. If this is consistently high, you need a faster CPU or fewer background apps.
*   **`swpd`:** Swap used (zRAM on Android).
*   **`bi` / `bo` (Blocks In/Out):** Disk Read/Write operations. High numbers here = High I/O Load.
*   **`in` (Interrupts):** Hardware demanding attention.
*   **`cs` (Context Switches):** The CPU switching tasks.
    *   **Normal:** 500 - 5,000.
    *   **Bad:** > 15,000 (The CPU is spending more time *switching* tasks than *doing* them).

---

## 5. Advanced Monitoring: `htop`

`htop` is not built-in but can be installed via **Termux** or a **Magisk Module**.

It provides a color-coded, visual bar graph of per-core usage.
*   **Green:** User processes.
*   **Red:** Kernel threads.
*   **Blue:** Low-priority threads.

**Pro Tip:** Press `F2` (Setup) -> Display Options -> Check "Detailed CPU Time". This breaks down usage into IO-Wait, IRQ, and SoftIRQ.

---

## Summary Workflow

1.  **Check Global Health:** `uptime` -> Is load high?
2.  **Identify Culprit:** `top -m 5` -> Is it a specific app?
3.  **Check Type:** `vmstat 1` -> Is it CPU crunching (high `us`) or I/O waiting (high `wa`/`bi`/`bo`)?
4.  **Deep Dive:** If it's a specific app, use [Advanced Profiling](./03-advanced-profiling-and-gui.md).
