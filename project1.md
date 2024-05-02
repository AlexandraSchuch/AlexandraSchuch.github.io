# Active Directory Project

## Objective

The Active Directory Lab project creates a controlled environment for simulating and detecting cyber attacks. The lab focuses on ingesting and analyzing logs within a Security Information and Event Management (SIEM) system, generating real-time traffic and test telemetry to mimic real-world attack scenarios. This is both a red and blue team project. This is because the project will both actively simulate a brute force attack and produce the logs to familiarize one's self with the detection and creation of alerts for these attacks on the network. This hands-on experience was designed to deepen the understanding of network security, attack patterns, and defensive strategies.

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- Splunk, as our Security Information and Event Management (SIEM) system for log ingestion and analysis. 
- Windows utility service, Sysmon, used to send both our Active Directory and Windows 10 system log data over to Splunk 
- Telemetry generation tool, Atomic Red Team, to create realistic network traffic and attack scenarios.
- Brute-forcing tool, Crowbar, to simulate a brute force attack.

### Network Diagram

We will refer back to this diagram at any time during the project to avoid any confusion and keep track of all devices being used within the project and their respective IPs. 

![[Active-Directory-Diagram7.png]]

### PRE-Project Checklist

- [ ] Download all required VMs- Windows 10 (Target-PC), Windows Server (Active Directory), Kali Linux (Attacker), Ubuntu Server (Splunk)
- [ ] Change all VM network's to NAT Network that you create to have all devices on same network with internet.
- [ ] Make network diagram

#### Setting up static IP address for Splunk Server
***
As you can see our current IP address is `192.168.10.4` and we need to change that to match our diagram address of `192.168.10.10`

![[Pasted image 20240420023738.png]]

We need to go into the Network Config and change some things to make the IP address static so we run the command
```Splunk
sudo nano /etc/netplan/00-installer-config.yaml
```

![[Pasted image 20240420024317.png]]

We are then present with this 
![[Pasted image 20240417204446.png]]

We edit the file to look like this and we save the changes. 
```yaml
network: 
  ethetnets:
   enp0s3:
     dhcp4: no
     addresses: [192.168.10.10/24]
     nameservers:
        addresses: [8.8.8.8]
    routes:
       - to: default
        via: 192.168.10.1
version: 2
```
![[Pasted image 20240420024606.png]]

We apply the new Netplan so we have our static IP by using this command. We ignore the warning. It doesn't effect this. 

```
sudo netplan apply
```

![[Pasted image 20240417205042.png]]
To check that the IP addresses has now changed to the static addresses we changed it to, we run this command.
```
ip a
```
we see that we were successful at changing the IP address because now it shows `192.168.10.10/24`

![[Pasted image 20240417205134.png]]

We ping google.com to make sure we have a connection by doing the command
```
ping google.com
```
![[Pasted image 20240417205248.png]]

### Install Splunk on HOST machine.

We want to install `Splunk` on our HOST computer, so we will head over to `Splunk.com` on our browser through our HOST computer.

![[Pasted image 20240501203059.png]]

If you DO NOT have an account already then you want to click `Sign up`, otherwise `Log in`.
![[Pasted image 20240501203133.png]]

Once you're signed in, we will hover over `Products` and then click on `Free Trials & Downloads`

![[Pasted image 20240501203314.png]]

Scroll down to `Splunk Enterprise` and click `Get My Free Trial`

![[Pasted image 20240501203433.png]]

We want to make sure that we select `Linux` as our OS because our Splunk Server is a Ubuntu Server.

![[Pasted image 20240501204255.png]]

We are interested in the `.deb` file. We will select `Download Now`.

![[Pasted image 20240501204356.png]]

We will save this in a directory of our choice. 

![[Pasted image 20240501204501.png]]

### Download guest add ons for VirtualBox
***

We want to download the `guest add ons` for VirtualBox
```
sudo apt-get install virtualbox-guest-additions-iso
```

