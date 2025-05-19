# Exploring Network Security Groups and Network Protocols in Azure

In this turtorial we will create two virtual machines and observe diffrent network protocals using Wireshark.


# Environment and Technologies Used
Microsoft Azure (Virtual Machines/Compute)
Remote Desktop
Various Command-Line Tools
Various Network Protocols (ICMP, SSH, DHCP, DNS, RDP)
Wireshark (Protocol Analyzer)

# Part 1 Set Azure Environment


### Step 1: Create a Resource Group

The first step involves creating a resource group. This acts as a logical container to manage all the resources we will create for this lab.

<img width="505" alt="Image" src="https://github.com/user-attachments/assets/96c28be2-c1e4-4912-95e3-de8ef9e2a7d4" />

### Step 2: Create the two Virtual Machines

Create a Windows 10 Virtual Machine (VM). While creating the VM, select the previously created Resource Group . While creating the VM, allow it to create a new Virtual Network (Vnet) and Subnet. When selecting the OS select Windows 10 Pro Version 22H2 - 64X Gen2.
When selecting size choose the following ensure it has 2 vcpus. Authentication type: Username/Password. 
- **Note:** (agree to license agreement when selecting Windows OS)

<img width="444" alt="Image" src="https://github.com/user-attachments/assets/67f09032-c14b-4a40-b00b-1ae0f00e4c97" />



Create a Linux (Ubuntu) VM. While creating the VM, select the previously created Resource Group and Virtual Network—the Virtual Network MUST BE THE SAME. Authentication type: Username/Password. 
Ensure both VMs are in the same Virtual Network / Subnet. Image can be Ubuntu server 22.04 and size can be 2vpcus and 8GB of memory.

<img width="313" alt="Image" src="https://github.com/user-attachments/assets/385a42b5-7c2f-4516-97ed-c1084c5ae8da" />


# Part Two Analyzing Network Traffic between our Virtual Machines

Continuing from Part 1, we now delve into analyzing network traffic between our virtual machines and observing how the Network Security Group (NSG) affects this traffic.

### Step 1: Connecting to the Windows VM

- We use Remote Desktop Connection to access the Windows VM.

  ![Remote Desktop Connection](NetworkImages/RDP.png)

### Step 2: Installing Wireshark

- We will install Wireshark on the Windows VM to capture and analyze network traffic.

  - Download and install the Windows 64-bit installer from the official Wireshark website.

- With Wireshark installed, we can view the traffic going from and to the Windows and Linux VMs.


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
