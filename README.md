<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Active Directory: Setup and Deployment</h1>
The goal of this tutorial is to understand the concept of deploying Active Directory within Azure, starting from the setup and breaking down into creation of users within Powershell and group policy/managing accounts.<br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Generalized Deployment Summary</h2>

- Setup Domain controller and client within Azure
- Step 2
- Step 3
- Step 4

<h2>Lab Setup</h2>

<p> <h3>Create Domain Controller</h3>
<img src="https://github.com/user-attachments/assets/1f999818-183d-48c8-9063-4a5daa3b4c06" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
  The Domain Controller is the server/computer that has an Active Directory installed on it. To create this in Azure, you will:
  1. Create Virtual Machine
  - Within the page for creating a new VM, create a Resource group and name it (ex Active-Directory-Lab)

  
  - Since this is for the Domain Controller, name the VM "dc-1"

  
  - Be sure your VM's region is the same as the Resource group is you set it up first

  
  - For the image, use "Windows Server 2022 Datacenter" and selected a size of at least 2 vcpus to ensure the VM will run smoothly

  - Set username and password to something easily remembered

  - If you have not already, under networking create a virtual network named "AD-Vnet". You may leave everything else as is.

  - Review and Create the VM

After creating the VM we're going to set the Domain Controllers NIC Private IP address to be static. Static means that the IP address will not change. To do this:

- Open dc-1

- Click Netowrking

- Click on the Network Interface Card

- Select IP configurations

- Select ipconfig1

- Change the assignment from Dynamic to Static

- Click Save 
</p>

<p> <h3>Create Client Virtual Machine</h3>
<img src="https://github.com/user-attachments/assets/9cba4f60-d252-41b4-b92e-a706ce8e9de1" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
  With this VM, we'll be able to gain practice in joining clients to the domain and managing user accounts. To set it up, we'll follow the same steps as creating the Domain Controller but this time:
 
  
  - Name the VM "client-1"

  - For the image, select "Windows 10 Pro"

We'll client-1's DNS settings to DC-1's Private IP settings 

Everything else will be the same as the intial Domain Controller setup.  
</p>
  
<br />

<p> <h3>Testing Connectivity</h3>
<img src="https://github.com/user-attachments/assets/5890a809-7eab-43e6-9edc-c896b0eb3535" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
  Now we are going to test connectivity between the two VM's, but first we're going to disable the Windows Firewall on the Domain Controller VM. To do this:
- Login in to DM using the VM's IP address on the Remote Desktop application (a warning message will pop up in your attempt to sign in, always click "yes")

- If Server Manager appears upon opening the VM, you correctly set up the Domain Controller VM. If not, try searching "Server Manager" in the search bar next to the Windows icon (start menu). If it still is not coming up, you may have selected the wrong image of has some other type of error during set up.

- Right click the start menu

- Search wf.msc (Windows Firewall Microsoft Management Conesole)

- Select Properties

- Turn off Firewall

Now we must set Client-1's DNS settings to DC-1's Private IP settings. Do this by:

- Copying dc-1's private IP address in Azure

- Go to client-1 and under Network Settings select the NIC

- On the left, click DNS Servers

- Select Custom and paste the private IP from DC-1

When the computer needs to look up anything; like google.com; it will look to dc-1 for it. Before we can open client-1:

- Go the Virtual Machines on Azure

- Select the box next to client-1 and restart it

- Once that's complete, log in to client-1 using the Remote Desktop application

- Copy the Private IP address from dc-1

- Open Powershell by the start button

- Type ping and paste dc-1's private IP address

- When it's confirmed to be working, run ipconfig /all

- From here the Output for DNS settings should match dc-1's private IP address 
</p>
<br />
<h2>Installation and Setup: Active Directory</h2>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
  To setup up and configure Active Directory:
  - Go to open dc-1 in the Remote Desktop application and go to Server Manager 

  - Click Add Roles and Features (within this there should only be 1 server delection aka dc-1)

  - Add Active Directory Domain Services

  - Click next

  - Check the "restart if required" box

  - Next through and install

  - Once you complete those steps click the flag in the top right corner of the Server Manager

  - Click "promote server to domain controller"

  - Add a new Forest

  - Input "mydomain.com"

  - Set your password

  - Be sure to uncheck the "create DNS Delegations" box

From here you will restart and log back into the dc-1 server, but you must log in as a domain user this time. To do so you will go to the Remote Desktop application as you usually would and for the username input "mydomain.com\labuser". The labuser part is whatever username you created, but the mydomain.com part MUST be there. From here put in your password and dc-1 will reopen. 
</p>
<h3>Create Domain Admin User</h3>
<p>
Now, within Active Directory we are going to create two Organizational Units (OU) called "_ADMINS" and "_EMPLOYEES". When creating theses OU's, we must ensure that they are typed EXACTLY as they are above. To create these:
- At the start, search Active Directory Users and Computers and open it

- Right click mydomain.com--> click "new"--> and select Organizational Units

- For the first one type "_EMPLOYEES" and make another named "_ADMINS"

<h3>Create New Employee</h3>

After completing the previous steps, we are now able to create our employee. 

- Right click the admin folder

- Click "new"--> select "user"--> and input the information for your new employee (ex. user: jane_doe password: Cyberlab123)

- For our convenience, be sure to uncheck check "user must change password at next log on" and check "password never expires"

- After checking these you'll click finish

Now we will add our employee to the "Domain Admins" security group. To do this: 

- Right click the employees account (jane)

- Select properties--> member of--> click add--> and enter in Domain Admins

- Select "check names" and select ok twice, then apply

Now Jane is a domain admin and we can log in as her.
<h3>Joining Client-1 to Domain</h3>

- Open client-1 and right click the start menu

- Select "system"--> "rename this PC advanced"--> "change"--> "domain"--> and type in "mydomain.com"

- You'll input the employee log in from earlier to fulfill this next step (ex. mydomain.com\jane_admin)

- From here, click restart and yes

 The last thing we will do is confirm whether client-1 is in the domain by:

 - Going to dc-1

 - Select the start menu and search "Active Directory Users and Computers"

 - Select "mydomain.com"

 - Go to computers and you should see the employee under that file 
</p>

<br />

<h2>     </h2>
<p>



  
</p>















<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1
- Step 2
- Step 3
- Step 4