![[Pasted image 20240420030616.png]]

We will press `y` and `enter`. Then wait for everything to download.

![[Pasted image 20240420030925.png]]

Once we see this screen, we press `enter` and our package should install.

![[Pasted image 20240420031144.png]]

Now we want to click `Devices` on our VM at the top 
![[Pasted image 20240420031415.png]]

We then want to go to `Shared Folders` and then to `Shared Folders Settings`
![[Pasted image 20240420031556.png]]

We are presented with this.
![[Pasted image 20240420031705.png]]

We want to add a new folder by clicking on this icon on the right hand side.
![[Pasted image 20240420031748.png]]

For folder path, we want to use the same folder where we saved the `Splunk .Deb package` for me it was in an `ActiveDirectoryProject` folder. We will also checkmark `Read-only`, `Auto-mount`, and `Make Permanent`. Then press `OK`  
![[Pasted image 20240420032120.png]]

We then will reboot the VM running the command
```
Sudo reboot
```
![[Pasted image 20240420032337.png]]

We relog back in.

Then we want to add our user to vboxsf. In my case it's `chum_bucket` by running the command
```
sudo adduser chum_bucket vboxsf
```

![[Pasted image 20240417211043.png]]
If we get this 
![[Pasted image 20240420033006.png]]
then we might need some additional guest installations for VirtualBox so we will get them by running the command 
```
sudo apt-get install virtualbox-guest-utils
```

![[Pasted image 20240420033336.png]]
We will reboot again and relog in. 
```
sudo reboot
```

We will try again to add the user to the vboxsf and we see that it works.

![[Pasted image 20240417211744.png]]

We will make a new `directory` called `share` by running the command
```
mkdir share
```

We will check to make sure that its there by running the `ls` command
![[Pasted image 20240417211817.png]]

Now that the `Share` directory has been made, we want to run the following command to mount our shared folder onto our share directory. 

NOTE: if we forget where the folder path is, then we can go back to the top and click `Devices` -> `Shared Folders` -> `Shared Folders Settings` -> `Shared Folders` and look at `Folder Name`

```
sudo mount -t vboxsf -o uid=1000,gid=1000 ActiveDirectoryProject share/
```
![[Pasted image 20240417212250.png]]
NOTE2: if we get an error when entering this command then exit and relog back in because when you add a user into a new group, it doesn't take effect until you log out. 

We will run the command 
```
ls -la
```
and we see that `share` is now highlighted
![[Pasted image 20240420035421.png]]
we will change directories into `share` by running the command 
```
cd share
```
![[Pasted image 20240420035047.png]]
we will run the command again
```
ls -la 
```
![[Pasted image 20240420035331.png]]

We will then install `Splunk` by running the command
```
sudo dpkg -i (splunk-9.2.1-.deb) 
```

![[Pasted image 20240417212653.png]]

Now we will change into the directory where `Splunk` is located on our server by running the command

```
cd /opt/splunk
```
and run the command to see all 
```
ls -la
```
![[Pasted image 20240417212743.png]]
and we notice that all the user and group belong to `splunk` which is good because it limits the permission to that user or group.

We will change to the user `splunk` by running the command
```
sudo -u splunk bash
```
Now we are acting as the user `splunk`. We will change the directory to `bin` by running the command 
```
cd bin
```
NOTE: All the files in here are listed as the binaries that splunk can use.

![[Pasted image 20240417212824.png]]
We will run this command to run `splunk`
```
./splunk start
```
![[Pasted image 20240417212906.png]]
We will get a license and terms agreement. We will press `q` then `y` then `enter`. We then enter an admin name and password. 

![[Pasted image 20240420041107.png]]

Once installation is completed we see this.
![[Pasted image 20240420041631.png]]

We will run one more command to make sure that `splunk` starts up every time our VM reboots.

We will run the command
```
exit
```
to get out of the user `splunk` 

