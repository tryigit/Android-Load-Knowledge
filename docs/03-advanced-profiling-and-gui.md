# Advanced Profiling & GUI Tools

While CLI tools provide raw data, visual tools allow you to spot patterns, correlations, and anomalies that are impossible to see in a scrolling terminal. This section covers both high-end enthusiast apps and professional engineering profilers.

## 1. Enthusiast GUI Tools

These apps are excellent for real-time monitoring on a rooted device.

### Scene (formerly Scene5/Scene6)
**Scene** is arguably the most comprehensive monitoring dashboard for Android.
*   **Key Feature:** The "Monitor" overlay. It shows real-time load per-core, GPU usage, temperature, and current current (mA).
*   **Usage:** Great for spotting thermal throttling in real-time. If you see temperatures hit 45°C and frequencies drop simultaneously, you've confirmed a thermal event.

### Franco Kernel Manager (FKM)
A veteran tool in the custom kernel community.
*   **Key Feature:** Detailed battery stats and per-app profiles.
*   **Usage:** Use FKM to track "Active Drain" vs "Idle Drain" percentages. If Idle Drain > 1-2% per hour, you have a wakelock issue.

> [!WARNING]
> **The Observer Effect:** GUI tools draw their own frames. Running a heavy overlay like Scene *will* increase the system load average by 0.5 - 1.0. Always account for this overhead.

---

## 2. Professional Engineering Tools: Perfetto & Systrace

When you need to know *exactly* why a frame dropped or why a specific tap took 200ms to register, you need tracing.

### Perfetto (The Modern Standard)
Perfetto is the platform-wide tracing tool introduced in Android 9 (Pie) and is now the standard, replacing Systrace. It records kernel-level and userspace-level events.

#### How to capture a trace (On-Device):
1.  Enable **Developer Options**.
2.  Go to **System Tracing**.
3.  Enable categories: `freq`, `sched`, `gfx`, `view`, `input`.
4.  Tap **"Record trace"**.
5.  Reproduce the lag/issue.
6.  Stop recording and share the `.perfetto-trace` file to your PC.

#### Analysis
Open the trace file in [ui.perfetto.dev](https://ui.perfetto.dev).
*   **CPU Slices:** See exactly which process was on which core at any microsecond.
*   **Binder Transactions:** detailed view of IPC (Inter-Process Communication). If an app is waiting on `system_server`, you will see the dependency link here.

### Systrace (Legacy)
Still useful for older devices (Android 8 and below). It generates an HTML file.
Command:
```bash
python systrace.py -o mytrace.html sched gfx view wm
```

---

## 3. What to Look For in a Trace

When analyzing a trace for "Load" issues, look for these red flags:

### A. The `kswapd0` Daemon
If `kswapd0` is constantly active (taking up a significant CPU slice), your device is **thrashing**.
*   **Meaning:** RAM is full. The kernel is frantically trying to compress pages (zRAM) or discard cached files to free up memory for the foreground app.
*   **Result:** Extreme lag and high load.

### B. Binder Contention
Android apps talk to the system via **Binder**.
*   **Scenario:** App A asks the System Server to open the camera.
*   **Bottleneck:** If System Server is busy (high load), App A sleeps (blocked).
*   **Visual:** In Perfetto, you will see a long bar for the app process, but it's in a "Sleeping" state, waiting for a reply from `system_server`.

### C. Janky Frames
Look for the "Frames" row.
*   **Green:** Good (Under 16ms for 60Hz).
*   **Red:** Bad (Missed the VSYNC deadline).
*   **Correlation:** Check what was running on the CPU *during* that red frame. Was it a background backup? Garbage collection (GC)? That is your culprit.
