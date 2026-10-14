# Lab05 Server Core File Server Tech Journal

**Goal:** Build a Server Core machine (FS01-mathew) with no desktop, make it a file server, join it to the domain, manage it remotely from AD02 with RSAT, then lock a Sales share to one group and map it as the S: drive with Group Policy.

## Key Terms (Plain English)

**Server Core** Windows Server with no desktop, no Start menu, nothing to click. Just a command line. Fewer things installed means less to patch and less to attack, which is why people actually run it in production.

**sconfig** The little numbered menu that Server Core boots into. Handles the basics: computer name, IP, domain join, updates. Think of it as the settings app for people who gave up on settings apps.

**RSAT (Remote Server Administration Tools)** The management consoles you install on one machine so you can run other machines from it. You never need to sit at the file server or log into it directly. Important detail: it's a Windows _feature_, not a _role_.

**File Server role / FSRM** The File Server role is what turns a server into a file server. FSRM (File Server Resource Manager) is the add on for quotas, file screening, and storage reports.

**SMB share** A folder shared over the network. You reach it with \\\servername\sharename.

**Share permissions vs NTFS permissions** Two separate locks on the same door. Share permissions only apply when someone connects over the network. NTFS permissions apply to the folder itself, no matter how you get to it. If both exist, the most restrictive one wins. Least privilege, doing its thing.

**Security group (Global)** A bucket of users. You give the bucket permissions once instead of giving every person permissions one by one.

**Group Policy Preferences (Drive Maps)** A Group Policy setting that maps a network drive for users automatically. It's a _preference_, not an enforced policy, so the user can change it afterward. Fine for a mapped drive.

**Item-level targeting** A filter you attach to one preference item. "Only apply this if the user is in this group." Lets one GPO act differently for different people without building extra OUs.

**UAC elevation** Even if you're an admin, a normal PowerShell window runs with your admin rights stripped out. You have to open it with Run as administrator to get them back. This one shows up in the gotchas.

## What I Actually Did

**1. Booted the Server Core machine and looked at sconfig.** Fresh install: workgroup, random computer name, and an address from DHCP. Nothing is set up yet.

**2. Grabbed the network values from DNS Manager on AD02.** Before touching FS01 I checked what the domain already uses. ad02-mathew is 10.0.5.6 (DNS), fw02-mathew is 10.0.5.2 (gateway), and the DHCP server at 10.0.5.33 is not DNS, so I left it out of this.

**3. Set a static IP in sconfig.** Option 8, pick the adapter, set it static: 10.0.5.8, mask 255.255.255.0, gateway 10.0.5.2, DNS 10.0.5.6. A file server needs an address that doesn't move.

**4. Renamed the computer to FS01-mathew.** Option 2, enter the name, say yes to the restart.

**5. Tested DNS before the domain join. It failed.** Ping to the DC worked, but nslookup mathew.local came back with UnKnown and a fec0:0:0:ffff::1 address. That address is what Windows shows when no DNS server is set. So the DNS I typed into sconfig never actually stuck.

Fixed it in PowerShell:

Get-NetAdapter\
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.5.6

After that, nslookup answered from ad02-mathew.mathew.local.

**6. Joined the domain.** Option 1, D for domain, mathew.local, then the -adm credentials. Said no to renaming the computer and yes to the restart.

**7. Added FS01 to Server Manager on AD02. This is where things got fun.**

First try: Target name resolution error. AD02 had no idea what FS01-mathew was, because FS01 never created its own DNS record.

Created the A record by hand in DNS Manager with the PTR box checked. (ipconfig /registerdns on FS01 would also have worked.)

Second try: Online - Access denied. Great, it can see the server now, it just doesn't like me.

The problem was that I never restarted FS01 after the join, so I was still signed in as the local Administrator.

Restarted, signed in with Other user as my domain admin account, and it connected.

**8. Added the File Server role to FS01 from AD02.** Add Roles and Features, and on the server selection page I picked **FS01**, not AD02. Under File and iSCSI Services I checked File Server and File Server Resource Manager.

**9. Opened the firewall on FS01 for remote FSRM.** On FS01 in PowerShell:

netsh advfirewall firewall set rule group="Remote File Server Resource Manager Management" new enable=yes

**10. Got the FSRM console working on AD02.** It wasn't in the Tools menu. The role lives on FS01, but the console that manages it comes from RSAT on AD02.

Install-WindowsFeature RSAT-File-Services said NoChangeNeeded because that's only the parent feature. The FSRM tools underneath it were what was missing.

When FSRM first opened it threw a WMI error, because it was trying to manage AD02 itself, which doesn't have the role. Connect to Another Computer fixes that.

**11. Built the AD objects on AD02.** SYS255 OU with Users, Groups, and Computers inside it. A global security group called Sales-Users in Groups. Two standard users, Alice and Bob, in Users. Alice goes in Sales-Users. Bob does not, because he's the test case.

**12. Created the Sales share.** Server Manager, File and Storage Services, Shares, New Share. SMB Share Quick, server FS01, name Sales. The local path is C:\Shares\Sales and the network path is \\\FS01-mathew\Sales.

**13. Fixed the permissions. This is the part that matters.** New folders inherit permissions from the parent, and that included the built in Users group. Every domain user is in that group, so Bob would have been able to read the share even though he's not in Sales-Users.

Disabled inheritance, converted the permissions to explicit ones, removed both Users entries, and added Sales-Users with Full control. On the Share tab I removed Everyone and added Sales-Users there too.

**14. Tested as Alice and Bob on the workstation.** Alice created a file in the share and opened it back up.

Bob got the error I was hoping for.

**15. Built the GPO to map S:.** Group Policy Management, new GPO called Map Sales Drive linked to SYS255\Users. Then User Configuration, Preferences, Windows Settings, Drive Maps, new Mapped Drive: Action **Create**, location \\\fs01-mathew\Sales, reconnect checked, drive letter S:.

On the Common tab I turned on item-level targeting and set it to the security group MATHEW\Sales-Users. Without this, both Alice and Bob would get the drive, since they're in the same OU.

**16. Verified it as Alice.** gpupdate /force, logged off and back on, then gpresult /r /scope:user. The output shows her in the right OU, Map Sales Drive applied, and Sales-Users in her groups.

And the drive is there, with her test file already on it.

## Deliverables Captured

* Screenshot: sconfig on FS01 showing the network, computer name, update setting, and joined domain
* Screenshot: All Servers on AD02 showing AD02 and FS01 online
* Screenshot: FSRM connected to FS01 with the VM banner
* Screenshot: Alice reading and writing in \\\fs01-mathew\Sales
* Screenshot: Bob getting access denied
* Screenshot: gpresult showing Map Sales Drive applied
* Screenshot: the mapped S: drive

## 3 Handy Commands

**Get-SmbShareAccess -Name Sales** Lists who has access to a share and at what level, straight from PowerShell. A quick way to check the share side of the permissions without opening the GUI.

**gpresult /h report.html** Same data as gpresult /r, but it writes a full HTML report you can open in a browser. Shows which GPOs applied, which got filtered out, and why.

**Test-NetConnection fs01-mathew -Port 445** Checks that the server answers on the SMB port. If a share won't open, this tells you right away whether it's a network problem or a permissions problem.
