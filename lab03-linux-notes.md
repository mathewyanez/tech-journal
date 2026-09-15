# lab03 linux notes

> **Goal:** Bring up `dhcp01` running RockyOS, get it networked on the internal LAN, create a proper non root admin user, wire up DNS, and get comfortable navigating and elevating privileges on the command line.

## 📖 Key Terms (Plain English)

| Term                      | What It Actually Means                                                                                                                                                         |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **RockyOS (Rocky Linux)** | A free, open source Linux distribution built to be a drop in replacement for enterprise Linux. Common in server environments.                                                  |
| **wheel group**           | Linux's version of the "Administrators" group in Windows. Adding a user to `wheel` lets them use `sudo`.                                                                       |
| **sudo**                  | Lets a normal, non root user run a single command with elevated (root) privileges, then drops them back to normal permissions afterward.                                       |
| **root**                  | The Linux superuser account. Full control over everything, similar to Domain Admin in Windows but local to the machine. Best practice is to avoid logging in directly as root. |
| **SSH (Secure Shell)**    | The standard way to remotely and securely manage Linux systems from another machine. Windows 10 and later ship with a built in SSH client.                                     |
| **pwd**                   | "Print working directory." Shows you exactly where you are in the filesystem.                                                                                                  |
| **Home directory (`~`)**  | The personal folder you land in when you log in, shorthand is the tilde symbol.                                                                                                |
| **Hidden files**          | Files or folders starting with a period, like `.bash_history`. Not shown by a normal `ls`, only with `ls -la`.                                                                 |
| **.bash\_history**        | A log file of every command you have typed in that shell session. Useful for troubleshooting, but also a potential security risk if it captures sensitive info.                |

## 🛠️ What I Actually Did

{% stepper %}
{% step %}
## Found `dhcp01` in Proxmox and configured its network

Configured it to sit on the internal LAN segment. Took a snapshot first, before powering it on or changing any configuration, in case something needed to be rolled back.
{% endstep %}

{% step %}
## Set the hostname and IP address

Used `nmtui` (a friendlier text based UI for network settings) rather than editing config files by hand.

**dhcp01 network settings:**

| Setting              | Value           |
| -------------------- | --------------- |
| IP Address / Netmask | 10.0.5.3/24     |
| Gateway              | 10.0.5.2        |
| DNS                  | 10.0.5.5        |
| Search Domain        | yourname.local  |
| Hostname             | dhcp01-yourname |
{% endstep %}

{% step %}
## Created a named, non root user

Added them to the `wheel` group so they could use `sudo` without logging in as root directly.
{% endstep %}

{% step %}
## Tested networking

Logged in as the named user (not root) and pinged `google.com`, `ad01`, and `fw01` to confirm both internal and external connectivity.
{% endstep %}

{% step %}
## Added DNS records for dhcp01

Added A and PTR records on `ad01`, then confirmed it worked by pinging `dhcp01` from `wks01` using just the short hostname, no domain suffix needed.
{% endstep %}

{% step %}
## Connected over SSH

Connected from `wks01` to `dhcp01` as the named Linux user, using the built in Windows SSH client instead of a third party tool like PuTTY.
{% endstep %}

{% step %}
## Practiced core navigation and privilege commands

* `pwd` to confirm current location
* `cd /home` and `ls` to browse
* `cd ..` to move up a directory, relative to current location
* `ls -l` for a long listing of files and directories
* `man hier` to read about what each top level directory is used for
* `cd ~` to jump straight back to the home directory
* `mkdir sys255 && cd sys255` to create and move into a new folder
* Attempted to install the `tree` package as the regular user, confirmed it failed due to lack of privileges
* `sudo <command>` to run that same install as root, just for a single command
* `groups` to confirm which groups the user belongs to (wheel = admin equivalent)
* `sudo -i` to become root for an extended session, then `exit` to leave root (a second `exit` closes the SSH session entirely)
* `whoami` to confirm which user is currently active
{% endstep %}

{% step %}
## Reviewed command history

Used the `history` command (also works in PowerShell), and separately inspected `.bash_history` directly by first running a normal `ls`, then `ls -la` to reveal hidden files.
{% endstep %}
{% endstepper %}

## 💣 Gotchas / Things That'll Trip You Up

{% hint style="warning" %}
* Always snapshot a VM **before** changing its configuration, not after.
* Never do your regular work logged in as `root`. Use a named user in the `wheel` group and elevate with `sudo` only when needed.
* DNS records do not create themselves. Forward (A) and reverse (PTR) records both need to be added manually on the DNS server.
* `cd ..` is relative, it depends entirely on where you currently are in the directory tree.
* `sudo` runs one command with elevated rights, `sudo -i` opens an entire root shell. Don't forget to `exit` out of it when finished.
* `.bash_history` is a double edged sword: great for accountability and catching intrusions, but a risk if it captures sensitive commands or data. It can be wiped with `history -c && history -w`.
{% endhint %}

## 🧠 Reflection Questions

<details>

<summary>What security implications does <code>.bash_history</code> represent?</summary>

Pro: it leaves a trail, so if an attacker (or a careless user) runs commands, there is a record of it afterward.

Con: if someone gains access to that file, past commands could leak confidential information they were never supposed to see.

</details>

<details>

<summary>What command clears bash history?</summary>

`history -c && history -w`

</details>

## 📸 Deliverables Captured

* Screenshot: three successful pings (google.com, ad01, fw01) as the named non root user
* Screenshot: successful ping from wks01 to dhcp01 using just the short hostname
* Screenshot: successful SSH session as the named Linux user from wks01
* Screenshot: first 10 commands from `history`
* Screenshot: `ls` vs `ls -la` showing hidden files
* Screenshot: contents of `.bash_history`

## 🔍 3 Interesting Linux Commands Worth Knowing

* **`ncdu`** — an interactive disk usage analyzer. Instead of squinting at `du -sh` output, it gives you a navigable, color coded breakdown of what is actually eating up disk space, folder by folder.
* **`htop`** — a much friendlier, color coded upgrade over the classic `top` command. Shows live CPU, memory, and process activity, and lets you sort, search, and even kill processes right from the interface.
* **`tldr`** — pulls up short, example driven cheat sheets for a command instead of a full `man` page. Great for a quick reminder of common usage without reading pages of documentation.