and run this command to change to the `bin` directory
```
cd bin
```

and then run the command 
```
sudo ./splunk enable boot-start -user splunk
```
This will make it so that anytime the VM reboots, `splunk` will run with the user `splunk`
![[Pasted image 20240417213321.png]]

#### Installing Splunk Universal Forwarder and Sysmon on Windows 10 AKA "Target PC"
***
On Windows 10 machine, we search up `pc` and go into `Properties` -> `Rename this PC`. 

We rename it to `target-PC` so that it makes things more simple to understand in the logs. We then reboot and check in the `pc` -> `Properties` again to verify the name has changed.
![[Pasted image 20240417213836.png]]

Next, we want to search for `cmd` and see our current IP address by running the command

```Command Prompt
ipconfig
```
![[Pasted image 20240420142237.png]]

Right now the Windows 10 machine is getting assigned a dynamic IPv4 address through DHCP and we need to change it to the static address `192.168.10.100` to match our network diagram. 

So, to do that we will right click the `network`![[Pasted image 20240420142636.png]] icon in the bottom right. Then click `Open Network & Internet settings` -> `Change adapter options`. Right click `Ethernet` -> `Properties` -> `Internet Protocol Version 4 (TCP/IPv4)` 

![[Pasted image 20240420142615.png]]
![[Pasted image 20240420142839.png]]
![[Pasted image 20240420143105.png]]

We now will set our static IP with the configurations below
![[Pasted image 20240417214706.png]]

Then we run the `ipconfig` command again to verify that the IP address has changed to `192.168.10.100` and we see that it has.

![[Pasted image 20240417215147.png]]

We will now open up a web browser and try accessing our `splunk` server by typing in its IP address that we statically assigned, followed by `:8000` as so, `192.168.10.10:8000` into the address bar. 

NOTE: `Splunk` listens on port 8000 and that is why we specify it. Also make sure your `Splunk` server is powered on or else it won't work.

![[Pasted image 20240420143848.png]]

We see this screen and know that we can access `splunk`
![[Pasted image 20240420144439.png]]

Now we will install `Splunk Universal Forwarder` by heading to the `splunk.com` site and logging into our account. 
![[Pasted image 20240420144735.png]]

Once signed in, we click `Products` -> `Free Trials & Downloads` and scroll down till we see `Universal Forwarder`. This is what we will use to send data to our `Splunk` server from our `Target PC` aka. windows 10 machine. 
![[Pasted image 20240420145330.png]]

Click `Get My Free Download` and then on the OS that we are running, which will be the `Windows 10 - 64-bit` download.
![[Pasted image 20240420150328.png]]

Once the download is complete, we will open it up through the `Downloads` folder in `File Explorer`.

![[Pasted image 20240420150635.png]]

We will click the `Check this box to accept the License Agreement`. Make sure we have the `An on-premises Splunk Enterprise instance` picked and click `Next`.

![[Pasted image 20240420150850.png]]

We will put `admin` as `Username:` and leave the `Generate random password` checked. Then click `Next`. 

![[Pasted image 20240420151025.png]]

We do not have a deployment server so we will skip this and press `Next`.

![[Pasted image 20240420151140.png]]

The `Receiving Indexer` is going to be our `Splunk` server so we will put it's IP and put the `port`  as the default `9997` for receiving events.

![[Pasted image 20240420151320.png]]

We click `Install`. 

![[Pasted image 20240420151508.png]]

Now we are going to install `Sysmon` by going back to our web browser and searching `sysmon` and selecting the `sysmon` by `Sysinternals`. 

![[Pasted image 20240420151740.png]]

We scroll down and click `Download Sysmon`. 

![[Pasted image 20240420151837.png]]

Now we need to get our Sysmon configuration, which will be the "sysmon olaf config." We will search that on our browser to find the github page.

![[Pasted image 20240424141835.png]]

Then we will scroll down till we see `sysmonconfig.xml` and click on it. 

