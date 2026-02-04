# Linux Cyberpatriot Notes

## 🐧 System Hardening and Security Automation Script

This repository contains a checklist for automating system hardening in Linux environments. **Start with Forensic Questions**, as deleted files can make it impossible to investigate certain issues effectively.

***

### Table of Contents

* 🔍 Preliminary Checks & Security Policies
* 🔐 Password & Authentication Policies
* 👤 System and User Management
* 🛠 Security & System Services
* 🌐 Browser Security
* ⚙️ Service Status
* 👥 Administrator and Account Management
* ✅ Final Steps

***

#### 1. 🔍 Preliminary Checks & Security Policies

* **Update and Upgrade System**:
  *   Begin by updating and upgrading to the latest package versions:

      ```plaintext
      sudo apt update && sudo apt upgrade -y
      ```

ADD GO TO ROOT FIRST

* **Locate Forbidden MP3 Files**:
  *   Locate and remove `.mp3` files:

      ```plaintext
      locate *.mp3
      ```
* **Import Information from Text File**:
  *   Display hidden files to locate text information:

      ```plaintext
      ls -a
      ```
* **Update Applications**:
  * Check the info page to confirm the absence of prohibited files.

***

#### 2. 🔐 Password & Authentication Policies

**Important:** Change all user passwords before adjusting the password policy to avoid setup errors.

* **Change User Passwords**:
  * Set compliant passwords for all users.
* **Edit Password Policy**:
  *   Open `Login.defs` to set the policy:

      ```plaintext
      sudo nano /etc/Login.defs
      ```

      * Set:
        * **Max days**: 90
        * **Min days**: 10
        * **Pass warn age**: 7
* **Configure PAM for Password Requirements**:
  *   Install `libpam-cracklib` and configure:

      ```plaintext
      cd /etc/pam.d
      sudo apt install libpam-cracklib
      sudo nano common-password
      ```

      * Settings:
        * Password history of 7
        * Minimum length of 10 characters
        * Complexity enabled: one of each character type

***

#### 3. 👤 System and User Management

MAKE THIS FASTER

SWITCH TO cat /etc/group

* **List All User Accounts (except LDAP or remote)**:
  *   To list all user accounts on the system, you can use:

      ```bash
      cut -d':' -f 1 /etc/passwd
      ```
  *   Alternatively, you can use:

      ```bash
      awk -F ':' '{print $1}' /etc/passwd
      ```
*   **Remove User from Group**:

    ```plaintext
    sudo gpasswd -d username groupname
    ```
* **Daily System Updates**:
  * Open **Software & Updates** > **Updates** tab and set to check daily.
  * Enable **Important Security Updates**.
* **Disable OpenSSH Root Login**:
  *   Edit SSH configuration:

      ```plaintext
      sudo nano /etc/ssh/sshd_config
      ```

      * Set `PermitRootLogin` to "no".

***

#### 4. 🛠 Security & System Services

cat /etc/X11/default-display-manager

* **Remove Unauthorized Packages**:
  *   List all packages and remove unauthorized ones:

      ```plaintext
      sudo apt list --installed
      sudo apt purge [package]
      ```
*   **Enable and Check UFW (Firewall) Status**:

    ```plaintext
    sudo ufw enable
    sudo ufw status
    ```

    * _Note:_ Some schedulers may turn off the firewall, so ensure it’s enabled and active.
* **Check Listening Ports with Netstat**:
  * Inspect open ports and high PIDs using `netstat`.
*   **UFW Deny Unnecessary Connections**:

    ```plaintext
    sudo ufw deny [port/service]
    ```
*   **If there, and not supposed to be there: Remove FTP Service**:

    ```plaintext
    sudo apt purge vsftpd
    ```

***

#### 5. 🌐 Browser Security

* **Chrome Certificate Compliance**:
  * Ensure Chrome certificates are compliant.
* **Remove Other Browsers**:
  * Delete non-specified browsers.
* **Verify Browser is Updated**:
  * Ensure the latest version of Browser is installed for security compliance.
* **Harden Firefox Browser**:
  * Open **Settings > Privacy & Security** in Firefox.
  * Enable:
    * Block dangerous and deceptive content
    * All security sub-options

***

#### 6. ⚙️ Service Status

* **List and Filter Services**:
  *   Use these commands to check for active, inactive, and non-compliant services:

      ```plaintext
      service --status-all             # Lists all services
      service --status-all | grep '\[ + \]'  # Filters running services
      service --status-all | grep '\[ - \]'  # Filters non-running services
      ```
* **Non-compliant Services**:
  * Remove or disable the following if present:
    * `nginx`
    * `apache2`

***

#### 7. 👥 Administrator and Account Management

* **Verify Correct Administrators**:
  * Review and confirm authorized administrator accounts.

***

#### 8. ✅ Final Steps

* **Administrator Correction and Account Deletion**:
  * Ensure all unauthorized or unused accounts are removed as a final step.

***

#### Notes

For further guidance, refer to official system documentation to align with best security practices.

* **Previous Competition Notes (Cybercomp 1)**:
  * Ensure accounts are:
    * Password protected
    * Using secure, compliant passwords
    * Set for password expiration at 90 days
  * Remove any browser other than **Google Chrome**.
  * Remove Civ
