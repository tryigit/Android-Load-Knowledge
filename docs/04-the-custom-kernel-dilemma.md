# The Custom Kernel Dilemma

In the Android enthusiast community, flashing a custom kernel is often seen as the ultimate optimization step. The promise of "better battery life," "overclocking," and "underservolting" is alluring.

However, from a Systems Engineering perspective, custom kernels are frequently the **primary cause** of system instability, high load averages, and inexplicable battery drain.

This document explains why the stock kernel—despite its conservative tuning—is usually superior for stability and longevity.

## 1. Energy Aware Scheduling (EAS) Disruption

Modern Android devices (since the Pixel 1/2 era) use **EAS**.
*   **How it works:** The kernel has a precise "Energy Model" of the specific CPU silicon. It knows exactly how much power the "Little" cluster consumes at 300MHz vs. the "Big" cluster at 2.0GHz.
*   **The Problem:** Custom kernel developers often "tweak" the scheduler tunables or introduce legacy governors (like `interactive` or `ondemand`) that are incompatible with EAS.
*   **The Result:** The scheduler makes wrong decisions, placing heavy tasks on weak cores (lag) or light tasks on power-hungry cores (drain). You break the mathematical model the hardware was designed for.

## 2. Out-of-Tree Drivers & "Dirty" Cherry-Picks

The Linux kernel is monolithic. Drivers (WiFi, Audio, GPU) are compiled directly into it.
*   **Stock Kernel:** Verified by the OEM (Samsung, Google, Xiaomi) against the specific hardware revision.
*   **Custom Kernel:** Often includes thousands of commits "cherry-picked" from other devices or newer Linux versions.
*   **The Risk:** A commit that improves performance on a Snapdragon 8 Gen 1 might introduce a race condition on a Snapdragon 888. These "merge conflicts" are often resolved blindly, leading to silent data corruption or random reboots (Kernel Panics).

## 3. The "Performance" Placebo

Many custom kernels achieve "smoothness" by cheating.
*   **Disabling Thermal Throttling:** The phone feels faster because it doesn't slow down when hot.
    *   **Consequence:** The battery degrades rapidly due to heat; the motherboard solder joints can weaken over time.
*   **Pinning Frequencies:** Forcing the CPU to stay at high frequencies.
    *   **Consequence:** Your load average drops (because queues clear instantly), but your active power consumption skyrockets.

## 4. Security Compromises

To achieve compatibility or ease of development, some kernels:
1.  **Disable SELinux:** Setting it to `Permissive`. This removes the sandbox that prevents a rogue app from taking over the root system.
2.  **Disable Mitigation Patches:** Turning off Spectre/Meltdown mitigations to gain 1-2% performance.

## Conclusion: When to use a Custom Kernel?

Use a custom kernel **only** if:
1.  You need a specific feature not present in stock (e.g., specific USB driver support, docker support).
2.  The kernel developer is reputable and prioritizes **upstream stability** over "ricing" (adding random features).

**For 99% of users:** The stock kernel, combined with userspace optimization (removing bloatware), provides the best balance of performance, battery, and reliability.
