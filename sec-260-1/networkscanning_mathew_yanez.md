# NetworkScanning\_Mathew\_Yanez

All of my Kali machines shut down on proxmox so I had to make due with what I had.

* `-sV` Version detection probes open ports to determine service/version info
* `--version-intensity 5` Sets how aggressively to probe for version info (0=light, 9=max)
* `-T4` Timing template aggressive timing for faster scans on reliable networks
* `-O` OS detection attempts to identify the operating system of the target
* `-F` Fast mode scans only the 100 most common ports instead of all 1000

### Deliverable 2: Host Scan Results

Due to not having access to the promox network, finding results held smaller amounts of information.

#### Host 10.0.2.2

The IP address of this host is 10.0.2.2. The Quick Scan Plus scan revealed that this host had 0 open ports, 100 filtered ports, and 0 closed ports. Because there were no open ports, no services were detected running on this host. The OS was not able to be detected by Nmap. The MAC address associated with this host is 52:55:0A:00:02:02.

#### Host 10.0.2.3

The IP address of this host is 10.0.2.3. The scan revealed that 1 port was open on this host. Port 53 was open using the TCP protocol, and the service running on it was identified as a domain service with the version detected as Simple DNS Plus. This host had 99 filtered ports and 0 closed ports out of 100 total scanned ports. The OS was not detected by Nmap. The MAC address associated with this host is 52:55:0A:00:02:03.

#### Host 10.0.2.15

The IP address of this host is 10.0.2.15. The scan revealed that this host had 0 open ports, 100 filtered ports, and 0 closed ports out of 100 total scanned ports. No services were detected and no OS information was available. No MAC address was listed for this host, as it was the local machine.

### Deliverable 3

Running netcat in listening mode on port 5555 allowed the Intense Scan with UDP to detect that port as active on the host. Compared to the Quick Scan Plus, the Intense Scan with UDP provides much deeper detail including service version numbers, protocols, and UDP based services like DNS and DHCP that a TCP only scan would miss. Port 5555 appeared as an open TCP port on the local machine, and other machines on the same subnet would also be able to detect it when scanning that host.

### Deliverable 4

During the Quick Scan Plus, a full 3-way handshake was visible on port 53 of host 10.0.2.3, confirming it was open. Filtered ports showed only an outgoing SYN with no response, meaning a firewall was silently dropping the packets. Closed ports would return a RST-ACK instead. The netcat traffic appeared on port 5555 using TCP with no recognizable application layer protocol since netcat passes raw data.

Two indicators that would alert a defender to scanning activity are a single IP sending SYN packets to many ports in rapid succession, and a high volume of RST-ACK responses returning in a short time. The Wireshark timeline shows a dramatic traffic spike during the scan that drops off immediately after, showing that Nmap scans are very loud and easily distinguishable from normal traffic.

<details>

<summary>Question 1</summary>

Based on the scan, the top three security risks are the exposed DNS service on 10.0.2.3, the large number of filtered ports that may not be actively monitored, and the unidentified host 10.0.2.15 with no detectable OS or MAC address. An unpatched DNS service can be exploited for spoofing or amplification attacks, unknown hosts on a network are a red flag for potential rogue devices, and unmonitored filtered ports can give attackers useful information about network layout even without being open. Keeping services patched, auditing firewall rules regularly, and maintaining a full device inventory would be the first steps in addressing these risks.

</details>

<details>

<summary>Question 2</summary>

Attackers use scanning to find weak points and plan their approach, while defenders use the exact same tools to find those weak points first and close them. The fact that both sides are using identical tools makes regular security audits not just useful but necessary. If you are not scanning your own network, someone else probably is, and unlike Squidward, they are not going to announce themselves before showing up.

</details>

<details>

<summary>Question 3</summary>

Quick Scan Plus is fast and light, checking only the top 100 TCP ports, making it practical for routine sweeps across a large number of hosts. Intense Scan Plus UDP goes much deeper, adding a full UDP scan which can take over an hour but reveals services that TCP scans miss entirely. Use Quick Scan Plus when you need speed and coverage, and Intense Scan Plus UDP when you need the full picture on a specific target.

</details>

<details>

<summary>Question 4</summary>

Filtered ports are what happens when a firewall decides it simply is not going to answer. No SYN-ACK, no RST, just silence. This is actually intentional on the defender side because a non-response gives attackers less information to work with than a flat rejection would. It tells us the network has active filtering in place, which is a good sign, though it does not mean those ports are completely invisible to a determined scanner.

</details>

<details>

<summary>Question 5</summary>

Scanning a network without permission is illegal, plain and simple. In the US the Computer Fraud and Abuse Act covers unauthorized scanning even if nothing is exploited, and consequences can include criminal charges and civil liability. Beyond the legal risk, unauthorized scanning can disrupt services and immediately burn any trust with the system owners. The rule is simple: are you ready kids? If you do not own it or have written permission, do not scan it.

</details>
