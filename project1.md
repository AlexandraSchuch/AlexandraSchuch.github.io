# Active Directory Project

### Objective

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

![Active-Directory-Diagram7](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/86b401f2-c979-4c51-8438-260b67aca06c)

### PRE-Project Checklist

- [ ] Download all required VMs- Windows 10 (Target-PC), Windows Server (Active Directory), Kali Linux (Attacker), Ubuntu Server (Splunk)
- [ ] Change all VM network's to NAT Network that you create to have all devices on same network with internet.
- [ ] Make network diagram

<br></br>
### Setting up static IP address for Splunk Server
***
As you can see our current IP address is `192.168.10.4` and we need to change that to match our diagram address of `192.168.10.10`

![Pasted image 20240420023738](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/cca71461-8681-4a33-ab71-6eb87401b10c)

We need to go into the Network Config and change some things to make the IP address static so we run the command
```Splunk
sudo nano /etc/netplan/00-installer-config.yaml
```
![Pasted image 20240420024317](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/15412bf3-a93d-412b-9339-c4031bd2b773)

We are then present with this 

![Pasted image 20240417204446](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c39106bb-1314-4511-965e-51f2c9c8f3c2)


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
![Pasted image 20240420024606](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d0e555de-7b5f-4ac1-9508-d3d633096a81)


We apply the new Netplan so we have our static IP by using this command. We ignore the warning. It doesn't effect this. 

```
sudo netplan apply
```

![Pasted image 20240417205042](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e2a55b0b-0ab8-4218-9339-c81d7bb15412)

To check that the IP addresses has now changed to the static addresses we changed it to, we run this command.
```
ip a
```
we see that we were successful at changing the IP address because now it shows `192.168.10.10/24`

![Pasted image 20240417205134](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/06766005-fa5a-444a-97cd-69eb7952528c)


We ping google.com to make sure we have a connection by doing the command
```
ping google.com
```
![Pasted image 20240417205248](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a4b64c12-1481-4506-b9d0-6d605e556a59)


### Install Splunk on HOST machine.
***

We want to install `Splunk` on our HOST computer, so we will head over to `Splunk.com` on our browser through our HOST computer.

![Pasted image 20240501203059](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/97a6f497-05ba-4f0a-b5bd-3b80d4e56491)


If you DO NOT have an account already then you want to click `Sign up`, otherwise `Log in`.
![Pasted image 20240501203133](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/eaa11c9b-ff79-4659-a218-ea93beeef94e)


Once you're signed in, we will hover over `Products` and then click on `Free Trials & Downloads`

![Pasted image 20240501203314](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9705e136-1682-4688-8447-39755874699e)


Scroll down to `Splunk Enterprise` and click `Get My Free Trial`

![Pasted image 20240501203433](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/bce4ceaf-9a64-4f87-aa15-6a644b39429d)


We want to make sure that we select `Linux` as our OS because our Splunk Server is a Ubuntu Server.

![Pasted image 20240501204255](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9beea78e-3144-4c0d-92b0-4fdfb6ea9f02)


We are interested in the `.deb` file. We will select `Download Now`.

![Pasted image 20240501204356](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5267caa7-1976-4887-b3e9-b70a72c90363)


We will save this in a directory of our choice. 

![Pasted image 20240501204501](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ab14b587-8c92-45d7-9982-0884eb7461cc)


### Download guest add ons for VirtualBox
***

We want to download the `guest add ons` for VirtualBox
```
sudo apt-get install virtualbox-guest-additions-iso
```

![Pasted image 20240420030616](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/09b54308-0194-4dd0-bdc8-750b673c3cc3)


We will press `y` and `enter`. Then wait for everything to download.

![Pasted image 20240420030925](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5274b6c9-b71e-44a8-b0a5-25965bd9c812)


Once we see this screen, we press `enter` and our package should install.

![Pasted image 20240420031144](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ef024ddd-3e4a-4ac6-bcb1-5fc290d4d009)


Now we want to click `Devices` on our VM at the top 

![Pasted image 20240420031415](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2bb6d884-ad8b-43f1-b6c1-75375b5fe334)


We then want to go to `Shared Folders` and then to `Shared Folders Settings`

![Pasted image 20240420031556](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/1076f3c4-8669-4f67-bbf3-9834e9affb07)

We are presented with this.

![Pasted image 20240420031705](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/abd42a38-30c0-470b-8738-162032ec6ad3)


We want to add a new folder by clicking on this icon on the right hand side.

![Pasted image 20240420031748](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0f462cdb-50d2-4c3f-8a48-3b49242cbc48)


For folder path, we want to use the same folder where we saved the `Splunk .Deb package` for me it was in an `ActiveDirectoryProject` folder. We will also checkmark `Read-only`, `Auto-mount`, and `Make Permanent`. Then press `OK`  

![Pasted image 20240420032120](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4141d990-14cd-4085-b267-308465575dd6)

We then will reboot the VM running the command
```
Sudo reboot
```