![[Pasted image 20240424141929.png]]

We will then click the ![[Pasted image 20240424142110.png]] button on the right side of the page. Right-click `save as` and then save it within the `Downloads` directory.

![[Pasted image 20240424142214.png]]

Now lets go to our `Downloads` directory. Right-click on our `sysmon` zipped folder and press `extract all`.

![[Pasted image 20240424142530.png]]

`extract`.
![[Pasted image 20240424142604.png]]

We see the `sysmon` folder now and will click on the path bar and copy the file path. 

![[Pasted image 20240424142812.png]]

We open `PowerShell` and `run as administrator`. We run the command below to change directories to our copied path. 
Note: Your pathway will be different than mine unless you also named your user, "WINDAS".

```PowerShell
cd C:\Users\WINDAS\Downloads\Sysmon
```

We will then run this command once we are in the `Sysmon` directory. 

```PowerShell
.\Sysmon64.exe -i ..\sysmonconfig.xml
```

The "-i" flag indicates that we want to specify on a configuration file. In which, we use the olaf one we just downloaded.  

The ".." indicates that we want to go back one directory since our config file was saved in the `Downloads` directory. 

![[Pasted image 20240418004033.png]]

We press enter and it begins to install. 

![[Pasted image 20240418004233.png]]

We press `Agree`

![[Pasted image 20240424144001.png]]

We also see that `Splunk Universal Forwarder` has also finished.

![[Pasted image 20240424145314.png]]


#### Instruct Splunk Universal Forwarder (on Windows 10 AKA "Target-PC") on what we want to send to our Splunk Server. 
***

We must configure a file named `inputs.conf`. Which if we open our `File Explorer` then we can locate it by going to `This PC` -> `Local Disk (C:)` -> `Program Files` -> `SplunkUniversalForwarder` -> `etc` -> `System` -> `default` and we see the file `inputs.conf`. We will leave it alone. 

![[Pasted image 20240424150705.png]]

We will go back a directory to `system` and go into `local` Here we will create a new `inputs.conf` file. 

NOTE: it is VERY important that we DO NOT edit the `inputs.conf` file within the `default` directory. That one is there just in case we mess anything up and need to replace the file with the default one. 

![[Pasted image 20240424151132.png]]

However, we notice that we can only make folders within the directory and we want to create the `inputs.conf` file. So, instead we will open up `notepad` and create the file that way and save it within the `local` directory.

![[Pasted image 20240424151543.png]]

So we open `notepad` and `run as administrator`. We copy the contents (below) of the original `inputs.conf` within the `default` directory and paste it into the notepad file.

