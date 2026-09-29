# MathewYanez Lab 03   Linux

Lab 03 Linux Configuration

| 💡It is very rare that you will find an enterprise that is either fully Windows, Mac or Linux. You will likely find a heterogeneous environment where many different operating systems are leveraged to accomplish the organization's mission. In this lab, you will configure an operating system called RockyOS. This particular operating system is open source and has been pre-built for you. Your job will be to complete the configuration steps to make it useful and manageable in your growing enterprise. |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### Networking dhcp01

Find dhcp01 Virtual Machine in your Proxmox environment, & configure the network so that it is using your internal LAN segment. And don’t forget the Snapshot prior powering dhcp01 on, if you like to have a backup before changing configurations.

| 💡Heads up: Starting with Week 03, reducing steps you covered past two weeks. |
| ----------------------------------------------------------------------------- |

The following video describes the process of setting a hostname and IP address, though many students appreciate the **nmtui** application. For those having trouble, the following [video tutorial](https://drive.google.com/file/d/159rkSPeDkNa1tEDCzLfifZNbSoiXVGUq/view?usp=sharing) shows the process of setting the networking configuration.

dhcp01 network settings:

| Setting                | Value           |
| ---------------------- | --------------- |
| IP Address and Netmask | 10.0.5.3/24     |
| Gateway                | 10.0.5.2        |
| DNS                    | 10.0.5.5        |
| Search Domain          | yourname.local  |
| Hostname               | dhcp01-yourname |

### Adding a privileged user

Figure out how to add a named user who is a member of the “[wheel](https://en.wikipedia.org/wiki/Wheel_\(computing\))” group (Linux’s local admin group on Rocky OS). Hint: The linked video above also shows how to do this.

### Networking Test

If you did everything right, then you should be able to ping systems inside 10.0.5.0/24 and outside (SYS-255-WAN) of your network.

**Deliverable 1. Login as the named user (not root!) and attempt to ping google.com, ad01 and fw01. Provide a screenshot similar to the one below that shows the three successful pings.**



### DNS

Take a look at last week's lab, and figure out how to add A and PTR records for dhcp01 to the DNS configuration on ad01. Test this by issuing a ping from wks01 to dhcp01 using the undistinguished (read: short) hostname.

**Deliverable 2. Using WKS01, Provide a screenshot showing the successful ping using dhcp01's hostname only (leave off yourname.local).**

### Remote Access from ad01

Systems Administrators will typically manage linux systems remotely via SSH (Secure Shell). An application called PuTTY was popular for this purpose and can still be optionally installed. Fortunately, Windows 10 now ships with an SSH client and we’ll use this.

**Deliverable 3. Provide a screenshot that shows a successful ssh session as your named Linux user from wks01:**

### Getting around and sudo

When you login via SSH or locally, you land in the logged-in users home directory (\~). The pwd command shows you this.

\| Show the present working directory | | | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | | | Navigate up to the /home directory and list the contents | | | Navigate up to the parent directory using a relative _cd .._ command. It is relative to where you are in the directory structure. | | | long listing of files and directories with ls -l. | | | show the manual page indicating what each directory in the file hierarchy is used for. man hier. Read the description of the first level directories. | | | Use the "tilde" shortcut to go to the home directory \~ | | | Create and navigate to a directory called sys255 in your home directory | | | Try to install the "tree" package as the named non root user. This should fail because your named account does not have privileges to install software. | | | Elevate privileges using the sudo command. This will only work if you set your named user to be an administrator. The sudo command run this way will execute a single command as a privileged user then drop you back down to normal permissions. | | | Show the groups your user has been assigned to. In this case, the wheel group is analogous to the Administrator's group in Windows. | | | Become root for an extended time with sudo -i. This is necessary if you have a lot to do in a privileged state, be sure to "exit" the root shell when you are done. A second exit will probably close your SSH session.The whoami command will show what user you are logged in as. | |

### History

Sometimes it is useful to see the history of those things you've typed at the command line (this works in powershell too!). Type history to see what commands have been typed.

**Deliverable 4: Provide the first 10 commands recorded in your history file:**

Hidden Files: Go to your home directory and do a normal ls. Follow that by the ls -la command. This command lists those hidden files (those that start with a period.)

View .bash\_history (if it is missing, logout of ssh and log back in again).

**Deliverable 5. This is a two part question:**

**What security implications does this file represent? List at least one pro and one con.**

**A pro is that if hacker do hacker thing it leaves a snail trail, on the bad side if someone wanted to obtain confidential information past commands might sneak some information to them.**

**What command is used to clear bash history?**

**history -c && history -w**

**Deliverable 6.This is the final reminder for specs, and is expected for the remaining labs.**

**Deliverable 7. Tech Journal entry reminder. There are loads of Linux commands, so go exploring for at least 3 new ones. Also, their&#x20;**_**man**_**&#x20;page is your new BFF.**