![Pasted image 20240420032337](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c793dc52-b379-4763-818d-228a24766b9f)

We relog back in.

Then we want to add our user to vboxsf. In my case it's `chum_bucket` by running the command
```
sudo adduser chum_bucket vboxsf
```

![Pasted image 20240417211043](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9ecf2244-adab-4b23-8612-9ff6cba8a7f1)

If we get this 

![Pasted image 20240420033006](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/32dad1cc-d914-4f95-a734-aaabe9f48ec3)

then we might need some additional guest installations for VirtualBox so we will get them by running the command 
```
sudo apt-get install virtualbox-guest-utils
```

![Pasted image 20240420033336](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/50349245-628e-4c4b-b916-79c03862060b)

We will reboot again and relog in. 
```
sudo reboot
```

We will try again to add the user to the vboxsf and we see that it works.

![Pasted image 20240417211744](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9d3a24e8-a3ff-454a-83ec-8e7ce099c269)

We will make a new `directory` called `share` by running the command
```
mkdir share
```

We will check to make sure that its there by running the `ls` command

![Pasted image 20240417211817](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/63ac6dad-4bd0-479c-9e68-2800af1551f8)

Now that the `Share` directory has been made, we want to run the following command to mount our shared folder onto our share directory. 

NOTE: if we forget where the folder path is, then we can go back to the top and click `Devices` -> `Shared Folders` -> `Shared Folders Settings` -> `Shared Folders` and look at `Folder Name`

```
sudo mount -t vboxsf -o uid=1000,gid=1000 ActiveDirectoryProject share/
```
![Pasted image 20240417212250](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7a6ef735-1a58-4ba7-a520-a0f404c2be12)

NOTE2: if we get an error when entering this command then exit and relog back in because when you add a user into a new group, it doesn't take effect until you log out. 

We will run the command 
```
ls -la
```
and we see that `share` is now highlighted

![Pasted image 20240420035421](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b2f0e97f-6a99-4197-85e1-0ee99d665361)

we will change directories into `share` by running the command 
```
cd share
```
![Pasted image 20240420035047](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/1409cd3a-1533-41b0-a111-5964107dc775)

we will run the command again
```
ls -la 
```
![Pasted image 20240420035331](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e0592f11-b066-42bc-9e57-38bbd59ba286)

We will then install `Splunk` by running the command
```
sudo dpkg -i (splunk-9.2.1-.deb) 
```
![Pasted image 20240417212653](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5a481bde-425d-44f3-8c23-d8d2ecddb083)

Now we will change into the directory where `Splunk` is located on our server by running the command

```
cd /opt/splunk
```
and run the command to see all 
```
ls -la
```
![Pasted image 20240417212743](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/66e7f5ec-dc3a-455c-affb-b4236dde1156)

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

![Pasted image 20240417212824](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4c63a19a-c7e4-4270-af7f-2a284b275dc2)

We will run this command to run `splunk`
```
./splunk start
```
![Pasted image 20240417212906](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c5861190-52e6-4692-8458-6eaf0f605c5b)

We will get a license and terms agreement. We will press `q` then `y` then `enter`. We then enter an admin name and password. 

![Pasted image 20240420041107](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/901d7f44-0783-4624-9582-8e62204fda44)

Once installation is completed we see this.

![Pasted image 20240420041631](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ad33337f-db64-492e-9e36-ae4743c18411)

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

![Pasted image 20240417213321](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/8c8cb525-74b6-448a-b5da-4ce6427253cb)

#### Installing Splunk Universal Forwarder and Sysmon on Windows 10 AKA "Target PC"
***
On Windows 10 machine, we search up `pc` and go into `Properties` -> `Rename this PC`. 

We rename it to `target-PC` so that it makes things more simple to understand in the logs. We then reboot and check in the `pc` -> `Properties` again to verify the name has changed.

![Pasted image 20240417213836](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b7d6d4f4-b6d4-42a2-8cf4-97dd0ea6d895)

Next, we want to search for `cmd` and see our current IP address by running the command

```CommandPrompt
ipconfig
```

![Pasted image 20240420142237](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/447bdccb-69fc-4867-892b-6f2fb27332be)

Right now the Windows 10 machine is getting assigned a dynamic IPv4 address through DHCP and we need to change it to the static address `192.168.10.100` to match our network diagram. 

So, to do that we will right click the `network`![Pasted image 20240420142636](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ef7b470a-31d4-4d40-9c92-dd23e280798c)
 icon in the bottom right. Then click `Open Network & Internet settings` -> `Change adapter options`. Right click `Ethernet` -> `Properties` -> `Internet Protocol Version 4 (TCP/IPv4)` 

![Pasted image 20240420142615](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d9965ed5-47f3-4891-a6c8-1c7b59f6e8cd)

![Pasted image 20240420142839](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/fe1405af-148c-4581-b4db-a036e09ce5f5)

