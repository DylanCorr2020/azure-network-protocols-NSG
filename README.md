# Exploring Network Security Groups and Network Protocols in Azure

This document outlines the steps taken to explore Network Security Groups (NSGs) and network protocols within Microsoft Azure. This lab environment focuses on creating basic network infrastructure and observing the default and configurable network security behaviors.

## Lab Environment Setup

In this lab, we will be creating the following resources within Azure:

- **Resource Group:** A logical container for all our Azure resources.
- **Virtual Network:** A private network within Azure, providing isolation for our virtual machines.
- **Subnet:** A range of IP addresses within the Virtual Network.
- **Two Virtual Machines (VMs):**
  - One running Windows Server (Windows 10 Pro version 22H2 - 64-bit Gen2).
  - One running Ubuntu Server 22.04 LTS.
- **Network Security Group (NSG):** A virtual firewall containing a list of Access Control List (ACL) rules that allow or deny network traffic to resources connected to VNets (Virtual Networks).

## Prerequisites

Before proceeding, ensure you have the following:

- An active Azure Tenant Subscription.

## Part 1: Initial Setup and Connectivity

### Visualisation

The following image provides a visual representation of the initial steps in creating a resource group within the Azure portal.

![Create a resource group](NetworkImages/Create_a_resource_group.png)

### Step 1: Create a Resource Group

The first step involves creating a resource group. This acts as a logical container to manage all the resources we will create for this lab.

![Resource group creation details](NetworkImages/Resource_group_details.png)

### Step 2: Create the First Virtual Machine and the Initial Virtual Network

Next, we proceed to create our first virtual machine. During this process, Azure will also prompt us to configure the virtual network and subnet.

- We selected **Windows 10 Pro version 22H2 - 64-bit Gen2** as the image for our first virtual machine.

![Virtual machine image selection](NetworkImages/vm_image_selection.png)

- When sizing the virtual machine, we ensured it had at least **2 vCPUs**.

![Virtual machine size selection](NetworkImages/vm_size_selection.png)

- We then configured the network settings during the VM creation. Azure automatically creates a virtual network and subnet if one doesn't already exist. We accepted the default settings for this initial setup, noting the automatically generated **Virtual network name**, **Subnet**, and **Public IP**. We also observed the default inbound port rule for **RDP (3389)** being automatically opened for remote access.

![Network configuration during VM creation](NetworkImages/network_configuration.png)

- **Note:** (agree to license agreement when selecting Windows OS)

![Windows license agreement](NetworkImages/windows_license.png)

### Step 3: Create the Second Virtual Machine

For our second virtual machine, we will deploy an Ubuntu Server.

- We selected **Ubuntu Server 22.04 LTS** as the image. The size was kept as the default.

![Ubuntu VM image selection](NetworkImages/ubuntu_vm_selection.png)

- For the network settings of this second VM, we ensured it was placed within the **same virtual network** we created during the first VM deployment. We accepted the default subnet and did **not** assign a public IP address for this VM initially. We also observed that **no inbound port rules** are automatically created for Linux VMs by default.

- For the network security group, we didn't create a new one and associated it with the existing one from the first VM.

- Ensure it is in the same virtual network.

### Step 4: Inspect the Network Security Group

With both virtual machines deployed, our next step is to inspect the Network Security Group that was automatically created during the first VM's deployment. We will examine the default inbound and outbound security rules.

### Step 5: Explore Network Protocols

Finally, we will begin to explore different network protocols and how the Network Security Group affects communication between our two virtual machines.

## Part 2: Analyzing Network Traffic and Security Group Behavior

Continuing from Part 1, we now delve into analyzing network traffic between our virtual machines and observing how the Network Security Group (NSG) affects this traffic.

### Step 1: Connecting to the Windows VM

- We use Remote Desktop Connection to access the Windows VM.

  ![Remote Desktop Connection](NetworkImages/RDP.png)

### Step 2: Installing Wireshark

- We will install Wireshark on the Windows VM to capture and analyze network traffic.

  - Download and install the Windows 64-bit installer from the official Wireshark website.

- With Wireshark installed, we can view the traffic going from and to the Windows and Linux VMs.
- Start capturing network traffic using Wireshark.

  ![Wireshark Installation](NetworkImages/wiresharktrafficwo.png)

### Step 3: Filtering ICMP Traffic

