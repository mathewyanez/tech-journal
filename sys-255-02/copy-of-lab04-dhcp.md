# Copy of Lab04   DHCP

{% hint style="info" %}
Have you ever had to manually configure an IP address when connecting to a wifi or cellular network? Of course not! Behind the scenes, DHCP has been taking care of you. This lab will illustrate how DHCP works and why it is a core service in any network with clients.
{% endhint %}

## Objectives

Install and configure Linux [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) service on dhcp01.yourname.local.

## Prerequisites

Lab 3 is complete, and the environment is in a happy snappy state.

## SSH from AD01 to DHCP01

Disable IE Enhanced Security Configuration.

{% hint style="info" %}
Use either PuTTY (which you download + install) **or** Powershell SSH from AD01 to access your Rocky Linux server from now on! This allows you to copy/paste from your Windows system (you can login into Canvas from here). You can open up multiple windows should you wish. Note: Powershell SSH is somewhat newer, and you may experience occasional keyboard issues. If this is the case, then restart the SSH session or move to PuTTY. In order to pull down PuTTY from the internet, you will want to disable IE Enhanced Security Configuration on Server Manager as shown below:
{% endhint %}

{% hint style="info" %}
While it may seem otherwise, your Windows Server does indeed have a web browser. There are a number of ways to run the Program File iexplorer.exe.
{% endhint %}

Download, and then install PuTTY.

## Connect via PuTTY

## Install DHCP Services

We are going to use the dnf update manager to install DHCP on this server with your elevated user.

## Configuring DHCP Services

Become the system user for a brief period of time using the `sudo -i` command, copy the following `.conf` file to a `.bak` as a clean backup, and then open the dhcp configuration file using the vi or nano text editor (nano may be easier, but at some point you will need to learn vim so we will use it).

Typing very carefully, enter the following into your new file below the comments (`#` denotes a comment). Change the domain-name to yourname.local.

When you think you are done, write [changes](https://www.computerhope.com/unix/uvi.htm) to save and quit vi.

## Starting DHCP Services

{% hint style="danger" %}
Whenever you change or create a service configuration file, you generally need to start or restart the service involved. This fact trips up many Linux administrators! The [systemd](https://www.linux.com/training-tutorials/understanding-and-using-systemd/) control program systemctl is how you start, stop and status services.
{% endhint %}

To start dhcp, type the following as root:

```bash
systemctl start dhcpd
```

Start dhcpd and check its status (Fun fact: the dhcp**D** means it’s a [Daemon](https://en.wikipedia.org/wiki/Daemon_\(computing\)), which is Linux speak for Service). Note any errors and check the syntax of `dhcpd.conf` accordingly.

## Enabling the Service to Start at Boot

{% hint style="danger" %}
Forgetting to **enable** a service to start on boot will cause the service to fail when the system is rebooted. Remember this if a service worked fine one day and does not after a reboot.
{% endhint %}

## Configuring the Firewall to Allow Incoming DHCP Requests

The default configuration on Rocky is to enable the firewall and allow both ICMP and SSH requests in. The DHCP server will not work until we enable the firewall. We will be using firewalld and the firewall-cmd utility to make this happen. The `firewall-cmd --list-all` option shows the default firewall that allows dhcpv6-client (not to be confused with dhcp server) and ssh. We will add the dhcp service (as opposed to its ports).

The following syntax adds the ports associated with dhcp permanently. The `--permanent` flag is important. If you fail to add this, the next time you reboot, your changes will be lost. Make sure you reload the firewall to invoke the change. List all the rules and make sure your dhcp service has been added.

{% hint style="danger" %}
Don't forget the `--permanent` flag, nor forget to reload the firewall!
{% endhint %}

Type `exit` to leave your elevated state and `exit` again to exit PuTTY.

## Windows 10 DHCP Client

As a privileged user, you will now re-configure networking on WKS01 to use dynamic addressing rather than static addresses.

### Deliverable 1

Take a snapshot of the results of `ipconfig /all`. Note the DHCP server of `10.0.5.3` should be there, your IP address should be the first IP address in the scope you set earlier. Your domain name, netmask and gateway should also be set correctly.

### Deliverable 2

Log back into dhcp01 and find the DHCP log associated with wks01's request for DHCP information. Take a snapshot similar to the one below. The IP address, the workstation name, the layer 2 address should all match between deliverables 1 and 2.

{% hint style="info" %}
The following command below looks complex, but let's break it down:

* `sudo` = raises our privileges because `/var/log/messages` is owned by the root user.
* `cat` = writes the file `/var/log/messages` to the screen.
* `|` = Called “pipe”, and sends the output of the previous command to the next command.
* `grep wks01-yourname` = filters input for the string `wks01-yourname`.
{% endhint %}

## Wireshark

Run a capture session against WKS01's Ethernet0 adapter.

As an administrative power user (`-adm`), release the current DHCP release and then renew it on WKS01. Your objective: Capture the four DHCP messages between client and server.

You release your current DHCP release using the following command:

```powershell
ipconfig /release
```

Stop the capture and create a Wireshark display filter that shows UDP traffic sourced or destined to port `67`. Browse each of the four messages to get a handle on source, destination addresses (layer 2 & layer 3) and ports and the sequence of messages used to provide WKS01 another leased IP address.

### Deliverable 3

Provide a screenshot similar to the one below that shows the 4 Key DHCP Messages.

## Leveling Up

### Deliverable 4

Figure out how to change the default lease time given to dhcp clients to 1 hour with a max lease time of four hours. Provide a screenshot displaying the new configuration, along with the shot confirming the change.

### Deliverable 5

Tech Journal Entry - Explore 3 other items related to DHCP, and dig into their related Wireshark captured packets.
