# GNS3 DHCP Server Project

## Project Description

This project demonstrates the configuration of a DHCP (Dynamic Host Configuration Protocol) server on a Cisco router within a GNS3 (Graphical Network Simulator-3) environment.  It showcases how a router can be set up to automatically assign IP addresses to end devices (simulated PCs) connected to a local area network (LAN) through a switch.

This project is designed for beginners learning about DHCP and network configuration in GNS3. It provides a hands-on example of setting up a fundamental network service.

## Topology

**Devices Used:**

*   **Router:** Cisco Router
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
    *   `ip address 192.168.1.1 255.255.255.0`: Assigns the static IP address `192.168.1.1` and subnet mask `255.255.255.0` to this interface. This will be the gateway address for devices on the LAN.
    *   `no shutdown`:  Activates the interface, bringing it up.
    *   `exit`:  Exits interface configuration mode and returns to global configuration mode.
5.  **Configure DHCP Pool:**
    ```
    Router(config)# ip dhcp pool LAN-POOL
    Router(dhcp-config)# network 192.168.1.0 255.255.255.0
    Router(dhcp-config)# range 192.168.1.10 192.168.1.100
    Router(dhcp-config)# default-router 192.168.1.1
    Router(dhcp-config)# dns-server 8.8.8.8
    Router(dhcp-config)# exit
    ```
    *   `ip dhcp pool LAN-POOL`: Creates a DHCP pool named "LAN-POOL". This is just a name; you can choose another name if you prefer.
    *   `network 192.168.1.0 255.255.255.0`:  Defines the network address and subnet mask for the network segment where DHCP will be used. This should match the network of the router's interface (`192.168.1.0/24` in this case).
    *   `range 192.168.1.10 192.168.1.100`: Specifies the range of IP addresses the DHCP server will assign to clients, from `192.168.1.10` to `192.168.1.100`.
    *   `default-router 192.168.1.1`: Sets the default gateway address that the DHCP server will provide to clients. This should be the IP address of the router's interface on the LAN (`192.168.1.1`).
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
    *   `ip dhcp`: This command instructs the VPCS device to send a DHCP request to obtain an IP address and other network configuration parameters from a DHCP server on the network.

## Verification Steps: How to Check if DHCP is Working

After configuring the router as a DHCP server and issuing the `ip dhcp` command on the VPCS devices, you need to verify that DHCP is working correctly. Here's how:

1.  **Verify IP Address Configuration on VPCS Devices:**
    *   **Open the console of each VPCS device (PC1, PC2, PC3, PC4) one by one.**
    *   **In each VPCS console, type `show ip` or simply `ip` and press Enter.**
    *   **Examine the output.** You should see the following information for each VPCS, indicating successful DHCP configuration:
        *   **IP Address:**  The VPCS should have an IP address assigned within the DHCP range you configured on the router (`192.168.1.1` - `192.168.1.100`). For example, you might see `IP: 192.168.1.2`, `IP: 192.168.1.3`, etc.
        *   **Mask:** The Subnet Mask should be `255.255.255.0`.
        *   **Gateway:** The Gateway IP address should be `192.168.1.1`, which is the IP address of the router's interface.
        *   **DNS:** If you configured the `dns-server` option in the DHCP pool, you should see the DNS server IP address (e.g., `DNS: 8.8.8.8`).

    *   **If you see IP addresses, subnet mask, and gateway information in the VPCS output, it means the VPCS devices have successfully obtained IP configurations from the DHCP server.**

2.  **Verify DHCP Bindings on Router R1 (DHCP Server Verification):**
    *   **Access the Router R1 console.**
    *   **Enter privileged EXEC mode:** `enable`
    *   **Type the command:** `show ip dhcp binding` and press Enter.
    *   **Examine the output.** This command displays a list of DHCP bindings. Each binding represents an IP address that the DHCP server has leased (assigned) to a client.
    *   **You should see entries for each VPCS device (PC1, PC2, PC3, PC4).** Each entry will show:
        *   The IP address assigned to the VPCS.
        *   The MAC address of the VPCS device.
        *   The lease expiration time (how long the IP address is leased for).
        *   The client-identifier (usually the MAC address again).

    *   **Seeing entries for your VPCS devices in the `show ip dhcp binding` output on the router confirms that the router's DHCP server has successfully assigned IP addresses to these clients.**

3.  **Test Network Connectivity with Ping:**
    *   **Open the console of any VPCS device (e.g., PC1).**
    *   **Ping the Router's Interface (Gateway):** Type `ping 192.168.1.1` and press Enter.
        ```
        VPCS> ping 192.168.1.1
        ```
        You should see successful ping replies (indicated by `!!!!!` or similar). This confirms that the VPCS can reach its default gateway (the router).

    *   **Ping Another VPCS Device:** Find the IP address of another VPCS device (e.g., PC2) using `show ip` in its console. Then, from PC1's console, ping PC2's IP address.
        ```
        VPCS> ping <PC2's IP Address>  (e.g., ping 192.168.1.3)
        ```
        Successful ping replies between VPCS devices confirm that they can communicate with each other within the LAN, indicating the switch is working and IP connectivity is established.

    *   **Unsuccessful pings at any of these steps indicate a problem with your DHCP configuration or network setup. Double-check your router and VPCS configurations and the physical connections in GNS3.**

## VPCS Persistence Note

**Important:** VPCS (Virtual PC Simulator) in GNS3 is designed to be lightweight and resource-efficient. By default, VPCS does **not** save its configuration when you close and reopen a GNS3 project. This means that when you restart your GNS3 project, you will need to issue the `ip dhcp` command again in each VPCS console to get them to request and obtain new IP addresses.

This behavior is intentional to keep VPCS simple and minimize resource usage. For basic DHCP learning and testing, this is usually acceptable. If you require persistent configurations for end devices in more advanced simulations, you might consider using full virtual machines running lightweight operating systems instead of VPCS, but VMs are more resource-intensive.

## GNS3 Project File

For easier setup and to replicate this project in GNS3, you can include the `.gns3project` file in this repository. This will allow others to simply open the project in their GNS3 environment and explore the configuration.

## Further Exploration

*   **Add more VPCS devices:** Connect more VPCS devices to the switch and observe them receiving DHCP addresses.
*   **Experiment with DHCP Lease Time:** Configure the `lease` option in the DHCP pool on the router to control how long IP addresses are leased to clients.
*   **DHCP Reservations:** Configure DHCP reservations to assign specific IP addresses to certain devices based on their MAC addresses.
*   **DHCP Options:** Explore other DHCP options you can configure, such as setting different DNS servers, NTP servers, or other client configuration parameters.

## Author

Vedant

Feel free to contribute to this project or use it as a starting point for your GNS3 learning!
