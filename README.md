برای آموزش فارسی کلیک کنید [اینجا](farsi.md).



# MikroTik 7.5+ Container Setup and Configuration Guide

This guide provides detailed steps for setting up and configuring containers on a MikroTik router with version 7.5+ and installing the `x-ui` container. Follow the steps to ensure your containers are properly installed, configured, and accessible.

## Prerequisites

- MikroTik Router with version 7.5+.
- Access to MikroTik's web interface or terminal.
- A stable internet connection to download required packages.
  
---

## Steps to Install and Configure the Container Package

### 1. **Update MikroTik RouterOS to Version 7.5+**
   - Make sure your MikroTik router is updated to version 7.5 or higher.
   - Visit MikroTik’s official website to download the latest firmware if needed.

### 2. **Download the Container Package**

   - Visit the [MikroTik Archive](https://mikrotik.com/download/archive) and download the required container packages for your MikroTik model.
   - Download the correct archive package for your router and extract it.

### 3. **Upload the Container Files to MikroTik**

   - Upload the container files from the downloaded archive into the **File** section of the MikroTik router.
   - Restart the router to install the new container package.

### 4. **Activate Containers**

   After restarting the router, run the following command to activate the container functionality:

   ```bash
   /system/device-mode/update container=yes
   ```

   After activation, perform a power cycle:

   - **Power Off** the MikroTik router.
   - **Power On** the MikroTik router to complete the activation process.

### 5. **Configure Docker for Containers**

   Run the following configuration to enable container functionality:

   ```bash
   /container/config/set ram-high=0 registry-url=https://registry-1.docker.io tmpdir=pull
   ```

   **Note:**  
   If your MikroTik router is located in Iran, use the following registry URL:

   ```bash
   /container/config/set registry-url=https://docker.arvancloud.ir
   ```

---

## Setting Up Networking for Containers

To configure networking for containers, you need to add a Virtual Ethernet Interface (veth) and a bridge:

### 6. **Create Virtual Ethernet Interface (veth1)**

   ```bash
   /interface/veth add name=veth1 address=172.17.0.2/24 gateway=172.17.0.1
   ```

### 7. **Create Bridge for Containers**

   ```bash
   /interface bridge add name=dockers
   /interface bridge port add bridge=dockers interface=veth1
   ```

### 8. **Assign IP Address to the Interface**

   ```bash
   /ip/address/add address=172.17.0.1/24 interface=veth1
   ```

### 9. **For Additional Containers (Optional)**

   To add more containers, create additional virtual Ethernet interfaces (`veth2`, `veth3`, etc.):

   ```bash
   /interface/veth add name=veth2 address=172.17.0.3/24 gateway=172.17.0.1
   /interface bridge port add bridge=dockers interface=veth2
   ```

---

## Mounting and Configuring the `x-ui` Container

### 10. **Mount the Required Volumes**

   To persist the data, add the necessary mount points for the `x-ui` container:

   ```bash
   /container/mounts add dst=/etc/x-ui/ name=x-ui src=/x-ui/db
   /container/mounts add dst=/root/cert/ name=cert src=/x-ui/cert
   ```

### 11. **Add and Start the Container**

   Use the following command to add the container and configure it to start on boot:

   ```bash
   /container/add interface=veth1 mounts=cert,x-ui workdir=/app start-on-boot=yes remote-image=metaligh/3x-ui
   ```

---

## Configure Firewall and Port Forwarding

### 12. **NAT Firewall Rule for Container**

   Add a NAT rule to forward traffic to your container. This is crucial for accessing services hosted inside the container from external networks.

   ```bash
   /ip/firewall/nat add chain=srcnat action=masquerade
   ```

### 13. **Destination NAT Rule for Port Forwarding**

   Get the IP address of your router's interface and assign it to a variable. Use this IP in the following command to forward traffic to the container:

   ```bash
   :global myIP [/ip address get [find interface="ether1"] address]
   /ip/firewall/nat add chain=dstnat dst-address=$myIP protocol=tcp dst-port=2053 action=dst-nat to-addresses=172.17.0.2 to-ports=2053
   ```

   **Note:**  
   **Ensure that the IP address you use for the forwarding rule is correct for your network setup.** You can check and adjust the IP address in `/ip/firewall/nat` to ensure that it is properly forwarded to the right container.

---

## Conclusion

You have successfully set up containers on your MikroTik router and configured the `x-ui` container. Your network should now be able to access the services running inside the container through the specified ports.

---

## Important Note

- **Always double-check the port forwarding rules in `/ip/firewall/nat`** to ensure the correct IP and port are forwarded. Adjust the `dst-address` and `to-addresses` if needed based on your router and network configuration.