![Pasted image 20240420143105](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/decac03f-fd5f-4931-ae9b-7c02fdb9311a)

We now will set our static IP with the configurations below

![Pasted image 20240417214706](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e4a55a85-8696-4691-8039-a002534ee7f6)

Then we run the `ipconfig` command again to verify that the IP address has changed to `192.168.10.100` and we see that it has.

![Pasted image 20240417215147](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/47bb5e54-2ad3-4d4e-9407-9fa69d8b2033)

We will now open up a web browser and try accessing our `splunk` server by typing in its IP address that we statically assigned, followed by `:8000` as so, `192.168.10.10:8000` into the address bar. 

NOTE: `Splunk` listens on port 8000 and that is why we specify it. Also make sure your `Splunk` server is powered on or else it won't work.

![Pasted image 20240420143848](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2636a9e5-f696-49e1-aa74-db7eb8341e85)

We see this screen and know that we can access `splunk`

![Pasted image 20240420144439](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c1bdb360-03f4-4593-afae-23a68a68cb3e)

Now we will install `Splunk Universal Forwarder` by heading to the `splunk.com` site and logging into our account. 

![Pasted image 20240420144735](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/6a585e2f-b6a2-4784-a033-96219c3f3313)

Once signed in, we click `Products` -> `Free Trials & Downloads` and scroll down till we see `Universal Forwarder`. This is what we will use to send data to our `Splunk` server from our `Target PC` aka. windows 10 machine. 

![Pasted image 20240420145330](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/77b0d8b5-08ec-42cd-8a39-737c11cf1dbd)

Click `Get My Free Download` and then on the OS that we are running, which will be the `Windows 10 - 64-bit` download.

![Pasted image 20240420150328](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/880a5394-e7ba-4cd1-91b5-0faa829559e7)

Once the download is complete, we will open it up through the `Downloads` folder in `File Explorer`.

![Pasted image 20240420150635](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/cfb0047c-3180-4e68-96a3-07c5e82e2801)

We will click the `Check this box to accept the License Agreement`. Make sure we have the `An on-premises Splunk Enterprise instance` picked and click `Next`.

![Pasted image 20240420150850](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b25cdde8-b9d7-499d-ae42-c54b6933b75d)

We will put `admin` as `Username:` and leave the `Generate random password` checked. Then click `Next`. 

![Pasted image 20240420151025](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b9ee54a0-55c6-41de-9ff5-61cf7c7b0f2d)

We do not have a deployment server so we will skip this and press `Next`.

![Pasted image 20240420151140](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d802ae11-d0a5-4d1f-be1b-a2089805f517)

The `Receiving Indexer` is going to be our `Splunk` server so we will put it's IP and put the `port`  as the default `9997` for receiving events.

![Pasted image 20240420151320](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/8eb64a27-0402-4cf1-854f-e78bd84186d3)

We click `Install`. 

![Pasted image 20240420151508](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/87d2145d-7206-455c-ab3c-7cd57ac6a388)

Now we are going to install `Sysmon` by going back to our web browser and searching `sysmon` and selecting the `sysmon` by `Sysinternals`. 

![Pasted image 20240420151740](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/639fa5c1-a137-4c7c-b75a-1434664b3bfa)

We scroll down and click `Download Sysmon`. 

![Pasted image 20240420151837](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/617341be-962e-4760-b1a7-0bbca1e8bbe5)

Now we need to get our Sysmon configuration, which will be the "sysmon olaf config." We will search that on our browser to find the github page.

![[Pasted image 20240424141835.png]]

Then we will scroll down till we see `sysmonconfig.xml` and click on it. 

![Pasted image 20240424141929](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e0146876-21c3-4a12-8fc5-8b7ab272915c)

We will then click the ![Pasted image 20240424142110](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3a5a3f42-62cf-4c90-8d95-dff1e8866c38)
button on the right side of the page. Right-click `save as` and then save it within the `Downloads` directory.

![Pasted image 20240424142214](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c2fbaced-f816-478e-b73f-fd04f46db55a)

Now lets go to our `Downloads` directory. Right-click on our `sysmon` zipped folder and press `extract all`.

![Pasted image 20240424142530](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d7293858-f428-4615-8d32-18d446b7436b)

`extract`.

![Pasted image 20240424142604](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2f2a941c-9793-4ce4-84b8-00a851b8cd3e)

We see the `sysmon` folder now and will click on the path bar and copy the file path. 

![Pasted image 20240424142812](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/32686248-68ca-454b-ab5b-01005b717428)

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

![Pasted image 20240418004033](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7add7e2e-fb4d-4a28-89cb-cf80e50543bf)

We press enter and it begins to install. 

![Pasted image 20240418004233](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5d38d03a-2201-4e13-8209-cb9f57742868)

We press `Agree`

![Pasted image 20240424144001](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f6c240a8-5441-4f70-8732-be1ae85c0a93)

We also see that `Splunk Universal Forwarder` has also finished.

