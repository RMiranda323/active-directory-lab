# active-directory-lab
Practice using active directory. 

<h2>Set up</h2>
<b>Needed Software</b>

- VirtualBox
- Windows 10 ISO
- Windows Azure Server ISO

<h2>Lab Steps</h2>

I'm first going to create the Domain Controller (DC) for the lab. The DC virtual machine will be created using the Windows Server ISO.
<img src = "https://i.imgur.com/uhB4dNc.png">
<br>The only setting that must be configured is to add an internal network adapter 

Setting > Network > Adapter 2 > Enable Adapter > Attached to: Internal Network
<img src = "https://i.imgur.com/sJfBrnj.png" >


In the DC machine we need to configure our IP so we can eventually get a network setup that looks something like this
<img src = "https://i.imgur.com/5ZQhG5z.png">

I'm going to rename the PC just to easily tell the two apart 
System > Rename this PC 
<br><img src = "https://i.imgur.com/DTiST3S.png">
<br>I named it DC for domain controller 

I also renamed the network adapters to easily distinguish internal and external (right click on adapter and "rename")
<br><img src = "https://i.imgur.com/E0QenFe.png">

Lastly we want to change the IP for the Internal network
Right click Internal adapter > Properties > Internet Control Protocol Version 4
Configure the IP to look like our diagram 

<img src = "https://i.imgur.com/TSu7LnU.png" >

Note: DNS can be set to our own IP or 127.0.0.1 which is the loopback address

Next I need to install Active Directory Domain Services (ADDS)
Server Manager > Dashboard > Add Roles and Features > Sever Roles > Active Directory Domain Services

<img src = "https://i.imgur.com/BtFMKYb.png">

Now on the dash there's a notification that is to promote the server to DC
<br>The only setting we want to change is to create a forest and the domain name 

<img src = "https://i.imgur.com/zhNoDVx.png">

Now we're going to create our admin account
Start > Admin tools > Active directory users and computers

Right click on mydomain.com and create a new Organizational Unit (OU) to save our account in 

And finally right click the OU to add a new user

<img src = "https://i.imgur.com/uMiMZhj.png">

To make the user an admin just right click > properties > member of > Add...

Then add the object Domain Admins and apply it to the user 

<img src = "https://i.imgur.com/Osdzoyu.png">
<br>Going forward I'll be logging into the admin account instead of the default

The next step is to install RAS/NAT
<br> Same process as adding ADDS but installing Remote Access instead
<br> <img src = "https://i.imgur.com/Qe6Jxnz.png">
<br> and add Routing in services 
<br> <img src = "https://i.imgur.com/XBeACC7.png">

Once installation is complete we need to configure it by going to tools in the server manager > Routing and remote access management
<br> then right click DC (local) and Configure and Enable Routing and Remote Access
<br> We want to configure NAT and select our external netowrk 
<br><img src="https://i.imgur.com/wWSDZLt.png"> 
<br><img src = "https://i.imgur.com/PgTbPbN.png">

Now we're going to set up a DHCP server on the DC to allow the internal network devices access to the internet
<br> Sever manager > add roles and features > server roles > DHCP server
<br> Now back to tool > DHCP to configure the scope 

Right click > New scope 
<br> Our range is going to be 172.16.0.100-200 so I'm going to name it that and set the start and end 
<br> <img src = "https://i.imgur.com/RYC11RV.png">
<br> Set the length to 24 and the subnet mask will be set automatically
<br> I'm not gonna change any exclusions or lease delays, but when configuring in a non-lab environment these are really useful

Now to configure the DHCP we need to add the IP of what we want our gateway to be which is the IP of the internal adapter
<br> <img src = "https://i.imgur.com/8fTD1NC.png">
<br> Add the IP using the first box then select it in the second and continue 

Next is DNS which is automatically included in the DC so just select the IP and go next 
<br> <img src = "https://i.imgur.com/53cn9nc.png">

<b>We are going to ignore WINS server and go to activate the scope

Now we just need to authorize our DHCP Server
<br> Right click > Authorize 
<br> <img src="https://i.imgur.com/ziSR3z2.png">
<br> Might have to refresh sometimes but if the arrows on ipv4/ipv6 are green then it's working 

Next we're going to change a setting in the server manager to allow users on the server access to the internet
<br>It's in Dashboard > Configure this local server > IE Enhanced Security Configuration and turn it off 
<br><img src = "https://i.imgur.com/cROzjc1.png">
<br><b>Note: Don't do this in a production environment</b>

Now I'm going to create a lot of users to simulate a working server. I'm going to use a <a href = "https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbERxbzNEbkhIWWxfWjg4blN2ekpUcjRjcDVvQXxBQ3Jtc0tuX1ZoRjJqanFjOGdlX2t4LUFnYmY3SHo4MEVadkxMS184TUd1WGJIdVZ5XzJtNVZrMWdPcWl3NWJfNGpkdXNPbWFjaGJlY3hDcU9lT09kaDFVd002WUM3dnp1S1g1VVlpcHBLYk9GNGdVeC05WkFSOA&q=https%3A%2F%2Fgithub.com%2Fjoshmadakor1%2FAD_PS%2Farchive%2Frefs%2Fheads%2Fmaster.zip&v=MHsI8hJmggI"> script made by Josh Madakor</a> to create the users.

Once it's downloaded just open the "1_CREATE_USERS" script using Windows Powershell ISE running on admin mode
<br> Before we can run it we have to use the command Set-ExecutionPolicy Unrestricted to allow us to run the script
<br> <img src = "https://i.imgur.com/2pKBVTo.png">
<br> Note: Again do not do anything that can compromise security in a non-lab environment 

Summary of script: using a foreach loop on a list of names it will create a user account with the first and last name, user of first initial lastname and password of "Password1" for simplicity. It will also create an Organizational Unit named Users and add the new accounts into it.

To check if it worked go to AD Users and Computers to see if the OU was created and filled with users
<br> <img src = "https://i.imgur.com/6ewptFL.png">

And that'll be all for the server and user creation now to create a VM for a user device. 


