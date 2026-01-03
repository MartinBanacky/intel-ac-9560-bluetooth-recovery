Recover Missing Bluetooth on Intel Wireless-AC 9560 (Windows 10/11)

This repository documents a real-world, reproducible fix for a common but poorly documented Windows issue where Bluetooth completely disappears on systems with the Intel Wireless-AC 9560.

Symptoms: No Bluetooth toggle, no device in Device Manager, not even hidden/ghost entries visible.
Common fixes (reinstalling drivers, restarting services, power drain) fail.

This guide explains:
- Why this happens
- Why standard troubleshooting doesn't work
- A low-level driver-store reset that restores Bluetooth without reinstalling Windows or replacing hardware

Affected Systems
----------------
This issue typically occurs on:
- Intel Wireless-AC 9560 (Wi-Fi + Bluetooth combo card)
- Windows 10 and Windows 11
- Laptops with aggressive thermal/power limits (common in 2018–2020 gaming or ultrathin performance models)

Symptoms
--------
You may experience one or more of the following:
- Bluetooth toggle missing in Settings > Devices
- Bluetooth section entirely absent in Device Manager
- Troubleshooter reports "Device does not have Bluetooth" or "Bluetooth not available"
- Only greyed-out (hidden) Bluetooth devices visible in Device Manager
- Reinstalling drivers has no effect
- Simple power drain/reset does not help

Root Cause
----------
The Intel AC 9560 is a combo card exposing:
- Wi-Fi via PCIe
- Bluetooth via an internal USB interface

After a firmware crash, sudden power loss, or thermal throttling:
- Windows corrupts entries in the driver store
- Old Intel Wi-Fi drivers (netwtw*.inf) remain bound
- The Bluetooth USB interface fails to re-enumerate
- Windows "remembers" the broken state permanently

In most cases, this is NOT a hardware failure.

Why Common Fixes Fail
---------------------
Attempt                        | Reason It Fails
-------------------------------|-------------------------------------------------------
Restart Bluetooth services     | Device never enumerates
Reinstall Bluetooth drivers    | Blocked by corrupted Wi-Fi driver bindings
Power drain / hardware reset   | Driver-store corruption persists
Windows "Reset this PC"        | Often preserves the corrupted driver store

The Fix: Driver-Store Reset + Forced Re-enumeration
--------------------------------------------------
This method removes all Intel Wi-Fi driver packages, forcing Windows to rediscover the card and its Bluetooth USB interface from scratch.

Before You Begin
----------------
Important warnings:
- This procedure temporarily removes Wi-Fi drivers
- You will lose Wi-Fi connectivity until drivers are reinstalled (or Windows falls back to inbox drivers)
- Have an Ethernet connection or USB tethering ready if needed

Recovery Procedure
------------------

Step 1: Confirm Hidden Bluetooth Devices
1. Open Device Manager
2. Go to View → Show hidden devices
3. Look for greyed-out Bluetooth entries

→ Presence of ghost devices confirms driver-store corruption.

Step 2: List Installed Intel Wi-Fi Drivers
Open Command Prompt as Administrator and run:

pnputil /enum-drivers | findstr /i netwtw

Example output:
Original Name: netwtw02.inf
Original Name: netwtw04.inf
Original Name: netwtw06.inf
Original Name: netwtw08.inf

Step 3: Identify Published Driver Names
Export the full driver list:

pnputil /enum-drivers > drivers.txt
notepad drivers.txt

For each netwtwXX.inf, note the corresponding Published Name (e.g., oem12.inf, oem45.inf, etc.).

Step 4: Delete All Intel Wi-Fi Driver Packages
For each identified oemXX.inf, run:

pnputil /delete-driver oemXX.inf /uninstall /force

Repeat until the following returns no results:

pnputil /enum-drivers | findstr /i netwtw

Step 5: Reboot (Critical)
Reboot the system immediately.
- Do NOT install any drivers yet
- Do NOT run Windows Update

Step 6: Check Results After Reboot
1. Open Device Manager
2. Enable View → Show hidden devices

Success (Most Common)
- Bluetooth category reappears
- Bluetooth devices are present and functional
- Bluetooth toggle returns in Settings
- Bluetooth works without installing any additional drivers

This confirms the hardware and firmware are healthy — the issue was purely driver-store corruption.

Failure (Rare)
- No Bluetooth devices appear at all
This usually indicates actual hardware failure of the Bluetooth USB interface.

After Recovery
--------------
- You do not need to install drivers immediately
- Windows inbox drivers are sufficient for:
  - Bluetooth pairing
  - HID devices (mice, keyboards, headphones)
  - Stable daily use

Warning: Installing the wrong or outdated Intel drivers can reintroduce the problem.

Recommendations
- Avoid Intel Driver & Support Assistant
- Avoid optional Windows Update drivers for Intel Wireless
- If you must install newer drivers later:
  1. Install Wi-Fi driver first → Reboot
  2. Install Bluetooth driver → Reboot

Why This Works
--------------
Removing all netwtw*.inf packages forces Windows to:
- Forget corrupted driver bindings
- Re-enumerate the Intel card firmware from scratch
- Properly detect and expose the internal Bluetooth USB interface

This is a true low-level reset, not a temporary workaround.

Good luck! If this guide helped you, consider starring the repo.