![Pasted image 20240424145314](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/698a72f5-3060-42b5-af94-0a6090384b57)

### Instruct Splunk Universal Forwarder (on Windows 10 AKA "Target-PC") on what we want to send to our Splunk Server. 
***

We must configure a file named `inputs.conf`. Which if we open our `File Explorer` then we can locate it by going to `This PC` -> `Local Disk (C:)` -> `Program Files` -> `SplunkUniversalForwarder` -> `etc` -> `System` -> `default` and we see the file `inputs.conf`. We will leave it alone. 

![Pasted image 20240424150705](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/1e2a8b67-2fc4-4335-89fc-d61db5454fa8)

We will go back a directory to `system` and go into `local` Here we will create a new `inputs.conf` file. 

NOTE: it is VERY important that we DO NOT edit the `inputs.conf` file within the `default` directory. That one is there just in case we mess anything up and need to replace the file with the default one. 

![Pasted image 20240424151132](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0969a6bc-56ea-485a-adc3-0e921a7bf101)

However, we notice that we can only make folders within the directory and we want to create the `inputs.conf` file. So, instead we will open up `notepad` and create the file that way and save it within the `local` directory.

![Pasted image 20240424151543](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/8f9c84f6-2612-49be-9f88-9265464f3d16)

So we open `notepad` and `run as administrator`. We copy the contents (below) of the original `inputs.conf` within the `default` directory and paste it into the notepad file.

