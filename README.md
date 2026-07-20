<img width="300" height="168" alt="image" src="https://github.com/user-attachments/assets/95a06a12-1621-411c-b799-60cdbf0b5250" />

Configuring Active Directory (On-Premises) Within Azure
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.
Environments and Technologies Used
Microsoft Azure (Virtual Machines/Compute)
Remote Desktop
Active Directory Domain Services
PowerShell
Operating Systems Used
Windows Server 2022
Windows 10
High-Level Deployment and Configuration Steps
Create Resources
Ensure Connectivity between the client and Domain Controller
Install Active Directory
Create an Admin and Normal User Account in AD
Join Client-1 to your domain (myadproject.com)
Setup Remote Desktop for non-administrative users on Client-1
Create additional users and attempt to log into client-1 with one of the users
Deployment and Configuration Steps


Setup Resources in Azure

Create the Domain Controller VM (Windows Server 2022) named “DC-1”:

resource group vm ms server

Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was created in previous step:

vm windows

Set Domain Controller’s NIC Private IP address to be static:

static ip

Ensure that both VMs are in the same Vnet (you can check the topology with Network Watcher):

topology



Ensure Connectivity between the client and Domain Controller

Login to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping -t (perpetual ping):

perpetual ping

Login to the Domain Controller and enable ICMPv4 in on the local windows firewall:

enable ICMPv4

Check back at Client-1 to see the ping succeed:

ping success



Install Active Directory

Login to DC-1 and install Active Directory Domain Services:

active directory install

Promote as a Domain Controller:

domain controller promotion

Setup a new forest as myactivedirectory.com (can be anything, just remember what it is - I ultimately did set it up as myadproject.com which you'll see in the next pic):

set new forest

Restart and then log back into DC-1 as user: myadproject.com\labuser:

fqdn login



Create an Admin and Normal User Account in AD

In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES” and another one called "_ADMINS":

organizational unit organizational unit

Create a new employee named “Jane Doe” with the username of “jane_admin”:

admin creation

Add jane_admin to the “Domain Admins” Security Group:

security group

Log out/close the Remote Desktop connection to DC-1 and log back in as “myadproject.com\jane_admin”. Use jane_admin as your admin account from now on:

admin login



Join Client-1 to your domain (myadproject.com)

From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address:

client dns settings

From the Azure Portal, restart Client-1.

Login to Client-1 (Remote Desktop) as the original local admin (labuser) and join it to the domain (computer will restart):

domain joining

Login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the “Computers” container on the root of the domain.

Create a new OU named “_CLIENTS” and drag Client-1 into there:

active directory client verification



Setup Remote Desktop for non-administrative users on Client-1

Log into Client-1 as mydomain.com\jane_admin and open system properties.

Click “Remote Desktop”.

Allow “domain users” access to remote desktop.

You can now log into Client-1 as a normal, non-administrative user now.

Normally you’d want to do this with Group Policy that allows you to change MANY systems at once (maybe a future lab):

remote desktop setup



Create a bunch of additional users and attempt to log into client-1 with one of the users

Login to DC-1 as jane_admin

Open PowerShell_ise as an administrator.

Create a new File and paste the contents of this script (https://github.com/Xinloiazn/configure-ad/blob/main/adscript.ps1) into it:

create users script

Run the script and observe the accounts being created:

observe create users script

When finished, open ADUC and observe the accounts in the appropriate OU and attempt to log into Client-1 with one of the accounts (take note of the password in the script):

employee user accounts employee user selection employee user login



I hope this tutorial helped you learn a little bit about network security protocols and observe traffic between virtual machines. This can be easily done on a PC or a Mac. Mac would just have an extra step to download the Remote Desktop App.

Now that we're done, DON'T FORGET TO CLEAN UP YOUR AZURE ENVIRONMENT so that you don't incur unnecessary charges.

Close your Remote Desktop connection, delete the Resource Group(s) created at the beginning of this tutorial, and verify Resource Group deletion.