``` Copy this
[WinEventLog://Application]

index = endpoint

disabled = false

[WinEventLog://Security]

index = endpoint

disabled = false

[WinEventLog://System]

index = endpoint

disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]

index = endpoint

disabled = false

renderXml = true

source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

This will tell our `Splunk Universal Forwarder` to push events related to `Application`, `Security`, `System`, and `Sysmon` over to our `Splunk` Server. 

NOTE: The `index = endpoint`. This means when these events occur they will be sent over to the `Splunk` Server and placed within the index `endpoint`. If our `Splunk` Server does not have an index named `endpoint`, then it will not receive any of these events. Therefore, we would have to create an index named `endpoint` to correct that. 

![[Pasted image 20240424152924.png]]

So, we save the notepad file under the `local` directory like we mentioned before, name it `inputs.conf`, change the "save as type:" to `All Files`, and hit `save`. 

![[Pasted image 20240424153759.png]]

Now we see `inputs.conf` under our `local` directory.

![[Pasted image 20240424153926.png]]

NOTE: Anytime we update the `inputs.conf` file we will need to restart `Splunk Universal Forwarder` service. 

So, we will search up `Services` and `Run as administrator` 

![[Pasted image 20240424154153.png]]

Click on any of the services once and type `s` to go to the services that start with "s." 

![[Pasted image 20240424154334.png]]

We will look for the `SplunkForwarder` service.

![[Pasted image 20240424154426.png]]

If we scroll to the right of `SplunkForwarder` and look under the column `Log On As`, we might notice it says `NT SERVICE`. We DO NOT want this. 

![[Pasted image 20240424154739.png]]

We will double click the service to view its `properties`.

![[Pasted image 20240424154912.png]]

We will go to `Log On`. 

![[Pasted image 20240424155010.png]]

If we see `NT SERVICE` for "This account:" then it might not be able to collect logs due to permissions so instead we will click `Local System account` and hit `Apply`. 

![[Pasted image 20240424155310.png]]

We see this message. This is fine since we planned on restarting the service anyways because we updated our `inputs.conf` file. 

![[Pasted image 20240424155359.png]]

Notice that it now says that `SplunkForwarder` under the `Log On As` is `Local System`. This is what we want. 
![[Pasted image 20240424155546.png]]

We will also scroll down to the `Sysmon64` service and make sure it's running and the `Log On As` is also `Local System`.

![[Pasted image 20240424155907.png]]

We will go back to `SplunkForwarder` service. Right-click and press `Restart`. 

![[Pasted image 20240424160029.png]]

We wait. 

![[Pasted image 20240424160052.png]]

We get this message but its ok. 

![[Pasted image 20240424160159.png]]

We will press ![[Pasted image 20240424160228.png]] which is located on the left side. 

![[Pasted image 20240424160312.png]]

and it will do its thing.
![[Pasted image 20240424160342.png]]

It is done. We now have finished our process for our updated `inputs.conf` file and will finalize our `Splunk` Server configurations. 

#### Splunk Server configurations 
***
Let's head to our `Splunk` Server portal that we access through our web browser by entering `192.168.10.10:8000` into the address bar and logging in. 

NOTE: Make sure `Splunk` Server is powered on or else will not work. Also remember it listens on port 8000.

![[Pasted image 20240424174142.png]]

Once we log in and get this to this screen, then we will go to `Settings` at the top and click `indexes`.

![[Pasted image 20240424175622.png]]

![[Pasted image 20240424175903.png]]

This is the `Indexes` page for `Splunk`. As we can see there are currently 15 indexes and if we scroll down, then we will notice there isn't an `endpoint` index. So, we will create one.

![[Pasted image 20240424180355.png]]****

At the top right of the page we will see a button that looks like this.

![[Pasted image 20240424180602.png]]

We will click that. We add the name `endpoint` to the "Index Name" and we click `Save`. 

![[Pasted image 20240424180728.png]]

Now we see the `enpoint` index on the list.

![[Pasted image 20240424180849.png]]

Now we must enable our `Splunk` Server to receive data. So, we will click on `Settings` again and go to `Forwarding and receiving` under `DATA`.

![[Pasted image 20240424181332.png]]

Now we want to click on `Configure receiving` under `Receive data`.

![[Pasted image 20240424181432.png]]

We will click on the button that says `New Receiving Port`.

![[Pasted image 20240424181606.png]]

We will configure the `9997` default port that we mentioned previously and click `Save`.

![[Pasted image 20240424181748.png]]

If we have everything set up correctly, then we should start seeing data come in from our "Target PC" aka. Windows 10. 

To check this, we will go to `Apps` in the top left corner and click `Search & Reporting`.

![[Pasted image 20240424194649.png]]

We then search for our `endpoint` index

![[Pasted image 20240424195729.png]]

We see 2,982 events in the last 24 hours and 1 host.

![[Pasted image 20240424195910.png]]

If we look at `host` we see `Target-PC`. 

![[Pasted image 20240424200707.png]]

If we look at `source` we see the 4 that we configured with the `inputs.conf` file.

![[Pasted image 20240424200838.png]]

#### Installing Splunk Universal Forwarder and Sysmon on Active Directory Server 2022 aka. "ADDC01"

We will follow the same exact steps as we did previously to install `Splunk Universal Forwarder` and `Sysmon` on our Windows 10 AKA "Target-PC" for our Active Directory Server AKA "ADDC01". I will show some of the steps but not as thorough as before.  Refer to this section to follow the detailed steps. [[#Installing Splunk Universal Forwarder and Sysmon on Windows 10 AKA "Target PC"]].

First things first, we will rename the "Active Directory Server" to "ADDC01" (Active Directory Domain Controller)

![[Pasted image 20240424203534.png]]

We will also change the IP address to a static IP address. Specifically to `192.168.10.7` to match our diagram. 

![[Pasted image 20240424203802.png]]

We see that we have connectivity and that our Active Directory Server can talk with our Splunk Server.

NOTE: Active Directory Server CAN NOT ping our Windows 10 "Target-PC" because they are both Windows and ICMP (ping) is blocked by default by Windows Defender. We would have to create an inbound rule to allow pings. We see that the Active Directory Server can ping the Splunk Server so that's good enough to assume that we also are connected with the Windows 10 "Target-PC". 
![[Pasted image 20240424204059.png]]

As we did previously on our Windows 10 "Target-PC", we will download `Splunk Universal Forwarder` onto our Active Directory Server "ADDC01". 

![[Pasted image 20240425140110.png]]

We go through the same Setup process for the `Splunk Universal Forwarder` by setting our "Receiving Indexer" to our `Splunk Server`.

![[Pasted image 20240425140341.png]]

We install `Sysmon` with the same "sysmonconfig.xml" configurations. 

![[Pasted image 20240425140912.png]]


#### Instruct Splunk Universal Forwarder (on Active Directory Server AKA "ADDC01") on what we want to send to our Splunk Server. 
***
We will follow the same exact steps as we did previously to instruct `Splunk Universal Forwarder` on what we want to send to our `Splunk` Server on our Windows 10 AKA "Target-PC" for our Active Directory Server AKA "ADDC01". I will show some of the steps but not as thorough as before.  Refer to this section to follow the detailed steps. [[#Instruct Splunk Universal Forwarder (on Windows 10 AKA "Target-PC") on what we want to send to our Splunk Server.]]

We made our new `inputs.conf` file in `notepad` and saved it within the `local` folder. 

![[Pasted image 20240425143917.png]]

We restart the `SplunkForwarder` service.

![[Pasted image 20240425144214.png]]

Now everything is done for the Active Directory Server AKA "ADDC01". 

 Let's head to our `Splunk` Server portal that we access through our web browser by entering `192.168.10.10:8000` into the address bar and logging in. 

We want to check that our "ADDC01" VM is showing up within the `host` section and that things are running as they should. 

We now see both our hosts that we set up to send data to our `Splunk` Server. 

![[Pasted image 20240425145446.png]]


#### Set up Active Directory as Domain Controller
***

Now we will head to our `Server Manager` Dashboard and click on `Manage` in the top right. 

![[Pasted image 20240424204854.png]]

Then `Add Roles and Features`

![[Pasted image 20240424204928.png]]

When we see this screen, we press `Next`.
![[Pasted image 20240424205017.png]]

Make sure `Role-based or feature-based installation` is selected and press `Next`.

![[Pasted image 20240424205158.png]]

Since we only have one server and it's already selected, we will press `Next` again. 

![[Pasted image 20240424205319.png]]

We then want to select `Active Directory Domain Services`.

![[Pasted image 20240424205416.png]]

Select `Add Features`

![[Pasted image 20240424205521.png]]

We click `Next`.

![[Pasted image 20240424205623.png]]

`Next` again

![[Pasted image 20240424205715.png]]

and again.

![[Pasted image 20240424205741.png]]

Finally, `Install`.

![[Pasted image 20240424205828.png]]

We see the installation is done. 

![[Pasted image 20240424210340.png]]

We go back to `Server Manager` and see a yellow ! icon next to the `flag` icon. We click on that and then on `Promote this server to a domain controller`.

![[Pasted image 20240424210513.png]]

A new screen opens up. We want to select `Add a new forest` because we are creating a brand new domain. 

![[Pasted image 20240424210752.png]]

We also want to name the `Root domain name:`, for me I named mine "ChumBucket.local"

NOTE: The domain name must have a top level domain that is why we put ".local" instead of simply "ChumBucket"

The ".local" could be anything such as ".test"

![[Pasted image 20240424211036.png]]

We then press `Next`.

![[Pasted image 20240424211433.png]]

We create a `Password` and press `Next`.

![[Pasted image 20240424211620.png]]

`Next`

![[Pasted image 20240424211654.png]]

`Next`

![[Pasted image 20240424211742.png]]

NOTE: These will be the paths used to store our database file named `NTDS.dit`, which is a file that attackers love to target because it contains everything related to Active Directory including password hashes. IF we notice any unauthorized activity towards this file then we can assume that our entire domain is sadly compromised. 

We press `Next`.

![[Pasted image 20240424211834.png]]

`Next`

![[Pasted image 20240424212357.png]]

`Install`

![[Pasted image 20240424212442.png]]

We have to reset have this installation but when we go to log back in, we notice our domain followed by a \, which means that we have successfully installed `ADDS (Active Directory Domain Services)` and promoted our Active Directory Server to a `Domain Controller` 

![[Pasted image 20240424213508.png]]


#### Add Users to Domain
***

Now we will log in and create some users. We will go to `Tools` in our `Server Manager` and then `Active Directory Users and Computers`.

![[Pasted image 20240424213807.png]]

Here we can create objects such as users, computers, groups, etc. 

![[Pasted image 20240424213918.png]]

We will expand our Domain 

![[Pasted image 20240424214033.png]]

If we click `Builtin`, we see all of the groups Active Directory has automatically created on the hand side. We can double click any of these to find out more about them by reading the description. 

![[Pasted image 20240424214141.png]]

If we double click `Administrators` we can see the description. 

![[Pasted image 20240424214357.png]]

If we click `members`, we can see who is assigned to this group. 

![[Pasted image 20240424214614.png]]

If we click `Member Of`, we can see what other groups this group is in.

NOTE: CAN NOT add additional groups within a `Builtin` group but you can create a custom group and add that `Builtin` group to the custom group. 
![[Pasted image 20240424214721.png]]

For example, if we go back to the groups and right-click on a blank space and go to `New` -> `Group`. 

![[Pasted image 20240424215037.png]]

Type "Test" and `OK`

![[Pasted image 20240424215258.png]]

Double click `Test`

![[Pasted image 20240424215351.png]]

Click on `Member Of` and notice that we can now press `Add...` unlike before.

![[Pasted image 20240424215438.png]]

If we click `Add...` then type "Administrators" in `Enter the object names to select` box and click `Check Names`. We notice "Administrators" gets underlined. We click `OK`. 

![[Pasted image 20240424215553.png]]

Now we see we made "Administrators" a `Member Of` the `Test` group. 

![[Pasted image 20240424215808.png]]

Now we go to `Users`

![[Pasted image 20240424221051.png]]

We could make a user similar to the way we right clicked the empty space in the `Builtin` folder but usually users would be within different departments such as HR or IT. Departments in this context would be `Organizational Units`. 

We can create a new `Organizational Unit` by right clicking the `Domain` -> `New` -> `Organizational Unit`.

![[Pasted image 20240424221631.png]]

We will name this `IT` and click `OK`.

![[Pasted image 20240424221716.png]]

We see that `IT` is now a folder on the side.

![[Pasted image 20240424221809.png]]

we can add users within it. 

![[Pasted image 20240424221910.png]]

We will out our new User's `Name` and `User logon name`. We click `Next`.

![[Pasted image 20240424222149.png]]

We give our user a new password and uncheck `User must change password at next logon` since we are within a lab environment. Click `Next`.

![[Pasted image 20240424222247.png]]

`Finish`

![[Pasted image 20240424222415.png]]

We see our new user in the `IT` folder.

![[Pasted image 20240424222514.png]]

We will create another `Organizational Unit` named `HR` We create a new user within `HR`.

![[Pasted image 20240424222731.png]]

#### Add Windows 10 AKA "Target-PC" to our Domain. 
***

On our Windows 10 AKA "Target-PC" we want to search up `PC` -> `Properties`. 

![[Pasted image 20240425150604.png]]

Scroll down and click on `Advanced system settings`.

![[Pasted image 20240425150730.png]]

We want to click on the tab `Computer Name`

![[Pasted image 20240425150902.png]]

Then on the `Change...` button. 

![[Pasted image 20240425150927.png]]

We want to make sure we select `Domain:` 

**![[Pasted image 20240425151007.png]]**

We type in our `.local` Domain and press `OK`.

![[Pasted image 20240425151206.png]]

We see this error.

This is because our "Target-PC" AKA Windows 10 doesn't know how to resolve the `.local` domain. So, we will fix that. 

![[Pasted image 20240425151249.png]]

We will right-click our `Network Adapter` ![[Pasted image 20240425151619.png]] on the right hand side of the task bar and click `Open Network & Internet settings`.

![[Pasted image 20240425151743.png]]

We will click on `Change adapter options`

![[Pasted image 20240425151831.png]]

Right click our adapter and click `Properties`

![[Pasted image 20240425151921.png]]

Double click `Internet Protocol Version 4 (TCP/IPv4)`

![[Pasted image 20240425152013.png]]

We currently have our `Preferred DNS server:` pointing to Google's DNS and we want to change this to our `Domain Controller`. 

![[Pasted image 20240425152113.png]]

So, we will change that to our `Domain Controller`'s address of `192.168.10.7`.

![[Pasted image 20240425152435.png]]

It's important that we also click `OK` here as well or else it will not save. 

![[Pasted image 20240425152559.png]]

We will open `Command Prompt` and make sure everything is correct by running the command 

```Command Prompt
Ipconfig /all
```

In which, we do see now that our `DNS Servers` is `192.168.10.7`. 
![[Pasted image 20240425152917.png]]

Now we can add "Target-PC" AKA Windows 10 to our Domain without getting an error like before. 

So we go back to this and press `OK`.

NOTE: Be sure to have your Active Directory Server AKA "ADDC01" powered on as well. 

![[Pasted image 20240425153231.png]]

We see this pop up.

![[Pasted image 20240425153539.png]]

We will use the `Administrator` account of the Active Directory Server because it has the proper permissions to join the domain. 

![[Pasted image 20240425153842.png]]

We get this and press `OK`.

NOTE: In a real world environment we would create users and put them into a custom group that are authorized to allow computers to join the domain. 

![[Pasted image 20240425154020.png]]

We press `Ok` again

![[Pasted image 20240425154207.png]]

We `Restart Now`

![[Pasted image 20240425154247.png]]

Back on the log in screen. We want to log into our new user account that we made. For me, it was called "Sheldon.Plankton"

![[Pasted image 20240425154419.png]]

To do this, we will select `Other user` in the bottom left.

![[Pasted image 20240425154625.png]]

We want to make sure that we are signing into our domain. 

![[Pasted image 20240425154828.png]]

We fill out the login info for our user and sign in. 

![[Pasted image 20240425154808.png]]

It begins to log in

![[Pasted image 20240425154924.png]]


#### Use Kali Linux to perform a brute force attack
***

dd



















#### Setting up Atomic Red Team on our Windows 10 AKA "Target-PC" to generate telemetry and view on Splunk. 
***

dd

[back](./)
