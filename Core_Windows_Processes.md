How can we know what is bad running processes on our Windows systems if we don't recognize good running processes?

Today, we explore the glorious world of *core processes within a Windows system.* with the "Core Windows Processes" module on TryHackMe. 

![OGC](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5df56f45-ba97-4ea8-9ac8-4a1e434b7606)

Fun fact: The Windows operating system is the most used system in the world. Although I'm sure you could have guessed that. 

However... most of its users don't fully understand the interworkings of the system. As long as it makes it on the ole Facebook or Youtubes, that's enough right? WRONG!

We must be BETTER! Be security savvy! Remember, when in doubt, defense in depth out!

Long ago in a galaxy far far away... it used to be enough to have just antivirus protection and call it a day. Then the bad actors became smarter. They began disguising malware to appear as innocent window processes and free non-sus (sarcasm here) game downloads. Suspicious binary and processes became challenging to detect. That's why it's essential to use a multi-layer approach. If one security solution doesn't catch it then maybe another one will. That's for another day though... 

Let's now go back to our original question, "How can we know what is bad running processes on our Windows systems if we don't recognize good running processes?"

You'll soon learn that Windows has a lot of processes.
![Pasted image 20240413204215](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/394f3090-1b2d-41a8-919a-e927ab190e7b)

Here is a rundown of the hierarchy of Windows Core Processes. I will cover all of these, starting from the top with "system" and working my way down the chain.

![Pasted image 20240414200054](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f0a1d2a7-9b36-400f-983b-c86748f0d074)

If you weren't already wondering. It's important to know the meaning behind both session 0 and session 1.

Session 0 represents the kernel mode session vs. session 1 represents the user mode session. You'll notice smss.exe has both, which we will cover later. 

To better understand kernel mode vs user mode, here are some resources 

