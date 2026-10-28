# lab linux domain join

**Goal:** Give web01 a static IP and DNS records, then join it to the `mathew.local` Active Directory domain so I can log in with AD accounts instead of only local Linux users.

## Key Terms (Plain English)

| Term                  | What it means                                                                                            |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| Domain join           | Telling a computer to trust Active Directory for logins, so AD accounts work on it                       |
| realmd / realm        | The Linux tool that handles the domain join for you                                                      |
| SSSD                  | The background service that lets Linux look up AD users and check their passwords                        |
| Kerberos              | The login system AD uses behind the scenes. It is picky about uppercase and lowercase in the domain name |
| adcli                 | A helper tool that realm uses to create the computer account in AD                                       |
| oddjob-mkhomedir      | Automatically makes a home folder the first time an AD user logs in                                      |
| A record / PTR record | A is name to IP. PTR is IP to name                                                                       |
| Static IP             | An address I set by hand so it never changes                                                             |

## What I Actually Did

**1. Set a static IP on web01**

The connection was named "Wired connection 1" on device ens18. nmcli needs the address set before it will accept the manual method.

```bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.5.10/24
sudo nmcli con mod "Wired connection 1" ipv4.method manual
sudo nmcli con mod "Wired connection 1" ipv4.gateway 10.0.5.2
sudo nmcli con mod "Wired connection 1" ipv4.dns 10.0.5.6
sudo nmcli con mod "Wired connection 1" ipv4.dns-search mathew.local
sudo nmcli con up "Wired connection 1"
```

Then I confirmed it with `ip a` and pinged AD02 at 10.0.5.6.

**2. Set the hostname**

```bash
sudo hostnamectl set-hostname web01-mathew
```

**3. Add the DNS records on AD02**

In DNS Manager I right-clicked `mathew.local` and chose New Host (A). Name `web01-mathew`, IP `10.0.5.10`, with "Create associated pointer (PTR) record" checked.

**4. Test DNS from web01**

Rocky does not ship nslookup, so I installed it first.

```bash
sudo dnf install -y bind-utils
nslookup web01-mathew
nslookup 10.0.5.10
```

Forward returned 10.0.5.10 and reverse returned web01-mathew.mathew.local.

**5. Check the clock**

Kerberos breaks if the time is off by more than a few minutes, so I checked that web01 and the DC agree.

```bash
timedatectl
```

Both were on America/New\_York with the clock synced.

**6. Install the join packages**

The lab lists these:

```bash
sudo dnf install -y realmd samba samba-common oddjob oddjob-mkhomedir sssd
```

The join also needed `adcli`, which the lab does not list:

```bash
sudo dnf install -y adcli
```

**7. Join the domain**

The realm name has to be uppercase. With lowercase I got "KDC reply did not match expectations".

```bash
sudo realm join --user=mathew.yanez-adm@MATHEW.LOCAL MATHEW.LOCAL
realm list
```

`realm list` showed `configured: kerberos-member` and `login-formats: %U@mathew.local`, which means AD users log in as `user@mathew.local`.

**8. Log in as an AD user**

From AD02 over SSH:

```bash
ssh mathew.yanez-adm@mathew.local@web01-mathew
id
whoami
pwd
```

`whoami` returned `mathew.yanez-adm@mathew.local` and `pwd` returned `/home/mathew.yanez-adm@mathew.local`, so the home folder was created automatically.

**9. Check Active Directory**

On AD02, Active Directory Users and Computers now lists WEB01-MATHEW in the Computers container.

## Deliverables Captured

* Deliverable 6: SSH login as the domain user with `id`, `whoami` and `pwd`
* Deliverable 7: web01 listed in Active Directory Users and Computers

## 3 Handy Commands

```bash
realm list
```

Shows whether the machine is joined and which domain it is in.

```bash
realm discover MATHEW.LOCAL
```

Checks that Linux can find the domain through DNS before you try to join.

```bash
id user@mathew.local
```

Looks up an AD user from Linux. If it returns a UID and groups, SSSD is working.
