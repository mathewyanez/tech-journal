# dhcp deep dive notes

> **Goal:** Go past "DHCP hands out IPs" and look at three related pieces: the **options** DHCP delivers, the **lease lifecycle** (renew, release, NAK), and **rogue DHCP / starvation attacks**. For each one, what it looks like on the wire in Wireshark.

## 📖 Key Terms (Plain English)

| Term                  | What It Actually Means                                                                                                                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **DHCP**              | Dynamic Host Configuration Protocol. The service that automatically gives a machine its IP, subnet mask, gateway, and DNS so nobody has to type them in by hand. |
| **DORA**              | The 4 step handshake every DHCP client does: **D**iscover, **O**ffer, **R**equest, **A**cknowledge.                                                              |
| **Lease**             | The IP isn't yours forever. You're _renting_ it for a set amount of time and have to check back in to keep it.                                                   |
| **Scope**             | The pool of addresses a DHCP server is allowed to hand out (for example `10.0.5.100` to `10.0.5.150`).                                                           |
| **DHCP Option**       | Extra settings tucked inside a DHCP packet, each with a number. Option 3 = gateway, Option 6 = DNS server, etc.                                                  |
| **UDP 67 / 68**       | DHCP's ports. The **server** listens on 67, the **client** listens on 68.                                                                                        |
| **Broadcast**         | A message sent to _everyone_ on the local network (`255.255.255.255`). A brand new client has no IP yet, so it has to shout.                                     |
| **Rogue DHCP server** | An unauthorized DHCP server on the network handing out its own (usually malicious) settings.                                                                     |
| **DHCP starvation**   | An attack that floods the server with fake requests until every address in the scope is used up.                                                                 |
| **DHCP snooping**     | A switch feature that only lets DHCP _server_ replies come from trusted ports. The main defense against rogue servers.                                           |

## 🔁 Baseline: What Normal DORA Looks Like

Before digging into the three topics, this is the "normal" you compare everything against.

| # | Packet       | From → To                           | Why                                                                                          |
| - | ------------ | ----------------------------------- | -------------------------------------------------------------------------------------------- |
| 1 | **Discover** | `0.0.0.0:68` → `255.255.255.255:67` | "Is there a DHCP server out there?" Client has no IP, so it broadcasts.                      |
| 2 | **Offer**    | Server → client                     | "Here's an IP you can have, plus settings."                                                  |
| 3 | **Request**  | `0.0.0.0` → `255.255.255.255`       | "I'll take that one." Still a broadcast so _other_ servers know their offer was turned down. |
| 4 | **ACK**      | Server → client                     | "It's yours. Here's your lease time."                                                        |

### 🦈 Wireshark Filters to Know

| Filter                  | What It Does                                                                        |
| ----------------------- | ----------------------------------------------------------------------------------- |
| `dhcp`                  | Shows all DHCP traffic. (Older Wireshark versions called it `bootp`. Same thing.)   |
| `port 67 or port 68`    | **Capture** filter version, set _before_ you start capturing so you only grab DHCP. |
| `dhcp.option.dhcp == 1` | Only Discovers. Swap the number for the message type below.                         |

**Message type numbers (Option 53):** 1 Discover, 2 Offer, 3 Request, 4 Decline, 5 ACK, 6 NAK, 7 Release, 8 Inform.

{% hint style="info" %}
**Easy way to generate a clean capture:** Start Wireshark on the client, then force a fresh lease. On Windows: `ipconfig /release` then `ipconfig /renew`. On Rocky: `sudo nmcli connection down <name> && sudo nmcli connection up <name>`.
{% endhint %}

## 1️⃣ DHCP Options: The Settings Hiding Inside the Packet

DHCP doesn't just hand out an IP. The **Offer** and **ACK** carry a bundle of options that configure the whole network stack. This is where most real world misconfigurations live.