[User mode vs Kernel mode in operating system (youtube.com)](https://www.youtube.com/watch?v=KOC3CZNjyJ4)

[What’s the Difference Between User and Kernel Modes?](https://www.baeldung.com/cs/user-kernel-modes)

[Architecture of Windows NT - Wikipedia](https://en.wikipedia.org/wiki/Architecture_of_Windows_NT)

Now let's talk about our friend, task manager. Many people already know the existence of Task Manager, a staple Windows utility that allows users to visually see what is currently running on their Windows system through a GUI (Graphic User Interface). It can also show users what programs aren't responding and allow them to end the process or see details on how much CPU a process is utilizes. 

Task Manager is a useful tool when troubleshooting or performing analysis on an endpoint device. However, it does have its drawbacks, such as not showing a Parent-Child process view. 
![Pasted image 20240412183916](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a148fbec-e395-448c-a04f-08a2c8bdd2e2)

![Pasted image 20240412184248](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/75b375b3-d34f-4540-b841-48fc30ac594c)

For example, System (Parent) to smss.exe (Child) or Services.exe (Parent) to svchost.exe (Child)

You can't tell the relationship through Task manager, but you can through other tools such as Process Hacker and Process Explorer.
![Pasted image 20240412184019](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/437bd7da-ae31-464e-8e85-231194cfa3ef)

![Pasted image 20240412184022](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/707a3a51-a22e-43bf-99a7-47e812927528)

*** 
## SYSTEM 
![Pasted image 20240413164609](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/b21e0ebe-8998-4be3-bd96-a1f848bd5633)

This is the first Windows process, "System." A pretty important one because its the start of it all. 

- The PID (process Identifier) for System is **ALWAYS** 4. 
- Only runs in Kernel mode
- The system process represents the running state of the *ntoskrnl.exe* in memory
- Parent is *System Idle Process (0)*


The official definition from Windows Internals 6th Edition: "_The System process (process ID 4) is the home for a special kind of thread that runs only in kernel mode a kernel-mode system thread. System threads have all the attributes and contexts of regular user-mode threads (such as a hardware context, priority, and so on) but are different in that they run only in kernel-mode executing code loaded in system space, whether that is in Ntoskrnl.exe or in any other loaded device driver. In addition, system threads don't have a user process address space and hence must allocate any dynamic storage from operating system memory heaps, such as a paged or nonpaged pool._"

Here we have the "properties" of the "System" process through Process Explorer

![Pasted image 20240412182840](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/36e19bb2-ada5-4a48-aea6-a0515c6f1f83)

As you can see, it doesn't give us much. In contrast, if we look at the "Properties" of the "System" process through Process Hacker, we get this.

![Pasted image 20240412184909](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4a9a71f1-9072-4f54-bbc2-35f09e44c3b3)

which gives us more information. We now can see that the Parent of System is System Idle Process (0), the image path is C:\Windows\system32\ntoskrnl.exe (NT OS Kernel), and it also show us this is verified to be Microsoft Windows.

Some unusual behavior we want to look out for when it comes to the "System" process would be things such:

- A parent process (aside from System Idle Process (0))
- Multiple instances of System. (Should only be one instance) 
- A different PID. (Remember that the PID will always be PID 4)
- Not running in Session 0

***
## smss.exe (Sesson Manager Subsystem)

![Pasted image 20240413164746](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3b40ed0b-ea33-4630-8420-392fc1fa4d5d)

If you're still reading, then our second Windows process I will talk about is "smss.exe" aka. the Windows Session Manager aka. Session Manager Subsystem Process. 

  - Parent is *System*
  - Starts in kernel mode
  - Image Path: %SystemRoot%\System32\smss.exe
  - creates a lot of child processes 

 As the name would imply, *Windows Session Manager (smss.exe)* is responsible for... well creating new sessions. Wow amazing stuff! 
 
![tenor](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e43955c0-96d9-458e-ac0f-2caa60c20fe1)

Anyways... this one is not so obvious. It's the first user mode process started by the kernel during the boot up process. 

![Pasted image 20240413171940](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9765ff7b-3e44-49b6-b3cd-6a455479569f)

smss.exe starts *csrss.exe* and *wininit.exe* in session 0

![Pasted image 20240413181912](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5c634157-5612-4dff-b0b5-4fc6d91350d8)

 and another csrss.exe and winlogon.exe for Session 1

![Pasted image 20240413185858](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/cb3c8183-b1d2-4c05-914d-e7414b828766)

*smss.exe* is also responsible for:

  - launching any other subsystem lited in the required value of  ![Pasted image 20240413190503](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ee570254-436e-4bb8-a741-abc742066c0c)

![Pasted image 20240413190624](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/da16fdc4-2217-423b-aadd-bf7a785db9fb)
  - creating environment variables
  - creating virual memory paging files

You'll notice something interesting about *smss.exe*, when it spawns a new process, it then self-terminates. This is why, when you check the parent of *csrss.exe* or others, it will show "Non-existent process" even though we know *smss.exe* is the parent. 
![Pasted image 20240413191942](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/8328148b-497a-466d-b187-23f44857026e)

It would be sus to see anything other than "Non-existent process" and should be investigated if so. 

Some unusual behavior we want to look out for when it comes to the "smss.exe" process would be things such:

- A different parent process other than System (4)
- The image path is different from C:\Windows\System32
- More than one running process. (children self-terminate and exit after each new session)
- The running User is not the SYSTEM user
- Unexpected registry entries for Subsystem

***
## csrss.exe (Client Server Runtime Process)

![Pasted image 20240413194640](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0b1c1e26-c637-465a-862d-3b8efb1bd7fd)

If you're still with me, then you've made it to the third process I will be talking about, and that is *csrss.exe*

- There will be one _Session 0_ and _Session 1_ csrss.exe processes. That is why you see two of this process. Their PIDs are different.
- - Image path %SystemRoot%\System32\csrss.exe
- Always running and is critical to system operation
- If terminated, will result in system failure
- Parent will be "non-existing process" (even though we know that it is spawned from smss.exe)

![Pasted image 20240413201554](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/85e093d8-4134-49b6-8219-04a0f58adefb)

The *csrss.exe* process is also responsible for:

  - making the Windows API available to other processes
  - mapping drive letters
  - handling the Windows shutdown process
  - Win32 console window and process thread creation/deletion
  - For each instance, csrsrv.dll, basesrv.dll, and winsrv.dll are loaded (along with others).

Some unusual behavior we want to look out for when it comes to the "csrss.exe" process would be things such:

- An actual parent process. (smss.exe calls this process and self-terminates)
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue processes masquerading as csrss.exe in plain sight
- The user is not the SYSTEM user.

***
## wininit.exe (Windows Initialization Process)

![Pasted image 20240413195009](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/6e1ce9ed-ae0d-46ef-bb90-e06a4aba2d03)

...and we continue with the fourth process I will be talking about, and that is *wininit.exe*

-  Image Path: %SystemRoot%\System32\wininit.exe
- Parent will be "non-existing process" (even though we know that it is spawned from smss.exe)
- There will only be one instance of this process

*wininit.exe* launches three child process within session 0, which are *services.exe* (Service Control Manager), *lsass.exe* (Local Security Authority), and *lsaiso.exe*.

![Pasted image 20240413200435](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9b58db5c-380c-45f5-a417-e843c75a8721)

Notice you don't see *lsaiso.exe*, this is because this process is associated with **Credential Guard and KeyGuard**. You will only see this process if Credential Guard is enabled.

![Pasted image 20240413200656](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/20e9f771-894b-4916-a65d-6a9624bb7d52)

Some unusual behavior we want to look out for when it comes to the "wininit.exe" process would be things such:

- An actual parent process. (smss.exe calls this process and self-terminates)
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue processes in plain sight
- Multiple running instances
- Not running as SYSTEM

BONUS- BSOD (Blue Screen of Death) demo 
[Power shell + wininit = BSOD (youtube.com)](https://www.youtube.com/watch?v=aYFvidT4o7g)

***
## services.exe ( SCM Service Control Manager)

![Pasted image 20240413201203](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/68bb8b03-a3ee-40e7-80cb-34283300b0dd)

Alright! Time to talk about the fifth process, *services.exe*

- Image Path: %SystemRoot%\System32\services.exe
- Parent is always *wininit.exe*
- Only one instance

*services.exe* primary job is to... handle services. Did you guess that one this time? 

![raw](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/1589c35b-0146-406c-81e8-88b52e08c79d)

Anyways, it loads, interacts, starts, and ends services.

It also maintains a database that can be queried by a OS native program called *sc.exe*

![image](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/406e8cfc-3d55-4dd7-b361-ad70c2d4b6f3)

*services.exe* has many child processes such as *svchost.exe*, *spoolsv.exe*, *msmpeng.exe*, and more. 

![Pasted image 20240414202902](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/0f27b362-5c36-4096-ac67-9177b8c21b11)

And service.exe was like...

![Pasted image 20240413205056](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a11ba94e-5b64-4723-a8ae-987aa6925111)

Some unusual behavior we want to look out for when it comes to the "services.exe" process would be things such:

- A parent process other than wininit.exe
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue processes in plain sight
- Multiple running instances
- Not running as **SYSTEM**

***

## svchost.exe (Host Process for Windows Services)

![Pasted image 20240413211757](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/93cde02d-f308-4041-8e05-dc913842c7eb)

We're more than halfway there! Now I will cover the sixth process, *svchost.exe*

  - aka Service Host
  - **Image Path**: %SystemRoot%\System32\svchost.exe
  - Will have many instances
  - Parent is *services.exe*
  - Where the DLL processes are stored

So here's something you probably don't know but service host (*svchost.exe*).... hosts services. yayyy! 

![dogecoin-gif-3](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/84dfdf27-948a-48ca-98a1-d0b4a3059415)

...and it manages them too! 

![[Pasted image 20240414125309.png]]

Services running in this process are implemented as DLLS and stored in the registry for the service under the "Parameters" subkey in "ServiceDLL."
![Pasted image 20240414133036](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ef7b834d-2b4a-49c2-9251-5b64b37bb855)


Here's an example showing the "Service DLL" value for "Dcomlaunch" service.

![Pasted image 20240414133217](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d84fac0a-aa65-4d89-852f-e79df96a77b5)

If using "Process Hacker," you can access this info. by right clicking the *svchost.exe* process. 

![Pasted image 20240414133320](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/3eddcd0d-e709-4ff0-992b-a6ecdbf22506)

then you right click the service column and select properties. Now you see the "Service DLL."

![Pasted image 20240414133517](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/11b91e84-6661-46ff-80d6-c9e184185d27)

Now look at the binary Path in the above screenshot. After where you see *svchost.exe*, you'll notice "-k." This is important because it's a key identifier and groups similar services to share the same process. We want to see the "-k" when inspecting and determining if something is a legitimate *svchost.exe* process. If we don't see it, well....then it's SUS because this is an indicator of compromise.

![6dx215](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ea6994bd-afa1-47cd-b3a8-d9371aee176d)

Here we can see multiple services running with the same binary path as the "Dcomlaunch." All with the "-k" identifier.

![Pasted image 20240414134424](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e5ec69f4-1f36-4ec5-a7da-cb9fdafa7ca0)

However, where these services differ is in their "Service DLL" values. 

For example, let's look at the properties of "LSM" and compare it with the properties of "Dcomlaunch."

LSM 

![Pasted image 20240414135554](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/34d3243a-4518-4353-bc4c-0377162fe7f9)

DCOM

![Pasted image 20240414135615](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c5714268-3403-4286-9f89-505f9c9e6bd1)

Notice the binary paths are the same, but the "Service DLL's" are different. 

It's important to note that *scvhost.exe* is a target for adversaries to hide malware on Windows systems. Because it is normal for *svchost.exe* to have multiple running processes, they can use this to their advantage to hide a fake *svchost.exe* (malware) process amongst legitimate *svchost.exe* processes and name it something slightly different like "scvhost.exe."

Some unusual behavior we want to look out for when it comes to the "svchost.exe" process would be things such:

- A parent process other than services.exe
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue processes in plain sight
- The absence of the -k parameter

***

## lsass.exe (Local Security Authority Subsystem Service)

![Pasted image 20240414152200](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/71044379-f99e-4bc4-bb23-e6c267395897)

We're getting to the end! Time to talk about the seventh process, *lsass.exe*

  - Parent is **always** wininit.exe
  - **Image Path**:  %SystemRoot%\System32\lsass.exe
  - One instance

*lsass.exe* is the enforcer! 

![avatar-clint-eastwood-www gifea com](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/9d524dd6-b2ff-49bb-a74c-bba9ea4b5745)

It enforces security policies on Windows systems. 

It does this by verifying user log ins on the system or server, handling password changes, and creating access tokens. It also writes to the Windows Security Log. 

*lsass.exe* creates security tokens for:
  - SAM (Security Account Manager)
  - AD (Active Directory)
  - NETLOGON

![OGC](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4d3c2db7-23bb-439e-9f81-50f35e35be9f)

However with great power comes a great amount of adversaries looking to credential dump or create a fake version of the process. *lsass.exe* is targeted often by adversaries. 

Some unusual behavior we want to look out for when it comes to the "lsass.exe" process would be things such:

- A parent process other than wininit.exe
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue processes in plain sight
- Multiple running instances
- Not running as SYSTEM

***

## winlogon.exe (Windows Logon)

![Pasted image 20240414175942](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4c00e490-4ca3-47b9-b601-1b95cfabfa93)

Look at that! We're on the other side with session 1! Now we cover the eighth process, *winlogon.exe*. 

![Pasted image 20240414173204](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ff9e37c0-1dcb-473a-85b5-079ef0775c60)

  - Can have one or more instances when new sessions are created (usually through Remote Desktop or Fast User Switching logons)
  - Parent is *smss.exe* so you should see "non-existing process" because *smss.exe* self-terminates
  - **Image Path**:  %SystemRoot%\System32\winlogon.exe

*winlogon.exe* is responsible for handling the Secure Attention Sequence (SAS). SAS is better known as the Ctrl+Alt+Delete combo and serves a a safe checkpoint, in which any actions after invoking it will be genuine and secure.

![Pasted image 20240414173707](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/bd089a1d-3c50-4125-ac2a-5587a6ff894c)

You can read more about it here:[Secure Attention Sequence (SAS) - NETWORK ENCYCLOPEDIA](https://networkencyclopedia.com/secure-attention-sequence-sas/)

*winlogon.exe* is also responsible for loading user profiles but also locking the screen and running the user's screensaver. 

![Pasted image 20240414175844](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/956e2c5f-310b-46b9-a848-a906823c6019)

It loads user's profiles by loading the user's "NTUSER.DAT" into HKCU, and userinit.exe then loads the user's shell.

![Pasted image 20240414174746](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/36fea8ad-6fcb-4756-a75b-1b14f8fbdf2a)

Some unusual behavior we want to look out for when it comes to the "winlogon.exe" process would be things such:

- An actual parent process. (smss.exe calls this process and self-terminates)
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue processes in plain sight
- Not running as SYSTEM
- Shell value in the registry other than explorer.exe

***

## explorer.exe (Windows Explorer)

![Pasted image 20240414180030](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/2f8cca45-7d42-489b-a929-160b4bb9002e)

HOORAYY!! It's the last process! This is number nine, with the *explorer.exe* process.

  - Parent is *userinit.exe* but like *smss.exe* it terminates its self after spawning *explorer.exe* so you'll see "non-existing process."
  - **Image Path**:  %SystemRoot%\explorer.exe
  - Can have one than one instances depending on interactively logged-in users. 

The *explorer.exe* process gives users access to their folders and files but also provides functionality for features like the Start Menu and Taskbar. 

Expect *explorer.exe* to have many child processes

![Pasted image 20240414181426](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/066a8514-d3e3-460b-89cd-a264f5dfcbe8)

Some unusual behavior we want to look out for when it comes to the "explorer.exe" process would be things such:

- An actual parent process. (userinit.exe calls this process and exits)
- Image file path other than C:\Windows
- Running as an unknown user
- Subtle misspellings to hide rogue processes in plain sight
- Outbound TCP/IP connections

***
## Conclusion

![raw 1](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/d147a997-b55a-48a8-9705-a20bc519711f)

Congrats! You made it to the end! I know. I know. It was A LOT! However, I hope you learned a thing or two. It must have been interesting enough because you made it here.

Personally, I found learning about the "Core Windows Processes" to be really interesting! That's most likely why I wrote so much. I feel a lot more comfortable opening up Task Manager and having a good idea of what's normal and what is not. I now know some of the indicators of compromise to look for, such as missing the "-k" identifier in the binary path of *svchost.exe* processes or looking for slightly misspelled core processes that is malware trying to disguise themselves.  

Hopefully, now you feel more comfortable with the question, How can we know what is bad running processes on our Windows systems if we don't recognize good running processes?
