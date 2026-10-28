# lab apache firewall

**Goal:** Lock down SSH, install the Apache web server, open the right firewall ports, serve a custom page, and get PHP running.

## Key Terms (Plain English)

| Term                     | What it means                                                             |
| ------------------------ | ------------------------------------------------------------------------- |
| httpd                    | The Apache web server service on Red Hat family Linux                     |
| firewalld / firewall-cmd | The Linux firewall and the command used to change its rules               |
| Service (firewall)       | A named bundle of ports. `http` is 80/tcp and `https` is 443/tcp          |
| Permanent rule           | A firewall rule that survives reboots. Needs a reload to apply right away |
| Web root                 | The folder Apache serves files from. It is `/var/www/html/`               |
| PHP                      | A scripting language that lets a web page build its content on the fly    |
| PermitRootLogin          | The SSH setting that decides if root can log in remotely                  |

## What I Actually Did

**1. Disable root login over SSH**

In `/etc/ssh/sshd_config` I set:

```
PermitRootLogin no
```

```bash
sudo systemctl restart sshd
```

From then on I logged in as my named sudo user and used `sudo -i` for root.

**2. Install Apache**

```bash
sudo dnf install -y httpd
```

Rocky 9 uses `dnf`. The lab screenshots show `yum`, which is the older CentOS 7 tool.

**3. Open the firewall**

I used the named services instead of raw ports, and made them permanent.

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

The services line now shows `cockpit dhcpv6-client http https ssh`.

**4. Start Apache and make it survive reboots**

```bash
sudo systemctl enable --now httpd
systemctl status httpd
```

From wks01 I browsed to `http://web01-mathew` and got the Rocky Linux HTTP Server Test Page.

**5. Replace the test page**

The test page comes from `welcome.conf`. I removed it and added my own page.

```bash
sudo rm /etc/httpd/conf.d/welcome.conf
echo "<h1>Welcome to web01-mathew</h1>" | sudo tee /var/www/html/index.html
sudo systemctl restart httpd
```

**6. Add PHP**

```bash
sudo dnf install -y php
sudo systemctl restart httpd
```

Apache has to restart or it will not pick up PHP. I wrote `/var/www/html/index.php` with a loop that prints "HELLO SYS255" and then X from 1 to 10. Browsing to `http://web01-mathew/index.php` showed the rendered output.

## Deliverables Captured

* Deliverable 1: SSH from AD02 as a named user, `sudo -i`, nslookup and ping
* Deliverable 2: `firewall-cmd --list-all` showing http and https
* Deliverable 3: wks01 browsing to web01 by hostname (test page)
* Deliverable 4: custom welcome page with VM name and hostname in the URL
* Deliverable 5: PHP code next to the rendered page

## 3 Handy Commands

```bash
sudo ss -tlnp | grep httpd
```

Shows Apache is actually listening on port 80 (and 443 if enabled).

```bash
sudo httpd -t
```

Checks the Apache config for typos before you restart.

```bash
sudo journalctl -u httpd --no-pager | tail -20
```

Shows the most recent Apache log lines when something will not start.