```CopyThis

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

![Pasted image 20240424152924](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/24bad108-68fa-4d0e-82c5-de635baa896d)

So, we save the notepad file under the `local` directory like we mentioned before, name it `inputs.conf`, change the "save as type:" to `All Files`, and hit `save`. 

![Pasted image 20240424153759](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ae5f0606-c73a-4252-8b2a-8a31e74e04ac)

Now we see `inputs.conf` under our `local` directory.

![Pasted image 20240424153926](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/41fd850c-2df1-48c8-b517-0747a93d230c)

NOTE: Anytime we update the `inputs.conf` file we will need to restart `Splunk Universal Forwarder` service. 

So, we will search up `Services` and `Run as administrator` 

![Pasted image 20240424154153](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b61df06e-cd8d-432d-b4b6-74c03795818d)

Click on any of the services once and type `s` to go to the services that start with "s." 

![Pasted image 20240424154334](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f55bacd5-d72c-4d8c-9d97-0ec56ad7a8e4)

We will look for the `SplunkForwarder` service.

![Pasted image 20240424154426](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/bf531a21-9d05-4f3f-ad34-50e07e50475c)

If we scroll to the right of `SplunkForwarder` and look under the column `Log On As`, we might notice it says `NT SERVICE`. We DO NOT want this. 

![Pasted image 20240424154739](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/07008c87-6978-44de-ac45-1e48bffaa486)

We will double click the service to view its `properties`.

![Pasted image 20240424154912](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/779082f6-978b-416a-8334-b75448feaefa)

We will go to `Log On`. 

![Pasted image 20240424155010](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/91c78d1d-17f7-4dfa-b072-d2aebf551733)

If we see `NT SERVICE` for "This account:" then it might not be able to collect logs due to permissions so instead we will click `Local System account` and hit `Apply`. 

![Pasted image 20240424155310](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7984a1d5-150b-4355-abaf-4becde27993d)

We see this message. This is fine since we planned on restarting the service anyways because we updated our `inputs.conf` file. 

![Pasted image 20240424155359](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a57973e2-6414-493b-849c-b98887567810)

Notice that it now says that `SplunkForwarder` under the `Log On As` is `Local System`. This is what we want. 

![Pasted image 20240424155546](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/481eb0ee-b7e2-40a7-b004-8055f5c5569a)

We will also scroll down to the `Sysmon64` service and make sure it's running and the `Log On As` is also `Local System`.

![Pasted image 20240424155907](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/35f84907-3426-4225-9333-705a86966db7)

We will go back to `SplunkForwarder` service. Right-click and press `Restart`. 

![Pasted image 20240424160029](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4424648d-a303-4f09-8617-4f7a68b2e661)

We wait. 

![Pasted image 20240424160052](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b97fbc20-5e74-40ba-9a5d-3e37467519f7)

We get this message but its ok. 

![Pasted image 20240424160159](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7fcf166b-f85a-4047-9ebe-3dcc0791120e)

We will press ![Pasted image 20240424160228](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/63b0f0cf-c6a2-4de9-a540-2eb417092891)
 which is located on the left side. 

![Pasted image 20240424160312](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b6a3ff82-34b3-4dd1-bc4e-98e4455a928f)

and it will do its thing.

![Pasted image 20240424160342](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b68e25ce-5694-46d0-bbbc-7d26b47e61dd)

It is done. We now have finished our process for our updated `inputs.conf` file and will finalize our `Splunk` Server configurations. 

### Splunk Server configurations 
***
Let's head to our `Splunk` Server portal that we access through our web browser by entering `192.168.10.10:8000` into the address bar and logging in. 

NOTE: Make sure `Splunk` Server is powered on or else will not work. Also remember it listens on port 8000.

![Pasted image 20240424174142](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/8b1ed54f-a906-4064-a359-3661e8e5e858)

Once we log in and get this to this screen, then we will go to `Settings` at the top and click `indexes`.

![Pasted image 20240424175622](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/125cd3c1-b1b2-48bf-bfe7-aac07992dd65)

![Pasted image 20240424175903](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/bc7178fc-1f1c-4b36-a287-8c9b44a45452)

This is the `Indexes` page for `Splunk`. As we can see there are currently 15 indexes and if we scroll down, then we will notice there isn't an `endpoint` index. So, we will create one.

![Pasted image 20240424180355](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/321287c9-b009-4805-8812-be50e9e0ba3a)

At the top right of the page we will see a button that looks like this.

![Pasted image 20240424180602](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5176052f-e5d3-4e9f-ac32-7e9528756dac)

We will click that. We add the name `endpoint` to the "Index Name" and we click `Save`. 

![Pasted image 20240424180728](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/aec074a0-b7ab-451c-b5b4-a3bfb00d653d)

Now we see the `enpoint` index on the list.

![Pasted image 20240424180849](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/dae436bc-d1e0-404e-9062-6480c3c14ad9)

Now we must enable our `Splunk` Server to receive data. So, we will click on `Settings` again and go to `Forwarding and receiving` under `DATA`.

![Pasted image 20240424181332](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a524405c-32aa-4c75-a327-5ac4a36bc4e1)

Now we want to click on `Configure receiving` under `Receive data`.

![Pasted image 20240424181432](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/331181c7-20dc-4a66-87db-d4efc7b9101a)

We will click on the button that says `New Receiving Port`.

![Pasted image 20240424181606](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e2eb3127-b8c1-43fb-b880-c14e40105ae4)

We will configure the `9997` default port that we mentioned previously and click `Save`.

![Pasted image 20240424181748](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4bd245ed-f4bc-4b51-9236-df617e1be766)

If we have everything set up correctly, then we should start seeing data come in from our "Target PC" aka. Windows 10. 

To check this, we will go to `Apps` in the top left corner and click `Search & Reporting`.

![Pasted image 20240424194649](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0150df65-b399-4ca7-8845-44153a089159)

We then search for our `endpoint` index

![Pasted image 20240424195729](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2715ac01-5272-4d50-95ca-46637207f40f)

We see 2,982 events in the last 24 hours and 1 host.

![Pasted image 20240424195910](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/623a834b-2c0d-44b2-9fae-bbbe382982bd)
424195910.png]]

If we look at `host` we see `Target-PC`. 

![Pasted image 20240424200707](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3fba80f2-51b0-4b2f-a34e-42e5f886e1fe)

If we look at `source` we see the 4 that we configured with the `inputs.conf` file.

![Pasted image 20240424200838](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4460453a-4279-43b1-a12c-93eac702ee2b)

### Installing Splunk Universal Forwarder and Sysmon on Active Directory Server 2022 aka. "ADDC01"
***

We will follow the same exact steps as we did previously to install `Splunk Universal Forwarder` and `Sysmon` on our Windows 10 AKA "Target-PC" for our Active Directory Server AKA "ADDC01". I will show some of the steps but not as thorough as before.  Refer to this section to follow the detailed steps. [[#Installing Splunk Universal Forwarder and Sysmon on Windows 10 AKA "Target PC"]].

First things first, we will rename the "Active Directory Server" to "ADDC01" (Active Directory Domain Controller)

![Pasted image 20240424203534](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d08ccd78-2571-491b-b962-2cd8d6e4dcec)

We will also change the IP address to a static IP address. Specifically to `192.168.10.7` to match our diagram. 

![Pasted image 20240424203802](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/1d1d3534-c0dc-4d90-9870-a8b870d687dc)

We see that we have connectivity and that our Active Directory Server can talk with our Splunk Server.

NOTE: Active Directory Server CAN NOT ping our Windows 10 "Target-PC" because they are both Windows and ICMP (ping) is blocked by default by Windows Defender. We would have to create an inbound rule to allow pings. We see that the Active Directory Server can ping the Splunk Server so that's good enough to assume that we also are connected with the Windows 10 "Target-PC". 

![Pasted image 20240424204059](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/6605bb48-6d08-42d7-9fdd-a72547beab09)

As we did previously on our Windows 10 "Target-PC", we will download `Splunk Universal Forwarder` onto our Active Directory Server "ADDC01". 

![Pasted image 20240425140110](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4194bcef-d81e-4bd5-8238-b5ff4ee582be)

We go through the same Setup process for the `Splunk Universal Forwarder` by setting our "Receiving Indexer" to our `Splunk Server`.

![Pasted image 20240425140341](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/6e8f9cf1-e1a5-4f92-8750-a4abdca34faf)

We install `Sysmon` with the same "sysmonconfig.xml" configurations. 

![Pasted image 20240425140912](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/20f60cf3-a014-48c9-864d-ff4da060785a)


### Instruct Splunk Universal Forwarder (on Active Directory Server AKA "ADDC01") on what we want to send to our Splunk Server. 
***
We will follow the same exact steps as we did previously to instruct `Splunk Universal Forwarder` on what we want to send to our `Splunk` Server on our Windows 10 AKA "Target-PC" for our Active Directory Server AKA "ADDC01". I will show some of the steps but not as thorough as before.  Refer to this section to follow the detailed steps. [[#Instruct Splunk Universal Forwarder (on Windows 10 AKA "Target-PC") on what we want to send to our Splunk Server.]]

We made our new `inputs.conf` file in `notepad` and saved it within the `local` folder. 

![Pasted image 20240425143917](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/415238a0-b0e9-4b66-b4f5-6353c60fed2c)

We restart the `SplunkForwarder` service.

![Pasted image 20240425144214](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3f15ba8a-2303-4f82-9846-16b530573133)

Now everything is done for the Active Directory Server AKA "ADDC01". 

 Let's head to our `Splunk` Server portal that we access through our web browser by entering `192.168.10.10:8000` into the address bar and logging in. 

We want to check that our "ADDC01" VM is showing up within the `host` section and that things are running as they should. 

We now see both our hosts that we set up to send data to our `Splunk` Server. 

![Pasted image 20240425145446](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/16f299b0-bc80-4da8-9905-9270c64f4d4b)


#### Set up Active Directory as Domain Controller
***

Now we will head to our `Server Manager` Dashboard and click on `Manage` in the top right. 

![Pasted image 20240424204854](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c1ecab55-eafd-473a-9eed-4e5a71e0c14d)

Then `Add Roles and Features`

![Pasted image 20240424204928](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/bfa36c01-5497-4f1a-8b3d-0a95035ea31e)

When we see this screen, we press `Next`.
![[Pasted image 20240424205017.png]]

Make sure `Role-based or feature-based installation` is selected and press `Next`.

![Pasted image 20240424205158](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c590a2db-8462-4f72-b8d7-15b18a0a11cf)

Since we only have one server and it's already selected, we will press `Next` again. 

![Pasted image 20240424205319](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3de31582-3d01-40ba-8867-ca2604c712d0)

We then want to select `Active Directory Domain Services`.

![Pasted image 20240424205416](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/df1d66dc-74d1-421b-926e-15481d84c4f4)

Select `Add Features`

![Pasted image 20240424205521](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f5b14ec0-5a43-4241-8b8e-0d812681770e)

We click `Next`.

![Pasted image 20240424205623](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/17ba2306-003e-43e9-ad37-aa7f256a2fee)

`Next` again

![Pasted image 20240424205715](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/fd873e93-06ed-463a-8ce1-0ccdd3fac788)

and again.

![Pasted image 20240424205741](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b9388286-1c24-43b3-9561-5aa933ba8155)

Finally, `Install`.

![Pasted image 20240424205828](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f550b5ff-795d-4a57-8e33-d10532b4bcc2)

We see the installation is done. 

![Pasted image 20240424210340](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/10807f6f-c8de-4a4a-95b9-d82e0c7ce3f9)

We go back to `Server Manager` and see a yellow ! icon next to the `flag` icon. We click on that and then on `Promote this server to a domain controller`.

![Pasted image 20240424210513](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/10894a62-dd23-47ed-ac83-a02cf50664c5)

A new screen opens up. We want to select `Add a new forest` because we are creating a brand new domain. 

![Pasted image 20240424210752](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a03f0f18-ab5a-4cd6-8285-6041aa93379c)

We also want to name the `Root domain name:`, for me I named mine "ChumBucket.local"

NOTE: The domain name must have a top level domain that is why we put ".local" instead of simply "ChumBucket"

The ".local" could be anything such as ".test"

![Pasted image 20240424211036](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/55e6f242-6c73-4065-b9ec-ca6f48cf82a1)

We then press `Next`.

![Pasted image 20240424211433](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f38c9c4e-acff-4519-bf05-613347f57ff9)

We create a `Password` and press `Next`.

![Pasted image 20240424211620](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/eeb1ef92-0de0-40bd-bf7c-6cb6c6f990bf)

`Next`

![Pasted image 20240424211654](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/69c248b7-5c3e-4dd0-b2f9-c1fcea35a2d3)

`Next`

![Pasted image 20240424211742](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2fc0ea73-2390-4383-b396-f1b5f4562916)

NOTE: These will be the paths used to store our database file named `NTDS.dit`, which is a file that attackers love to target because it contains everything related to Active Directory including password hashes. IF we notice any unauthorized activity towards this file then we can assume that our entire domain is sadly compromised. 

We press `Next`.

![Pasted image 20240424211834](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/713b9d63-f8c9-420a-ba57-3ff2a69ffdbc)

`Next`

![Pasted image 20240424212357](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/08de5be8-7bd5-4351-81ce-cc99d35b311d)

`Install`

![Pasted image 20240424212442](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/65cda552-0b7c-4d50-8e69-6e29c018c379)

We have to reset have this installation but when we go to log back in, we notice our domain followed by a \, which means that we have successfully installed `ADDS (Active Directory Domain Services)` and promoted our Active Directory Server to a `Domain Controller` 

![Pasted image 20240424213508](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d4c4e7e9-1f9c-460e-ae90-9cc97721f585)


### Add Users to Domain
***

Now we will log in and create some users. We will go to `Tools` in our `Server Manager` and then `Active Directory Users and Computers`.

![Pasted image 20240424213807](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/be8c13aa-15a9-4589-a466-c8a79f27565f)

Here we can create objects such as users, computers, groups, etc. 

![Pasted image 20240424213918](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/80a30d6b-6dd1-423e-9b08-b2632fc48fb7)

We will expand our Domain 

![Pasted image 20240424214033](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3fb6192b-d4ab-4f0f-a120-301604a828d0)

If we click `Builtin`, we see all of the groups Active Directory has automatically created on the hand side. We can double click any of these to find out more about them by reading the description. 

![Pasted image 20240424214141](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0586a233-63f4-4363-b7e5-2c424bcf33b4)

If we double click `Administrators` we can see the description. 

![Pasted image 20240424214357](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ed24c624-2a98-4ce3-90d1-a1d1e82a49d4)

If we click `members`, we can see who is assigned to this group. 

![Pasted image 20240424214614](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/dab2d9ea-7110-4b19-b774-32e5486d5d18)

If we click `Member Of`, we can see what other groups this group is in.

NOTE: CAN NOT add additional groups within a `Builtin` group but you can create a custom group and add that `Builtin` group to the custom group. 

![Pasted image 20240424214721](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3c28c028-177e-4712-8bd1-ec97d35ec1a9)

For example, if we go back to the groups and right-click on a blank space and go to `New` -> `Group`. 

![Pasted image 20240424215037](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/fcd8a948-c15f-4faa-863d-b5492fac9528)

Type "Test" and `OK`

![Pasted image 20240424215258](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/dc2fc87b-f69f-4130-85e1-fc25041549e6)

Double click `Test`

![Pasted image 20240424215351](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f6965004-1b70-4c22-bc3c-ad8cbb0d12df)

Click on `Member Of` and notice that we can now press `Add...` unlike before.

![Pasted image 20240424215438](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/82769d36-52ec-4db9-a96c-952cea58200b)

If we click `Add...` then type "Administrators" in `Enter the object names to select` box and click `Check Names`. We notice "Administrators" gets underlined. We click `OK`. 

![Pasted image 20240424215553](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ff9f0ad0-6c6c-475d-a2ce-995f0602665d)

Now we see we made "Administrators" a `Member Of` the `Test` group. 

![Pasted image 20240424215808](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b280c0ef-571f-4a69-814c-b0656e896fbc)

Now we go to `Users`

![Pasted image 20240424221051](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/bfa6153a-595a-4609-a177-eae3e4c0ac07)

We could make a user similar to the way we right clicked the empty space in the `Builtin` folder but usually users would be within different departments such as HR or IT. Departments in this context would be `Organizational Units`. 

We can create a new `Organizational Unit` by right clicking the `Domain` -> `New` -> `Organizational Unit`.

![Pasted image 20240424221631](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/55458a32-245a-4020-8444-c40a4e9927bd)

We will name this `IT` and click `OK`.

![Pasted image 20240424221716](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/dae936c6-9ba0-4bf8-8f21-39039cda0621)

We see that `IT` is now a folder on the side.

![Pasted image 20240424221809](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/6273dbca-2d2a-4486-be75-7f9baa75d069)

we can add users within it. 

![Pasted image 20240424221910](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7d46a7ca-852f-4fee-b395-e27666ec8f7b)

We will out our new User's `Name` and `User logon name`. We click `Next`.


![Pasted image 20240424222149](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a736197d-cb8f-4eed-8e60-8f8a2f290f99)

We give our user a new password and uncheck `User must change password at next logon` since we are within a lab environment. Click `Next`.

![Pasted image 20240424222247](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/068c28ba-7ce1-40f0-a555-e0d987caab81)

`Finish`

![Pasted image 20240424222415](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5eea47b8-72c1-422e-aacf-6d562f8c69e5)

We see our new user in the `IT` folder.

![Pasted image 20240424222514](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7b7734b6-efb0-4bd3-b291-30370e56f850)

We will create another `Organizational Unit` named `HR` We create a new user within `HR`.

![Pasted image 20240424222731](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2efbf59c-aea5-44b6-9465-dcaa3ebbbad5)

### Add Windows 10 AKA "Target-PC" to our Domain. 
***

On our Windows 10 AKA "Target-PC" we want to search up `PC` -> `Properties`. 

![Pasted image 20240425150604](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c93b3726-6497-42d4-9199-721fafd7591d)

Scroll down and click on `Advanced system settings`.

![Pasted image 20240425150730](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0f2ef0d8-ca4b-4bd3-a4b9-17260112cc44)

We want to click on the tab `Computer Name`

![Pasted image 20240425150902](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/dd91ea00-bd42-4b44-8542-e25085e01f1a)

Then on the `Change...` button. 

![Pasted image 20240425150927](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/060eca64-bb94-45af-ac7c-ed95a16462e0)

We want to make sure we select `Domain:` 

![Pasted image 20240425151007](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/beed85a3-7ee2-4fb6-a94f-a578f98e9db8)

We type in our `.local` Domain and press `OK`.

![Pasted image 20240425151206](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/fec47cb8-6485-43c5-af03-b145614d78c8)

We see this error.

This is because our "Target-PC" AKA Windows 10 doesn't know how to resolve the `.local` domain. So, we will fix that. 

![Pasted image 20240425151249](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/25144526-8a90-42d0-b1bf-e2c5cfc5655d)

We will right-click our `Network Adapter` ![[Pasted image 20240425151619.png]] on the right hand side of the task bar and click `Open Network & Internet settings`.

![Pasted image 20240425151743](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a45cacab-7a80-44d9-87a8-1e6af82447b3)

We will click on `Change adapter options`

![Pasted image 20240425151831](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a9215f7e-d7fa-4bd6-b053-f5d1b891c9b4)

Right click our adapter and click `Properties`

![Pasted image 20240425151921](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/881283fc-01b2-4a42-8287-4ac15c34a485)

Double click `Internet Protocol Version 4 (TCP/IPv4)`

![Pasted image 20240425152013](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/624f53ab-5ee9-45ab-a10b-41144979c718)

We currently have our `Preferred DNS server:` pointing to Google's DNS and we want to change this to our `Domain Controller`. 

![Pasted image 20240425152113](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/dcd692d1-b115-44a8-b14c-1e240cec5332)
0425152113.png]]

So, we will change that to our `Domain Controller`'s address of `192.168.10.7`.

![Pasted image 20240425152435](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/7df1b138-96e1-40fa-a850-ee6f4d897300)

It's important that we also click `OK` here as well or else it will not save. 

![Pasted image 20240425152559](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b6cf5a05-c5ef-4c5a-ade0-cf15945bf558)

We will open `Command Prompt` and make sure everything is correct by running the command 

```Command Prompt
Ipconfig /all
```

In which, we do see now that our `DNS Servers` is `192.168.10.7`. 

![Pasted image 20240425152917](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a09623f6-0c81-4794-a8fb-549ec73beaf7)

Now we can add "Target-PC" AKA Windows 10 to our Domain without getting an error like before. 

So we go back to this and press `OK`.

NOTE: Be sure to have your Active Directory Server AKA "ADDC01" powered on as well. 

![Pasted image 20240425153231](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/73694e9d-5153-46ed-8e59-ea40f6403cc2)

We see this pop up.

![Pasted image 20240425153539](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/093ea43e-c0d8-4458-ad34-0005145bfaac)

We will use the `Administrator` account of the Active Directory Server because it has the proper permissions to join the domain. 

![Pasted image 20240425153842](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4dd8245c-e797-4ae0-864f-230938ddcf94)

We get this and press `OK`.

NOTE: In a real world environment we would create users and put them into a custom group that are authorized to allow computers to join the domain. 

![Pasted image 20240425154020](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/334790a9-a5d8-4ac7-8300-0cd23b366c69)

We press `Ok` again

![Pasted image 20240425154207](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/8b770ea7-8b75-4c11-8b25-d5ac01808866)

We `Restart Now`

![Pasted image 20240425154247](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/1ad3de7c-74ba-4253-83b1-4592cfb26ad9)

Back on the log in screen. We want to log into our new user account that we made. For me, it was called "Sheldon.Plankton"

![Pasted image 20240425154419](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/fbcdfd20-eee7-479f-882e-0ab0158a2393)

To do this, we will select `Other user` in the bottom left.

![Pasted image 20240425154625](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/542b3772-3ec4-4475-ae5d-ff19216be641)

We want to make sure that we are signing into our domain. 

![Pasted image 20240425154828](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a5be1c68-f8ed-4e65-9730-4e7af81e38e3)

We fill out the login info for our user and sign in. 


![Pasted image 20240425154808](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f978d1f8-7c07-40a4-b740-a41e2bc46ea7)

It begins to log in

![Pasted image 20240425154924](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a94e2a2e-4a19-4d2b-aaac-c712e49a52fa)


### Use Kali Linux to perform a brute force attack
***

dd



















### Setting up Atomic Red Team on our Windows 10 AKA "Target-PC" to generate telemetry and view on Splunk. 
***

dd

[back](./)
