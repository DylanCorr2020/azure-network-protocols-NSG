# Exploring Network Protocols in Azure

In this turtorial we will create two virtual machines and observe diffrent network protocals using Wireshark.


# Environment and Technologies Used
* Microsoft Azure (Virtual Machines/Compute)
* Remote Desktop
* Various Command-Line Tools
* Various Network Protocols (ICMP, SSH, DHCP, DNS, RDP)
* Wireshark (Protocol Analyzer)

# Part One Set Azure Environment


### Step 1: Create a Resource Group

The first step involves creating a resource group. This acts as a logical container to manage all the resources we will create for this lab.

<img width="505" alt="Image" src="https://github.com/user-attachments/assets/96c28be2-c1e4-4912-95e3-de8ef9e2a7d4" />

### Step 2: Create the two Virtual Machines

Create a Windows 10 Virtual Machine (VM). While creating the VM, select the previously created Resource Group . While creating the VM, allow it to create a new Virtual Network (Vnet) and Subnet. When selecting the OS select Windows 10 Pro Version 22H2 - 64X Gen2.
When selecting size choose the following ensure it has 2 vcpus. Authentication type: Username/Password. 
- **Note:** (agree to license agreement when selecting Windows OS)

<img width="700" alt="Image" src="https://github.com/user-attachments/assets/67f09032-c14b-4a40-b00b-1ae0f00e4c97" />



Create a Linux (Ubuntu) VM. While creating the VM, select the previously created Resource Group and Virtual Network—the Virtual Network MUST BE THE SAME. Authentication type: Username/Password. 
Ensure both VMs are in the same Virtual Network / Subnet. Image can be Ubuntu server 22.04 and size can be 2vpcus and 8GB of memory.

<img width="505" alt="Image" src="https://github.com/user-attachments/assets/385a42b5-7c2f-4516-97ed-c1084c5ae8da" />


# Part Two Analyzing Network Traffic between our Virtual Machines

Continuing from Part 1, we now delve into analyzing network traffic between our virtual machines and observing how the Network Security Group (NSG) affects this traffic.

### Step 1: Connecting to the Windows VM

- We use Remote Desktop Connection to access the Windows VM.

  <img width="505" alt="Image" src="https://github.com/user-attachments/assets/beb2983c-9b67-4f73-a072-6d2529e5321f" />

### Step 2: Installing Wireshark

-   We will install Wireshark on the Windows VM to capture and analyze network traffic.
-   Download and install the Windows 64-bit installer from the official Wireshark website.
-   Download here: [Wireshark Official Website](https://www.wireshark.org/download.html)

### Step 3: Filtering ICMP Traffic

- ICMP (Internet Control Message Protocol): Used for error reporting and network diagnostics. It doesn't use a specific port in the traditional sense, but rather relies on IP directly.

- To test connectivity, we will use the `ping` command. Specifically, we will ping the Ubuntu VM (Linux VM) from the Windows VM using its private IP address (10.0.0.5).

- In Wireshark, we filter for ICMP traffic to focus on the `ping` requests and replies.

  <img width="505" alt="Image" src="https://github.com/user-attachments/assets/ab2a8bf9-f604-4063-b4d1-3be8fcb21a35" />

- We observe the ping requests and replies in Wireshark, confirming basic network connectivity.

### Step 4: NSG Configuration to disable incoming (inbound) ICMP traffic

- Initiate a perpetual/non-stop ping from your Windows 10 VM to your Linux VM.
- Open the Network Security Group your Linux VM is using and disable incoming (inbound) ICMP traffic


  <img width="505" alt="Image" src="https://github.com/user-attachments/assets/70caca35-2867-4b26-a9ba-6aba3266b48f" />

  
- Back in the Windows 10 VM, observe the ICMP traffic in WireShark and the command line Ping activity. The windows VM should not be able to ping the Linux VM request times out.

  
<img width="505" alt="Image" src="https://github.com/user-attachments/assets/12523efd-509f-4d3c-b584-801539bef4d7" />


### Step 6: Analyzing SSH Traffic

- SSH (Secure Shell): Provides a secure encrypted connection for remote login and command execution. Port 22.

- Start a packet capture in Wireshark and filter for SSH traffic on the Windows 10 VM. From Windows 10 VM, we will SHH into the linux VM using it's private ip address.

- Open PowerShell, and type: ssh labuser@(linux private IP address)

- Type commands (username, pwd, etc) into the linux SSH connection and observe SSH traffic spam in WireShark

- Exit the SSH connection by typing ‘exit’ and pressing [Enter]

<img width="505" alt="Image" src="https://github.com/user-attachments/assets/3c9bafdc-d424-46b9-8f62-4e8c415c632f" />

- All traffic is encrypted when using SSH, making the content unreadable in Wireshark without decryption keys.

### Step 7: Observing DHCP

- DHCP (Dynamic Host Configuration Protocol): Automatically assigns IP addresses and other network configuration to devices on a network. Ports 67 (server) and 68 (client).

- From the Windows VM, we will release its current IP address and request a new one from the DHCP server. We will then observe this DHCP traffic in Wireshark.

- Open Command Prompt on the Windows VM and run the following commands:

  ```bash
  ipconfig /release
  ipconfig /renew
  ```

- Observe the DHCP traffic in Wireshark. Filter for `dhcp` to see the DHCP Discover, Offer, Request, and Acknowledge (DORA) process.

<img width="505" alt="Image" src="https://github.com/user-attachments/assets/ba1dd7ee-38a2-478b-b632-3a5f6312e20e" />


### Step 8: Analyzing DNS Traffic

- DNS (Domain Name System): Translates human-readable domain names into IP addresses. Port 53.

- Back in Wireshark, filter for DNS traffic.

- On the Windows VM, use the `nslookup` command to query the IP address of a website (e.g., `nslookup disney.com`).

  ```bash
  nslookup google.com
  ```

  - Observe the DNS query and response in Wireshark. You should see the Windows VM sending a DNS query to the configured DNS server, and the server responding with the IP address of `disney.com`.
 
  <img width="505" alt="Image" src="https://github.com/user-attachments/assets/21b29306-7ccf-4724-89ba-ae7012fe48a2" />

### Step 9: Analyzing RDP Traffic

-  RDP (Remote Desktop Protocol): Allows a user to graphically control a remote computer. Port 3389.
-  Back in Wireshark, filter for RDP traffic only (tcp.port == 3389)
-  Observe the immediate non-stop spam of traffic
-  RDP (protocol) is constantly showing you a live stream from one computer to another, therefor traffic is always being transmitted.

  <img width="505" alt="Image" src="https://github.com/user-attachments/assets/144d152e-aa85-4ff9-9765-1db14b5d3327" />

# Conclusion 

- 	Close your Remote Desktop connection. Delete the Resource Group(s) created at the beginning of this lab. Verify Resource Group Deletion

-   This concludes our exploration of network protocols and security using Azure and Wireshark. We've examined ICMP blocking with NSGs, analyzed encrypted SSH communication, observed the DHCP process, DNS queries and RDP Traffic . 


