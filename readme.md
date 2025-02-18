# GNS3 DHCP Server Project

## Project Description

This project demonstrates the configuration of a DHCP (Dynamic Host Configuration Protocol) server on a Cisco router within a GNS3 (Graphical Network Simulator-3) environment.  It showcases how a router can be set up to automatically assign IP addresses to end devices (simulated PCs) connected to a local area network (LAN) through a switch.

This project is designed for beginners learning about DHCP and network configuration in GNS3. It provides a hands-on example of setting up a fundamental network service.

## Topology 


**Devices Used:**

*   **Router:** Cisco Router.
*   **Switch:** Ethernet Switch (Layer 2 Switch - labeled Switch1 in the diagram).
*   **End Devices:** VPCS (Virtual PC Simulator) - 4 instances (PC1, PC2, PC3, PC4).

## Configuration Steps

### 1. Router R1 Configuration (DHCP Server)

1.  **Access Router Console:** Open the console of Router R1 in GNS3 by right-clicking on the router and selecting "Console".
2.  **Enter Enable Mode:**
    ```
    Router> enable
    Router#
    ```
    Type `enable` and press Enter. This will take you to privileged EXEC mode, indicated by the `#` prompt.
3.  **Enter Configuration Mode:**
    ```
    Router# configure terminal
    Router(config)#
    ```
    Type `configure terminal` or `conf t` (short form) and press Enter. This will take you to global configuration mode, indicated by the `(config)#` prompt.
4.  **Configure Interface f0/0 (connected to the Switch):**
    ```
    Router(config)# interface FastEthernet 0/0
    Router(config-if)# ip address 192.168.1.1 255.255.255.0
    Router(config-if)# no shutdown
    Router(config-if)# exit
    ```
    *   `interface FastEthernet 0/0`:  Selects the FastEthernet 0/0 interface for configuration. Make sure this interface matches the one you connected to the switch in your GNS3 topology.
    *   `ip address 192.168.1.1 255.255.255.0`: Assigns the static IP address `192.168.1.1` and subnet mask `255.255.255.0` to this interface. This will be the gateway address for the LAN and the IP address of the DHCP server on this network.
    *   `no shutdown`:  Activates the interface, bringing it up.
    *   `exit`:  Exits interface configuration mode and returns to global configuration mode.
5.  **Configure DHCP Pool:**
    ```
    Router(config)# ip dhcp pool LAN-POOL
    Router(dhcp-config)# network 192.168.1.0 255.255.255.0
    Router(dhcp-config)# default-router 192.168.1.1
    Router(dhcp-config)# dns-server 8.8.8.8
    Router(dhcp-config)# exit
    ```
    *   `ip dhcp pool LAN-POOL`: Creates a DHCP pool named "LAN-POOL". This is just a name; you can choose another name if you prefer.
    *   `network 192.168.1.0 255.255.255.0`:  **Defines the network for DHCP address assignment.**  This command tells the DHCP server which network range to manage. It should match the network of the router's interface (`192.168.1.0/24` in this case).  IP addresses assigned to clients will be from this network.
    *   `default-router 192.168.1.1`: Sets the default gateway address that the DHCP server will provide to clients. This is essential so that devices on the LAN know how to reach networks outside their local network. It's set to the router's interface IP address (`192.168.1.1`).
    *   `dns-server 8.8.8.8`: (Optional) Configures the DNS server IP address that DHCP clients will receive. We use Google Public DNS (`8.8.8.8`) here. You can use a different DNS server or remove this line if you don't want to specify a DNS server via DHCP.
    *   `exit`: Exits DHCP pool configuration mode and returns to global configuration mode.
6.  **Exit Configuration Mode and Save Configuration:**
    ```
    Router(config)# exit
    Router# copy running-config startup-config
    ```
    or simply:
    ```
    Router# wr
    ```
    *   `exit`: Exits global configuration mode and returns to privileged EXEC mode.
    *   `copy running-config startup-config` or `wr`: Saves the current running configuration to the startup configuration file. This ensures that the router configuration is saved and will be loaded when the router restarts or when you reopen the GNS3 project.

### 2. VPCS (PC1, PC2, PC3, PC4) Configuration (DHCP Client)

1.  **Access VPCS Consoles:** Open the console for each VPCS device (PC1, PC2, PC3, PC4) in GNS3 by right-clicking on each VPCS device and selecting "Console".
2.  **Request IP Address via DHCP:** In each VPCS console, type the following command and press Enter:
    ```
    VPCS> ip dhcp
    ```
    *   `ip dhcp`: This command instructs the VPCS device to send a DHCP request to obtain an IP address and other network configuration parameters from the DHCP server on the network.

## Verification Steps: How to Check if DHCP is Working

After configuring the router as a DHCP server and issuing the `ip dhcp` command on the VPCS devices, you need to verify that DHCP is working correctly. Refer to the [Troubleshooting Guide](TROUBLESHOOTING.md) for detailed verification steps and troubleshooting if you encounter any issues.  The guide explains how to:

1.  **Verify IP Address Configuration on VPCS Devices:** Check that each VPCS has received an IP address, subnet mask, gateway, and (optionally) DNS server information via DHCP using the `show ip` command in the VPCS consoles.
2.  **Verify DHCP Bindings on Router R1 (DHCP Server Verification):** Use the `show ip dhcp binding` command on Router R1 to confirm that the router has assigned IP addresses to the VPCS devices.
3.  **Test Network Connectivity with Ping:** Use the `ping` command from VPCS devices to test connectivity to the router (gateway) and to other VPCS devices on the LAN.

## Troubleshooting

If you encounter any issues during setup or verification, please refer to the [Troubleshooting Guide](Troubleshoot.md) for common problems and solutions.

## VPCS Persistence Note

**Important:** VPCS (Virtual PC Simulator) in GNS3 is designed to be lightweight and resource-efficient. By default, VPCS does **not** save its configuration when you close and reopen a GNS3 project. This means that when you restart your GNS3 project, you will need to issue the `ip dhcp` command again in each VPCS console to get them to request and obtain new IP addresses. This is normal behavior.

## GNS3 Project File

For easier setup and to replicate this project in GNS3, you can include the `.gns3project` file in this repository. This will allow others to simply open the project in their GNS3 environment and explore the configuration.

## Further Exploration

*   **Add more VPCS devices:** Connect more VPCS devices to the switch and observe them receiving DHCP addresses.
*   **Experiment with DHCP Lease Time:**  *(This would require adding the `lease` command to the DHCP pool configuration on the router - explore this advanced option if interested.)*
*   **DHCP Reservations:** *(Explore configuring DHCP reservations to assign specific IP addresses to certain devices based on their MAC addresses - an intermediate to advanced DHCP topic.)*
*   **DHCP Options:** *(Investigate other DHCP options you can configure, such as different DNS servers, NTP servers, or other client configuration parameters - more advanced DHCP configuration.)*

## Author

Vedant

Feel free to contribute to this project or use it as a starting point for your GNS3 learning!