| Option # | Name                   | What It Sets                                                  |
| -------- | ---------------------- | ------------------------------------------------------------- |
| **1**    | Subnet Mask            | e.g. `255.255.255.0`                                          |
| **3**    | Router                 | Default gateway (in our lab, `fw01` at `10.0.5.2`)            |
| **6**    | DNS Server             | Where the client sends name lookups                           |
| **15**   | Domain Name            | The DNS suffix, e.g. `yourname.local`                         |
| **51**   | Lease Time             | How long the rental lasts                                     |
| **54**   | Server Identifier      | IP of the DHCP server that answered (important for section 3) |
| **55**   | Parameter Request List | Sent **by the client**: a wish list of which options it wants |
| **12**   | Hostname               | Client's name, sent by the client                             |

### 🔗 Why This Matters for Our Lab

Remember from Lab 02 that a workstation **must** point at `ad01` (`10.0.5.5`) for DNS or it can't join the domain? If DHCP hands out the wrong **Option 6**, every client it serves breaks the same way.

Also from Lab 02: `wks01` couldn't ping plain `fw01`, only `fw01.yourname.local`, because it had no DNS suffix. **Option 15** is exactly what fixes that automatically for every DHCP client.

### 🦈 In Wireshark

* Filter: `dhcp.option.dhcp == 5` to grab ACKs, then expand **Dynamic Host Configuration Protocol** in the packet details pane. Each option shows up as its own line like `Option: (6) Domain Name Server`.
* Filter by a specific option value: `dhcp.option.domain_name_server == 10.0.5.5`
* On the client's **Discover**, expand **Option 55** to see its wish list. Windows and Linux ask for different things, which is actually a fingerprinting trick (tools can guess the OS from this list alone).

{% hint style="success" %}
**Recommendation:** When you set up the DHCP scope, always configure Options 3, 6, and 15 at minimum, and point Option 6 at the **domain controller**, not the firewall. Pointing at the firewall "works" for internet but silently breaks domain joins.
{% endhint %}

## 2️⃣ The Lease Lifecycle: Renew, Rebind, Release, NAK

DORA only happens on the first connection. After that, the client quietly maintains its lease with a smaller conversation.

| Timer              | When                | What Happens                                                                                                |
| ------------------ | ------------------- | ----------------------------------------------------------------------------------------------------------- |
| **T1 (Renewal)**   | 50% of lease time   | Client sends a **Request directly (unicast)** to the server that gave it the lease. Just "can I keep this?" |
| **T2 (Rebinding)** | 87.5% of lease time | Original server didn't answer, so the client **broadcasts** a Request to _any_ server.                      |
| **Expiry**         | 100%                | Client drops the IP and starts over with a Discover.                                                        |

Two other messages worth knowing:

* **Release (type 7):** Client politely gives the IP back early. This is what `ipconfig /release` sends.
* **NAK (type 6):** Server says "no, you can't have that." Usually means the client moved to a different network and is asking for an IP that doesn't belong there.

### 🦈 In Wireshark: How to Tell a Renewal From a Fresh DORA

| Clue                                 | Fresh DORA        | Renewal                |
| ------------------------------------ | ----------------- | ---------------------- |
| Source IP of Request                 | `0.0.0.0`         | Client's current IP    |
| Destination                          | `255.255.255.255` | Server's IP (unicast)  |
| `ciaddr` field ("Client IP address") | `0.0.0.0`         | Filled in              |
| Discover/Offer present?              | Yes               | No, just Request + ACK |

* Look at Options **51, 58, 59** in the ACK for the lease, T1, and T2 times.
* Filters: `dhcp.option.dhcp == 7` for Releases, `dhcp.option.dhcp == 6` for NAKs.

### 🖥️ Check Your Lease From the Client

* **Windows:** `ipconfig /all` shows _Lease Obtained_ and _Lease Expires_.
* **Rocky:** `nmcli -f DHCP4 device show <interface>` dumps every option the server gave you.

