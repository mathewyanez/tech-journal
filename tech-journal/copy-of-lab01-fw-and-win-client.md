# Copy of Lab01   FW and Win Client

Lab 01 - Virtual Firewall and Windows Client Configuration

| 💡This lab is the very first step in building a small enterprise network and will serve as the foundation of future labs. Some of you may be familiar with Virtualbox or VMWare Workstation. This environment is very similar but allows remote access and leverages the Proxmox platform. In this lab you will:Become familiar with your lab environmentConfigure your own firewall that separates your student local area network (LAN) from the other students in the class (WAN)Configure a single Windows Workstation to communicate with the Internet |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### Resources

* URL for WAN IP, please refer to Network Assignments via Canvas Homepage
* URL for Remote Access (\*if you are not on Champlain’s environment) [https://viewportal.champlain.edu](https://viewportal.champlain.edu/)
* URL for Proxmox once on Champlain Network: [https://prox01.cyber.local](https://prox01.cyber.local/)

### PfSense - fw01

The [PfSense](https://www.pfsense.org/) firewall will provide routing services between a Local Area Network and Wide Area Network in your environment.

Figure out how to modify the settings of your fw01 VM, and make sure that the first network adapter is assigned to WAN, and the second assigned to the LAN-yourname network as shown in the following figures.

Select the First Network Adapter (net0), and then select your class section’s WAN to match its corresponding “cable” connection.

* Ex: if you’re in section SYS-255-01, then SYS-255-01-WAN is correct

While in the First Network Adapter’s config window, notice its Model Type and MAC Address. This will come in handy in a few moments.

* This is an example for net0’s MAC; yours will likely be different

Repeat the similar process for the Second Network Adapter (net1), making sure to select the your section’s corresponding LAN “cable” connection -

* This is an example for net1’s MAC; yours will likely be different

Once you OK’d the Network Device config windows, congrats … you have “cabled” your firewall VM’s 2x network adapters.

#### Power on your fw01 VM and Open a VM Console

Find the menu items or icons that allow you to first power on, and then open a web console to your firewall virtual machine.

Your console should now look similar to the following after power on and login:

| 💡The default username and password for pfsense is listed in Default Passwords on our Home Page |
| ----------------------------------------------------------------------------------------------- |

Your WAN and LAN are either missing network configuration information or contain default IP addresses. In the next few steps, we will assign our interfaces to the appropriate network and configure IP addresses on each interface.

#### Step 1: Assign Interfaces

Our interfaces should be assigned in the same order as they appeared in our Proxmox configuration, namely the WAN should be associated with the one interface and the LAN should be associated with the other interface.

To double check & match network interface addresses, first recall the earlier 2x MAC addresses of the FW01’s network interfaces.

* Select 1 to reassign Network Interfaces and follow the following steps:
  * For due diligence, double check the MAC addresses to match the earlier displays. In this example, we cabled the WAN network adapter on bc:24:11:flag\_bf:95:b8 in Proxmox net0, which matches vtnet0 in PfSense. This is good as it means we cabled it correctly and PfSense sees the Network Adapter we want to use for WAN connectivity. The similar principle of matching MAC addresses applies to the LAN network adapter and its MAC address displaying in PfSense for vtnet1. If these MAC addresses do not match, then effectively you have miscabled the VM, and thus no network connectivity until that is resolved.
*
  * Do not configure VLANs now
  * The WAN interface name should be changed
  * The LAN interface name should be changed
  * If prompted for an optional interface, just select
  * If successful, your interfaces should look like this:<br>
  * When prompted to proceed, do so.

#### Step 2: Set interface IP address

The first interface vtnet0 will be assigned to an WAN address that is documented in the course Network Assignments in Canvas. This interface represents the outside of your network. Make sure to use your assigned IP address

* Select 2 to Set interface IP Address
  * Select 1 to pick the WAN interface
  * Do not use DHCP for the WAN IPv4 address
  * Enter your Assigned WAN IP from the Network Assignments sheet in Canvas
  * You are using a 24 bit subnet mask
  * For the WAN, your upstream gateway is 10.0.17.2
  * Use the gateway as your IPv4 name server as well
  * We will not be using IPv6, respond no when asked about DHCP.
  * Press to bypass IPv6 configuration
  * Do not enable DHCP server on WAN
  * When asked about HTTP for the GUI, respond no (we want to use secure https)
* Select 2 again to configure the other Interface's IP Address
  * Select 2 to pick the LAN interface
  * We are not using DHCP
  * Your LAN IP Address is 10.0.5.2. This is the same for every student.
  * You are using a 24 bit subnet mask
  * You do not have an upstream LAN gateway (you are the gateway for the LAN). Press
  * No DHCPv6
  * Press to bypass IPv6 configuration
  * Do not enable a LAN DHCP Server
  * Do not revert to HTTP

| 💣Use your assigned IP address 10.0.17.1XX here for the WAN address, not the instructor's as shown below. |
| --------------------------------------------------------------------------------------------------------- |

We will come back to fw01 to complete the configuration through the web interface once we have a Windows client to use.

### Windows Client - wks01

Figure out how to adjust wks01's Proxmox Network Configuration so it is on your LAN segment (see below):

Your hostname/computer name should be set to wks01-yourfirstname.

Open File Explorer

Right-click on “This PC”

Click “Properties”

Click on “Change Settings”

Click “Change” next to “To rename this computer…”

Then type: wks01-yourfirstname

Check “firstname” to your real first name.

| 💡The Windows desktop system (wks01) will display the _champuser_ username which uses its default password (in Canvas). You will need to set up a new local administrator account, which you will use for the rest of the term.Here are [specific instructions](https://docs.google.com/document/d/1mnjUIZ1UqK6Klw2ZKlOs8Gf7nMETFqud6lb-eIyD2HE/edit?usp=sharing) on how to add a new local administrative user. |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

You should be now in your new local admin account, on a host with your new hostname.

Make sure you have the following network configuration items taken care of following installation:

| 💣Remember any passwords you used so far! |
| ----------------------------------------- |

### fw01 gui configuration

You may have noticed that your Windows 10 system is not connected to the internet, so we will need to adjust our firewall (fw01) to make this happen. Navigate to fw01's LAN IP address (bypass any certificate warning).

Use the same password you used when logging into the PfSense console.

The following are screens where you need to change the default.

**Skip over the wizard and leave the setting checked to override the DNS server on PPP/WAN**

* System Wizard: General Information
  * Hostname: fw1-yourfirstname
  * Domain: yourfirstname.local
  * Primary DNS: 8.8.8.8
* System Wizard: Configure WAN Interface
  * RFC1918 Networks: Uncheck "Block private networks from entering via WAN"
* System / User Manager: Set Root Password
  * Up to you. If you set it, then you need to remember it!

| 💣Remember any passwords you use. If you forget, you will need to repeat the installation. |
| ------------------------------------------------------------------------------------------ |



**Deliverable 1: Screenshot showing a successful ping from fw01 to champlain.edu: Select 8 to get a shell, and then execute the ping shown below. Type exit to leave the command shell.(3 points)**

**Deliverable 2: On wks01, figure out how to invoke powershell and provide a screenshot similar to the one below showing the output of the following commands: whoami, hostname, ping -n 1 google.com and ipconfig (2 points)**

**Deliverable 3: Take a screenshot showing successful navigation from wks01 to champlain.edu using chrome. Make sure to get the Virtual Machine Name VM window banner (2 points)**

**Deliverable 4: On wks01, use the tracert command against champlain.edu with a maximum of three hops. This command should illustrate how packets are being routed from your private LAN to your WAN. Provide a screenshot showing your tracert command and hops 1-3. (1 points)**

**Deliverable 5: Consider this lab. What technical terms or steps were you unfamiliar with? Provide at least 3 examples (1 point). Example:**

| The lab mentioned a default gateway a couple times. What is that and why is it important? |
| ----------------------------------------------------------------------------------------- |

**Deliverable 6. Your deliverable meets the submission** [**guidelines**](https://docs.google.com/document/d/1Mbjso3I5UD5bAm2SZisjxO6aqM3ao9mRbqnbAAfjmto/edit)**.**

**Deliverable 7. Tech Journal entry. Make your github public & include URL.**