- To test connectivity, we will use the `ping` command. Specifically, we will ping the Ubuntu VM (Linux VM1) from the Windows VM using its private IP address (10.0.0.5).

  ![Ping Request](NetworkImages/ICMP.png)

- In Wireshark, we filter for ICMP traffic to focus on the `ping` requests and replies.

  ![Wireshark ICMP Filter](NetworkImages/ICMPTwo.png)

- We observe the ping requests and replies in Wireshark, confirming basic network connectivity.

### Step 4: Firewall Configuration

- Ensure the firewall is configured correctly on both VMs to allow ICMP traffic, if necessary.

  ![Firewall Configuration](NetworkImages/SecurityGroupLinuxVM.png)

- Continue the non-stop ping from the Windows machine to the Linux machine.

  ![Continuous Ping](NetworkImages/NonStopPing.png)

- Wireshark captures the pings.

### Step 5: Network Security Group Blocking ICMP

- Next, we will modify the Network Security Group associated with the Linux VM to block ICMP traffic. This will simulate a firewall rule.

  ![NSG Block ICMP](NetworkImages/ICMPtrafficblocked.png)

- After applying the rule, we observe that the `ping` requests from the Windows VM no longer receive replies, demonstrating the NSG's effect.

  ![Blocked Pings](NetworkImages/RequestTimedOut.png)

### Step 6: Removing the Security Group Rule

- Finally, we remove the rule blocking ICMP traffic in the Network Security Group.

- Connectivity should be restored, and `ping` requests should again receive replies.

## Part 3: Exploring SSH Traffic and DHCP

In this final part of our lab, we will investigate SSH traffic, observe the DHCP process, and analyze DNS queries using Wireshark.

### Step 1: Analyzing SSH Traffic

- Start a packet capture in Wireshark and filter for SSH traffic.

- Use SSH to connect from the Windows VM to the Linux VM. You'll need an SSH client (like PuTTY on Windows). Ensure the SSH server is running on the Linux VM (it usually is by default).

- You might need to open port 22 on the Linux VM's Network Security Group if you haven't already.

- Use the Linux VM's private IP address (10.0.0.5) to connect.

  ![SSH Connection Attempt](NetworkImages/SHHLinuxVM.png)

- When prompted, enter the password for your user on the Linux VM.

  ![Wireshark SSH Traffic](NetworkImages/wiresharktrafficpng.png)

- All traffic is encrypted when using SSH, making the content unreadable in Wireshark without decryption keys.

### Step 2: Observing DHCP

- From the Windows VM, we will release its current IP address and request a new one from the DHCP server. We will then observe this DHCP traffic in Wireshark.

- Open Command Prompt on the Windows VM and run the following commands:

  ```bash
  ipconfig /release
  ipconfig /renew
  ```

  ![IP Release and Renew](NetworkImages/DHCP.png)

- Observe the DHCP traffic in Wireshark. Filter for `dhcp` to see the DHCP Discover, Offer, Request, and Acknowledge (DORA) process.

  ![Wireshark DHCP Traffic](NetworkImages/DHCP.png)

### Step 3: Analyzing DNS Traffic

- Back in Wireshark, filter for DNS traffic.

- On the Windows VM, use the `nslookup` command to query the IP address of a website (e.g., `nslookup google.com`).

  ```bash
  nslookup google.com
  ```

  ![DNS Lookup](NetworkImages/DNSLookup.png)

- Observe the DNS query and response in Wireshark. You should see the Windows VM sending a DNS query to the configured DNS server, and the server responding with the IP address of `google.com`.

  ![Wireshark DNS Traffic](NetworkImages/DNSLookup.png)

### Step 4: Analyzing HTTP Traffic (Non-Stop Spam?)

- Now, let's filter Wireshark for HTTP traffic.

- The observation here suggests a large amount of HTTP traffic that might appear as "non-stop spam." This could be due to various background processes on the VMs or network activity.

- Observing the immediate non-stop spam of HTTP traffic after applying the filter might indicate persistent communication or telemetry being sent. Analyzing the details of these packets in Wireshark would be necessary to understand the source and destination of this traffic.

  ![Wireshark HTTP Spam](NetworkImages/wireSharkHTTP.png)

This concludes our exploration of network protocols and security using Azure and Wireshark. We've examined ICMP blocking with NSGs, analyzed encrypted SSH communication, observed the DHCP process, and looked at DNS queries. The "HTTP spam" observation highlights the importance of understanding background network activity within virtual machines.
