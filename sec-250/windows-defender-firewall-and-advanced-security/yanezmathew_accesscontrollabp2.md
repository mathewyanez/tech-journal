# YanezMathew\_AccessControlLabP2

![](<../../.gitbook/assets/Unknown image>)

**Access Control Lab Part 2**

**Objective:**

In this activity, you will gain practical experience with advanced firewall configuration and security monitoring. You'll analyze system logs from multiple sources (IIS and Windows Event Viewer) to investigate simulated security incidents and create forensic timelines of suspicious activity.

**Instructions:**

1. **More Network Access Control**
2. Open Windows Firewall Advanced Settings again
3. Look at the top of the window - **Deliverable 1:** Answer the following questions
   1. What does "Inbound connections" show for each profile (Domain, Private, Public)?

**It shows the name group, profile, whether or not it's enabled along with authorized permissions, computers, and ports.**



1. What is the default behavior… block or allow?

**From the large amount of allows, going to go with allow.**



1. Why might this default make sense from a security perspective?**It saves a lot of time, but from a security perspective it makes sure that services that require these services to function are able to, and that way rules that need not be allowed can be cut off at a later date.**
2. Create a new inbound rule to block Remote Desktop.

* New Rule → Port → TCP → Specific port: 3389
* Block the connection → All profiles → Name it "Block RDP"

2. Create a new inbound rule to block a specific neighbor's IP completely:

* New Rule → Custom → All programs
* Any protocol → Scope tab
* Under "Remote IP address" → "These IP addresses" → Add \[neighbor's IP]
* Under "Local IP address" → "These IP addresses" → Add \[neighbor 2’s IP]
* Block → All profiles → Name it "Block \[Neighbor 2’s Name]"
* **Deliverable 2:** Take a screenshot showing your custom firewall rules list
* ![](<../../.gitbook/assets/Unknown image (1)>)

3. Test both rules:

* Neighbor 1 should now attempt an RDP session to the middle computer.
* Have Neighbor 2 attempt to ping the middle computer.
* \*\*Deliverable 3:\*\*Take a screenshot of a failed RDP session for Neighbor 1 or Neighbor 2.

\=![](<../../.gitbook/assets/Unknown image (2)>)

* \*\*Deliverable 4:\*\*Take a screenshot of a failed ping from the blocked IP address.

![](<../../.gitbook/assets/Unknown image (3)>)

*
  1. Delete/disable all your custom rules before continuing

1. **Log Correlation & Attack Timeline**

Now you'll act as a security analyst investigating a simulated incident.

*
  1. Have Neighbor 1 perform these actions in order:
  * Successfully access your web1.html three times
  * Try to RDP with wrong password three times
  * Successfully RDP with correct credentials one time

![](<../../.gitbook/assets/Unknown image (4)>)

*
  *
    * Access your web1.html once more while RDP'd in
  1. Now re-enable your firewall rule blocking port 80 and your rule blocking RDP.
  2. Have Neighbor 2:
  * Attempt to access your web1.html five times
  * Attempt RDP two times
  3. Review and analyze IIS logs
  * Open File Explorer and navigate to: C:\inetpub\logs\LogFiles\W3SVC1\\
  * Open the most recent log file (named with today's date, format: u\_exYYMMDD.log). You can open this in Notepad. It will show all HTTP requests to your web server
  * Look for columns including:
    1. date and time - when the request occurred
    2. c-ip - client IP address (who made the request)
    3. cs-uri-stem - the file requested (like /web1.html)
    4. sc-status - HTTP status code (200 = success, 404 = not found)
  4. Review and analyze logs in Windows Event Viewer
  * Press Windows Key + R, type eventvwr.msc, press Enter
  * Navigate to: Windows Logs → Security
  * Click Filter Current Log in the right panel
  * In the Event IDs field, type: 4624,4625 and click OK
    1. Event ID 4624 = Successful login
    2. Event ID 4625 = Failed login attempt
  * Look for these details in each event:
    1. Date and Time
    2. Source Network Address (the IP trying to connect)
    3. Logon Type (Type 3 or Type 5 = Network Login)
    4. Account Name (username used)
  5. \*\*Deliverable 5:\*\*Create an Attack Timeline (well, what would be an attack timeline if we had been attacked…). Fill in entries for:
  * All web access attempts (successful and blocked)

Was blocked multiple times on own system, RDP’d into their system and gained access to it.

* All RDP attempts (successful and failed)

Made about 5 attempts but could not make it in.

Made it in the last attempt.

* **Example:**

![](<../../.gitbook/assets/Unknown image (5)>)

*
  1. \*\*Deliverable 6:\*\*Answer these analysis questions
  * What activity shows up in IIS logs but NOT in Event Viewer?

Http Requests, unlike event viewer which only displays system processes.

* What activity shows up in Event Viewer but NOT in IIS logs?

Vice Versa, system processes.



* Which neighbor's blocked web attempts appear in your logs? Why?

1’s were completely blocked firewall wise, while the other ones were not rejected due to wall so showed in logs.

* If you saw 50 Event ID 4625 entries in 2 minutes, what would that indicate?

Someone is trying to break in.

* How many failed login attempts before you'd investigate?

Depends but I’d say over 10 within a very very short period of time could be suspicious while throughout a time period would not require investigation.
