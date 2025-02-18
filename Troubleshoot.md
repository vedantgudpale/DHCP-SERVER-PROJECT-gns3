# Troubleshooting Guide for GNS3 DHCP Server Project

This guide provides solutions to common problems you might encounter while setting up and running the GNS3 DHCP Server Project.

---

## Problem 1: VPCS Devices are Not Getting IP Addresses (No DHCP IP Address Assignment)

**Symptoms:**

*   You run `ip dhcp` in the VPCS console, but it seems to hang or does not display a successful DHCP IP address assignment message.
*   When you type `show ip` or `ip` in the VPCS console, you see `IP: 0.0.0.0` or no IP address information is displayed.
*   VPCS devices cannot ping the router's interface IP address (`192.168.1.1`).

**Possible Causes and Solutions:**

1.  **Router R1 is Not Started or Not Functioning as DHCP Server:**
    *   **Solution:**
        *   **Check Router Status in GNS3:** Ensure that Router R1 is started in GNS3. Look for the green triangle icon next to the router's name. If it's not green, right-click on Router R1 and select "Start".
        *   **Verify DHCP Server Configuration on Router R1:**
            *   Access the Router R1 console.
            *   Enter privileged EXEC mode (`enable`).
            *   Type `show running-config | section dhcp` and press Enter.
            *   **Carefully review the DHCP pool configuration:**
                *   Is the `ip dhcp pool LAN-POOL` configured?
                *   Is the `network 192.168.1.0 255.255.255.0` command present and correct? **(Crucial for defining the network)**
                *   Is the `default-router 192.168.1.1` command present and correct? **(Crucial for gateway assignment)**
                *   Is the `dns-server 8.8.8.8` command present and correct?
            *   If any part of the DHCP configuration is missing or incorrect, reconfigure it as per the [Configuration Steps in README.md](README.md#1-router-r1-configuration-dhcp-server). Pay close attention to the `network` and `default-router` commands.
            *   **Save the Router Configuration:**  After making changes, remember to save the router configuration using `copy running-config startup-config` or `wr`.

2.  **Router Interface (f0/0) is Not Configured or Down:**
    *   **Solution:**
        *   **Check Router Interface Configuration and Status:**
            *   Access the Router R1 console.
            *   Enter privileged EXEC mode (`enable`).
            *   Type `show ip interface brief` and press Enter.
            *   **Verify the `FastEthernet0/0` interface status:**
                *   **Status and Protocol should both be "up"**. If they are "administratively down" or "down", the interface is not active.
                *   **IP Address should be `192.168.1.1`**.
            *   **If the interface is down or IP address is incorrect:**
                *   Enter configuration mode: `configure terminal`
                *   Enter interface configuration mode: `interface FastEthernet 0/0`
                *   Ensure the IP address is set: `ip address 192.168.1.1 255.255.255.0`
                *   If the interface is "administratively down", use `no shutdown` to activate it.
                *   Exit configuration modes and save the router configuration.

3.  **Connectivity Issues Between VPCS, Switch, and Router:**
    *   **Solution:**
        *   **Verify GNS3 Topology Connections:** Double-check that you have correctly connected:
            *   Router R1's `FastEthernet0/0` to Switch1's `Ethernet0/1` (or similar port).
            *   Each VPCS device (PC1, PC2, PC3, PC4)'s `Ethernet0` to a different port on Switch1 (e.g., `Ethernet0/2`, `Ethernet0/3`, `Ethernet0/4`, `Ethernet0/5`).
            *   **In GNS3, look at the connection lines.** They should be solid green lines when devices are started and interfaces are up. If they are dotted or broken, there might be a connection issue. Delete the link and recreate it if necessary.
        *   **Check Switch Status (Basic Check):** While less common for basic Ethernet switches in GNS3 to fail, ensure the "Switch1" device is also started in GNS3 (green triangle icon).

4.  **DHCP Pool Network Mismatch:**
    *   **Solution:**
        *   **Ensure DHCP `network` Command Matches Router Interface Network:**  Double-check that the `network 192.168.1.0 255.255.255.0` command in your DHCP pool configuration on Router R1 **exactly matches** the network that the Router's `FastEthernet0/0` interface is part of (`192.168.1.0/24` in this project).  If there's a mismatch, DHCP might not work for devices on the intended LAN. Correct the `network` command in the DHCP pool configuration if needed and save the router configuration.

5.  **VPCS Network Settings Issue (Less Common):**
    *   **Solution:**
        *   **Restart VPCS Devices:** Sometimes, restarting the VPCS devices in GNS3 can resolve temporary issues. Right-click on each VPCS device and select "Stop", then right-click again and select "Start". After they restart, try `ip dhcp` again in their consoles.
        *   **Restart GNS3 (as a last resort):** In rare cases, restarting the GNS3 application itself might help resolve underlying issues.

---

## Problem 2: VPCS Devices Get IP Addresses, But Cannot Ping the Router (Gateway) or Other VPCS Devices

**Symptoms:**

*   VPCS devices successfully obtain IP addresses via DHCP.
*   `show ip` in VPCS consoles shows IP address, subnet mask, gateway, etc.
*   Pinging the router's interface IP address (`192.168.1.1`) from a VPCS fails (no ping replies).
*   Pinging other VPCS devices from one VPCS also fails.

**Possible Causes and Solutions:**

1.  **Incorrect Gateway IP Address on VPCS:**
    *   **Solution:**
        *   **Verify Gateway on VPCS:** In a VPCS console, type `show ip` or `ip` and press Enter. Check the "GW" (Gateway) field. It **must** be `192.168.1.1`.
        *   **Verify DHCP `default-router` Configuration:** On Router R1, check the DHCP pool configuration (`show running-config | section dhcp`). Ensure that the `default-router 192.168.1.1` command is correctly configured. If incorrect, fix it and save the router configuration. Then, on the VPCS devices, try releasing and renewing their DHCP leases (you might need to restart VPCS or try `ip release` followed by `ip dhcp`).

2.  **Router Interface (f0/0) is Down (Even if IP is Configured):**
    *   **Solution:** (Similar to Problem 1, Cause 2)
        *   **Check Router Interface Status:** Use `show ip interface brief` on Router R1 to confirm that `FastEthernet0/0` status and protocol are both "up".  If not, use `no shutdown` in interface configuration mode to activate it.

3.  **Incorrect IP Addresses or Subnet Masks on Router or VPCS:**
    *   **Solution:**
        *   **Double-Check IP Addresses:** Carefully re-examine the IP address configurations:
            *   Router R1 `FastEthernet0/0`: `192.168.1.1 255.255.255.0`
            *   DHCP Pool `network`: `192.168.1.0 255.255.255.0`
            *   VPCS devices should be getting IP addresses in the `192.168.1.0/24` network (e.g., `192.168.1.x` with mask `255.255.255.0`).
        *   **Correct any IP address or subnet mask errors** in the router or DHCP pool configurations. Save router configuration and have VPCS devices renew their DHCP leases.

---

## Problem 3:  "VPCS Persistence Note" - Having to Run `ip dhcp` After Reopening Project

**Symptom:**

*   You save and close your GNS3 project. When you reopen it later, the VPCS devices have lost their IP addresses, and you need to run `ip dhcp` again in each VPCS console to get them to obtain new IP addresses.

**Explanation and Expected Behavior:**

*   **VPCS is Stateless by Design:** This is **normal and expected behavior** for VPCS (Virtual PC Simulator) in GNS3. VPCS is designed to be lightweight and resource-efficient. By default, it does not save its configuration across GNS3 project sessions.
*   **Solution:**
    *   **Run `ip dhcp` Again:** The simplest solution is to just accept this behavior. After reopening your GNS3 project and starting the VPCS devices, simply run the `ip dhcp` command in each VPCS console. This is a quick and easy step to re-establish IP connectivity for your VPCS devices.

---

**If you are still experiencing problems after following these troubleshooting steps, please double-check all configuration steps in the [README.md](README.md) file and carefully review your GNS3 topology.**

If you are still stuck, consider:

*   **Re-reading the Project Instructions and Configuration Steps.**
*   **Searching online for GNS3 DHCP troubleshooting tips.**
*   **Describing your specific problem in detail in a forum or online community related to GNS3 or networking.** When asking for help, provide specific details about your topology, configurations, and the exact symptoms you are seeing.

Good luck troubleshooting!
