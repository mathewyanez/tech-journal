# ADDS Lab

## Objectives

* Create an organizational unit (OU) in our domain.
* Create a group policy that enforces various options.
* Apply settings to the groups and computers in the newly created OU.

## Pre-requisites

Lab 4 is complete and is in a happy place.

{% hint style="warning" %}
Watch this lab’s _firm_ _Due Date_. As mentioned in class, the current VM infrastructure will be deleted and replaced with newer VMs just prior to the next class for the Assessment. Additionally, since this is a lighter lab, there are no extensions.
{% endhint %}

## OU Structure Creation

Open up Active Directory Users and Computers.

The first thing we want to do is create an organizational unit called "SYS255." Within this OU, add child OUs for Accounts, Computers, and Groups.

{% hint style="info" %}
Although the default installation of ADDS provides a structure for Users and Computers, we are adding our own to distinguish the objects we add from those that are included by default.
{% endhint %}

Here's the completed structure shown below:

{% hint style="info" %}
Notice that now this is created, we can right-click and create users, groups, and other domain objects in Active Directory. All of these objects are defined by what's known as the [Schema](https://learn.microsoft.com/en-us/windows/win32/ad/active-directory-schema), which can be thought of as an instruction sheet/map listing all available pieces in AD. In this case, the schema objects make up a distributed database.
{% endhint %}

## Create Users and Groups

Within the `SYS255\Accounts` OU, create users Alice, Bob, and Charlie.

{% hint style="info" %}
When creating accounts for other users, it is wise to allow them to create a new password at first login. For purposes of the lab, you can clear this check mark.
{% endhint %}

Drag WKS01 from the `yourname.local\Computers` folder to the `SYS255\Computers` OU. This will allow us to treat SYS255 OU Computers differently than others.

Within the `SYS255\Groups` OU, add a global security group called _custom-desktop_ with users Alice and Bob, but not Charlie, as members.

{% hint style="info" %}
**BEST PRACTICE FOR GROUPS:** Many times, organizations will have a number of groups defined in their AD domain. For this reason, it is a best practice to have a naming convention that purposefully describes what the groups do. A lot of times, groups allow or disallow users permission to folders and resources on the network. For this reason, a commonly found group membership is in the form of something like this: `DepartmentName_RW_ACL` or `GP_WindowsIESettings_ACL`. This gives administrators an idea of what the group is for, and who may need to be a member.
{% endhint %}

## Group Policy - User

Now, let’s create a group policy that defines some User level settings.

The following screenshot illustrates the relationship between the OUs created in Active Directory and the Policy hierarchy shown in the Group Policy Management window. The big takeaway is that the group policy window does not show the contents of an OU like accounts and computers, but allows you to apply policy to them.

Notice how there is already a Default Domain Policy. This is what controls that pesky default password expiration and complexity requirements.

{% hint style="warning" %}
Weak Administrator credentials are the root cause for many security breaches! While the default password complexity rules are good, one should only increase security of credentials.
{% endhint %}

## Creating a User Policy

Select the SYS255 OU and create a new group policy object (GPO) called `sys255-desktop`. Once created, right-click on the object and select **Edit**.

Now, this SYS255-desktop Group Policy should only apply to those users in this OU who are members of the custom-desktop security group. You set this using the security filters section of the group policy. By default, All Authenticated Users have access to apply and read group policy; we will restrict this through the following steps.

{% stepper %}
{% step %}
### Add the custom-desktop group

Add the custom-desktop group created earlier to the Security Filter.
{% endstep %}

{% step %}
### Remove Authenticated Users

Remove Authenticated Users from the Security Filter.

{% hint style="info" %}
For more information on this error message which was a source of consternation among Windows Admins back in 2016, see [https://go.microsoft.com/fwlink?linkid=843010](https://go.microsoft.com/fwlink?linkid=843010)
{% endhint %}
{% endstep %}

{% step %}
### Add Domain Computers

Add Domain Computers.
{% endstep %}

{% step %}
### Configure delegation

Go to the **Delegation** tab, then **Advanced**. Uncheck **Apply Group Policy** and select **Deny**.
{% endstep %}
{% endstepper %}

Once we have defined who this policy applies to, we are now ready to author what the group policy does.

{% hint style="info" %}
This is the bulk of the group policy editor on a Windows server where we can define computer and user settings. Remember: **COMPUTER** settings are applied when workstations turn on, whereas **USER** settings apply after users log in. There are a handful of settings here that we can define and really control the experience of the workstation in this domain. This is commonly used to control things such as, but not limited to: desktop backgrounds, browser settings, password policies, network shares, printers, redirected folders, Microsoft Bitlocker, application allowed list policies, logon scripts, etc.
{% endhint %}

## Nuking the Recycle Bin

Your users are revolting against the Recycle Bin, so let’s remove it.

Find the **Remove Recycle Bin icon** setting under User Configuration, and click **Edit Policy Setting** in the group policy editor.

Enable the **Remove Recycle Bin Icon from Desktop** setting.

> Note: Frequently in AD GPO settings, its wording can be tricky, where you enable/allow the removal of a feature/function.

Click **Apply**, **OK**, and close the Group Policy editor.

### Deliverable 1

Log in to WKS01 as Alice. Your desktop should not include the Recycle Bin. Provide a screenshot showing both your VM name, the lack of Recycle Bin, and the results of `gpresult /r` using Alice's account.

## Creating a Computer Policy

{% hint style="info" %}
Unlike User policies that are associated with the logged-on user, Computer policies are applied before login and affect the entire system and thus any logged-in users.
{% endhint %}

## Disable Last Login

{% hint style="info" %}
The default display of previously logged-on users is widely considered a security vulnerability, particularly in shared systems. The next policy will turn off the default display of previously logged-in users.
{% endhint %}

Create and link a new GPO within the `SYS255\Computers` OU called _DisableLastLogin_.

The Security Filter on this policy should be applied to Domain Computers, not Authenticated Users, similar to earlier. Then edit the policy so that **Do not display last user name** is enabled.

### Deliverable 2

On WKS01, from an elevated domain administrative command prompt, issue the following commands:

```
gpupdate /force
```

```
gpresult /scope computer /r
```

Provide a screenshot showing the DisableLastLogin Policy was applied.

{% hint style="info" %}
Even though you may be logged into WKS01 as the `-adm` AD power account, you still need to elevate your command prompt or PowerShell session to “Run as Administrator.” Try right-clicking over the shortcut for Command Prompt or PowerShell.
{% endhint %}

### Deliverable 3

Sign out of WKS01, and provide a screenshot showing the changes to the login screen. You should no longer see evidence of the last user who had logged in.

### Deliverable 4

For your Tech Journal Entry, create a detailed plan of how to prepare for next week’s assessment. This plan should include a Current Network Diagram using an example tool such as https://app.diagrams.net containing at least devices, hostnames, IPs, services, and “cabling”.
