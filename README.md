<p align="center">
  <img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/95a06a12-1621-411c-b799-60cdbf0b5250" />
</p>
<div align="center">
  <h1>Configuring Active Directory (On-Premises) Within Azure</h1>
</div>


This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.

<u>**Environments and Technologies Used**</u>

* Microsoft Azure (Virtual Machines/Compute)
* Remote Desktop
* Active Directory Domain Services
* PowerShell

<u>**Operating Systems Used**</u>

* Windows Server 2022
* Windows 10

<u>**High-Level Deployment and Configuration Steps**</u>

* Create Resources
* Ensure Connectivity between the client and Domain Controller
* Install Active Directory
* Create an Admin and Normal User Account in AD
* Join Client-1 to your domain (myadproject.com)
* Setup Remote Desktop for non-administrative users on Client-1
* Create additional users and attempt to log into client-1 with one of the users


<u>**Deployment and Configuration Steps**</u>


<p align="center"><b>Setup Resources in Azure</b></p>

Create the Domain Controller VM (Windows Server 2022) named “DC-1”:
<img width="2876" height="1564" alt="image" src="https://github.com/user-attachments/assets/24dc7d0a-798b-4d12-9822-5c6552db32fe" />
<img width="2880" height="1562" alt="image" src="https://github.com/user-attachments/assets/dabf4c39-af9a-4290-b1c4-807a3e0a6748" />



Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was created in previous step:

<img width="2880" height="1564" alt="image" src="https://github.com/user-attachments/assets/49254693-d7f6-4b2c-b9d4-ebaf2b8bfaf2" />

Set Domain Controller’s NIC Private IP address to be static:

<img width="2852" height="1564" alt="image" src="https://github.com/user-attachments/assets/00b880c6-efe5-4d9d-9a78-1bb75f8e0ca3" />

Ensure that both VMs are in the same Vnet (you can check the topology with Network Watcher):

<img width="2854" height="1556" alt="image" src="https://github.com/user-attachments/assets/bc505c42-e483-431f-81f0-bdbac085769c" />



Ensure Connectivity between the client and Domain Controller

Login to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping -t (perpetual ping):

<img width="1716" height="734" alt="image" src="https://github.com/user-attachments/assets/99ac2900-2bce-4770-9bbd-59c25c66ae7f" />


Login to the Domain Controller and enable ICMPv4 in on the local windows firewall:

<img width="2880" height="1568" alt="image" src="https://github.com/user-attachments/assets/e8d0f814-e274-4fa9-a1b0-81a0ae7f74a5" />

Check back at Client-1 to see the ping succeed:

<img width="1720" height="738" alt="image" src="https://github.com/user-attachments/assets/d7d89628-6e8a-487c-a8e4-867965a933b8" />


<p align="center"><b>Install Active Directory</b></p>

Login to DC-1 and install Active Directory Domain Services:
<img width="2876" height="1716" alt="image" src="https://github.com/user-attachments/assets/17552eca-9b8c-4502-964a-d746faa58b84" />


Promote as a Domain Controller:

<img width="2872" height="1722" alt="image" src="https://github.com/user-attachments/assets/4a111e7c-1725-48c7-83d7-a1e0a91fec6e" />

Setup a new forest as myactivedirectory.com (can be anything, just remember what it is - I ultimately did set it up as myadproject.com which you'll see in the next pic):

<img width="2874" height="1716" alt="image" src="https://github.com/user-attachments/assets/d4605398-e6fc-4c4c-9614-2bb3f6e03a5d" />


Restart and then log back into DC-1 as user: myadproject.com\labuser:



<div align="center"><b>Create an Admin and Normal User Account in AD</b></div>

In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES” and another one called "_ADMINS":

<img width="2880" height="1720" alt="image" src="https://github.com/user-attachments/assets/5152e451-b774-40e2-8cdc-b3cb0fcd2c1b" />
<img width="2866" height="1720" alt="image" src="https://github.com/user-attachments/assets/6bc0ef80-e65f-4d38-a100-e5f43243b663" />

Create a new employee named “Jane Doe” with the username of “jane_admin”:

<img width="1530" height="1062" alt="image" src="https://github.com/user-attachments/assets/6443b955-1e88-4b1e-bbfb-b25ffec74a2c" />


Add jane_admin to the “Domain Admins” Security Group:

<img width="2142" height="1378" alt="image" src="https://github.com/user-attachments/assets/48f788f1-3a2f-4aaa-bb62-7057c441ed97" />

Log out/close the Remote Desktop connection to DC-1 and log back in as “myadproject.com\jane_admin”. Use jane_admin as your admin account from now on:

admin login



<p align="center">
  <b>Join Client-1 to your domain (myadproject.com)</b>
</p>

From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address:

<img width="2876" height="1564" alt="image" src="https://github.com/user-attachments/assets/33858d29-bbe6-4d08-ae1d-d3dfbafe1dcc" />

From the Azure Portal, restart Client-1.

Login to Client-1 (Remote Desktop) as the original local admin (labuser) and join it to the domain (computer will restart):

<img width="2874" height="1714" alt="image" src="https://github.com/user-attachments/assets/6bbb04ae-6904-4743-8b72-ad58dcbb5860" />

Login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the “Computers” container on the root of the domain.

Create a new OU named “_CLIENTS” and drag Client-1 into there:

<img width="2840" height="1718" alt="image" src="https://github.com/user-attachments/assets/bd9c6f99-cc0b-4bfa-8b91-d9c69bc11e06" />



<p align="center"><b>Setup Remote Desktop for non-administrative users on Client-1</b></p>

Log into Client-1 as mydomain.com\jane_admin and open system properties.

Click “Remote Desktop”.

Allow “domain users” access to remote desktop.

You can now log into Client-1 as a normal, non-administrative user now.

Normally you’d want to do this with Group Policy that allows you to change MANY systems at once (maybe a future lab):

<img width="2878" height="1716" alt="image" src="https://github.com/user-attachments/assets/618c2698-7ae7-43e8-8bea-2f8af1d6c301" />



Create a bunch of additional users and attempt to log into client-1 with one of the users

Login to DC-1 as jane_admin

Open PowerShell_ise as an administrator.

Create a new File and paste the contents of this script (https://github.com/Xinloiazn/configure-ad/blob/main/adscript.ps1) into it:

<img width="2582" height="1502" alt="image" src="https://github.com/user-attachments/assets/d87bce56-9d93-4265-8014-f68e30101b0f" />

Run the script and observe the accounts being created:

<img width="2878" height="1712" alt="image" src="https://github.com/user-attachments/assets/612f22cb-73bf-4e14-a65a-cdd8760fa9b6" />

When finished, open ADUC and observe the accounts in the appropriate OU and attempt to log into Client-1 with one of the accounts (take note of the password in the script):

<img width="2840" height="1720" alt="image" src="https://github.com/user-attachments/assets/804e9c43-4677-4576-95bf-31e21da14b5e" />
<img width="2846" height="1800" alt="image" src="https://github.com/user-attachments/assets/98c27854-5e85-42a1-8f3d-8d3ef98ed67c" />



I hope this tutorial helped you learn a little bit about network security protocols and observe traffic between virtual machines. This can be easily done on a PC or a Mac. Mac would just have an extra step to download the Remote Desktop App.

Now that we're done, DON'T FORGET TO CLEAN UP YOUR AZURE ENVIRONMENT so that you don't incur unnecessary charges.

Close your Remote Desktop connection, delete the Resource Group(s) created at the beginning of this tutorial, and verify Resource Group deletion.
