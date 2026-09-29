# Copy of Module 02\_ A Deeper Look at DNS

Assignment - A Deeper Look at DNS

| 💡 Networking is key to systems administration. In your previous classes (Network Fundamentals), you should have learned a bit about how DNS works and had the opportunity to use wireshark. These skills should be part of your systems administration toolkit. It's hard to debug a networking problem if you don't know what is going on under the hood. DNS problems are the root of many systems administration challenges. You may have experienced this when trying to join WKS01 to the domain.If you've forgotten some of these techniques, some self study will be required to get on track. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

### Install Wireshark

Log off and login to wks01 again as a named domain admin user, and install Wireshark on WKS01.

### Capture DNS Traffic

1. Open Wireshark
2. Open an elevated command prompt on wks01 (right-click over Command Prompt or Powershell App)<br>
3. Release any Cached DNS records<br>
4. Begin a Capture in WIreshark on Ethernet0

* Hint: Look for the connection with network activity -
* Use Case 1: Ping a non-existent site (yourname\_abc.edu)
* Use Case 2: Ping an existing local host (fw01-yourname)
* Use Case 3: Ping an existing remote site (champlain.edu)

1. Stop and save the Capture.

| 💣 Reminder: This is an assignment, so you are on your own for answering the deliverables section. |
| -------------------------------------------------------------------------------------------------- |

### Deliverables

**In your capture, what are the destination IP addresses, ports and protocols for DNS traffic?**

**The Destination IP address/port is 10.0.5.2 and port 53 with the protocols being IPV4, and UDP.**

**Perform some basic research. Does DNS ever use a protocol different than the one found in Deliverable 1? If so, why?**

**Yes, after taking a quick search I found out that DNS uses different ports for encrypted protocols. Port 443 for example will be used for DNS over HTTPS which hides DNS requests inside normal encrypted web traffic. Port 853 will also be used for DNS over TLS which encrypts DNS queries directly with a dedicated privacy protocol.**

**Figure out how to create a display filter showing only DNS traffic, provide a screenshot that shows at least six DNS packets.**

**For use case 1, what are the authoritative name servers for the .edu top level domain?**

**A\[.]edu-servers\[.]net through m\[.]edu-servers\[net]**

**For use case 1, Provide a screenshot that shows the reply code from your .edu lookup (note this will be part of the flags field)**

NEver received a response, canceled the ping before it ever received one was taking forever to send anything. No reply code.

Here is flag query for one of the packets:

**For use case 2, provide a screenshot showing the Answer's section of the DNS response for the fw01-yourname query.**



**For use case 3, what server responds to the dns request for champlain.edu, & is it authoritative?**

**So question 7 stands as I had it:**



**The server that responds to the champlain.edu query is 10.0.5.2, my router's LAN interface (also the DNS forwarder configured on it). It is not authoritative for champlain.edu. The actual authoritative servers for champlain.edu are further up that chain, outside what my local capture can see.**

**What are the different types of DNS Records? Provide a brief description of each in your own words. There are dozens of DNS record types, so discuss the more common ones (< 10 of them) but more than (CNAME,A,PTR) discussed in class.**

MX: tells email where to go, like a address for a house.

NS: says who's in charge of answering questions about a domain.

TXT: just a little note attached to a domain, used to prove things or leave instructions.

SOA: says who runs the domain and how often to check for updates.

AAAA: same as an A record, but for the newer, longer kind of address (IPv6) instead of the older short one.

SRV: points to exactly which door and which room a specific service lives at, not just the house.

CAA: a rule saying only certain companies are allowed to hand out the domain's security certificate.

DNSKEY: a secret key stamp that proves the domain's answers are real and not faked.

**Your deliverable meets the submission** [**guidelines**](https://docs.google.com/document/d/1Mbjso3I5UD5bAm2SZisjxO6aqM3ao9mRbqnbAAfjmto/edit)**.**