{% hint style="success" %}
**Recommendation:** Keep leases short (hours) on networks where devices come and go, like guest Wi-Fi, and longer (days) on stable office LANs. And any **server** (like `ad01` or `dhcp01` itself) should have a **static IP or a reservation**, never a normal lease.
{% endhint %}

## 3️⃣ Rogue DHCP and DHCP Starvation (The Security Side)

DHCP has **zero authentication**. The client trusts whichever server answers first. That makes it one of the easiest protocols on a LAN to abuse.

### 🏴‍☠️ Rogue DHCP Server

An attacker (or someone's home router plugged into the wrong jack) starts answering Discovers. If it wins the race, it can hand out itself as the **gateway (Option 3)** or **DNS server (Option 6)**, putting the attacker in the middle of all the victim's traffic.

### 🍽️ DHCP Starvation

The attacker floods Discovers from thousands of **fake MAC addresses**. The real server hands out an address to each one until the scope is empty. Real clients get nothing. Often used as step one: starve the real server, _then_ bring up a rogue one so it's the only server left answering.

### 🦈 In Wireshark

| What You're Hunting    | How to Spot It                                                                                                                     |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Rogue server**       | Filter `dhcp.option.dhcp == 2` (Offers). Add **Option 54 (Server Identifier)** as a column. More than one server IP answering = 🚩 |
| **Starvation**         | Filter `dhcp.option.dhcp == 1`. A flood of Discovers in a few seconds, each with a different `dhcp.hw.mac_addr` (Client MAC).      |
| **Quick sanity check** | _Statistics → Conversations_ or _Statistics → Endpoints_ to see how many unique MACs are talking DHCP.                             |

{% hint style="info" %}
**Pro tip:** Right click the _Server Identifier_ field in any Offer and choose **Apply as Column**. Rogue servers jump out immediately when you can see every Offer's source side by side.
{% endhint %}

### 🛡️ Defenses

* **DHCP snooping** on switches: only trusted ports (the ones leading to real DHCP servers) are allowed to send Offers and ACKs.
* **Port security:** limit how many MAC addresses a single switch port can learn, which kills starvation.
* **Dynamic ARP Inspection (DAI):** builds on the snooping table to also block ARP spoofing.
* **Windows DHCP authorization:** In AD, a Windows DHCP server has to be authorized in Active Directory before it will serve clients. Doesn't stop a Linux box or a random router, but stops accidental rogue Windows servers.

{% hint style="success" %}
**Recommendation:** Safe way to check your own lab for extra DHCP servers is `nmap --script broadcast-dhcp-discover`. It sends a Discover and lists every server that responds. **Only run attack tools like Yersinia inside an isolated lab you own**, never on the campus network.
{% endhint %}

## 🧠 Gotchas

* A new client's first packets come from `0.0.0.0`. If you filter by the client's IP in Wireshark, you'll **miss the whole DORA**. Filter by `dhcp` or by MAC (`eth.addr == xx:xx:xx:xx:xx:xx`) instead.
* DHCP broadcasts **don't cross routers**. If the DHCP server is on a different subnet than the clients, the router needs a **DHCP relay** (`ip helper-address` on Cisco) or clients get nothing.
* Getting an IP starting with `169.254.x.x` means the client never heard back from any DHCP server. That's an **APIPA** self-assigned address, not a real lease.
* Renewals are **unicast** and easy to overlook in a capture because there's no Discover/Offer in front of them.

## 🧰 3 Handy DHCP Commands

* **`ipconfig /displaydns`** (Windows): after a lease, shows what the client has already resolved and cached using the DNS server DHCP gave it. Good for confirming Option 6 actually took.
* **`sudo tcpdump -i <interface> -n port 67 or port 68 -v`** (Linux): watch DORA live in the terminal without opening Wireshark. Great on a headless box like `dhcp01`.
* **`sudo journalctl -u dhcpd -f`** (Rocky DHCP server): follows the DHCP server's log live so you can watch every `DHCPDISCOVER`, `DHCPOFFER`, and `DHCPACK` as the server sees it. Pairs nicely with a Wireshark capture on the client side.
