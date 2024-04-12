How can we know what is bad running processes on our Window systems if we don't recognize what is good running processes?

Today we explore the world of *core processes within a Windows system.* with the "Core Windows Processes" module on TryHackMe. 

![OGC](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5df56f45-ba97-4ea8-9ac8-4a1e434b7606)

Fun fact: The Windows operating system is the most used in the world. Although i'm sure you could have guessed this. 

However... most of its users don't fully understand the interworkings of the system. As long as it makes it on the ole facebook or Youtubes, then thats enough right? WRONG!

We must be BETTER! Be security savvy! Remember, when in doubt, defense in depth out!

Long ago in a galaxy far far away... it used to be enough to just have antivirus protection and call it a day. Then bad actors got smarter and began disguising malware to look like innocent window processes and free non-sus game downloads. Suspicious binary and processes became harder to detect. That's why its important to use a multi-layer approach. If one security solution doesn't catch it then maybe another one will. 

Lets talk about our friend, task manager. A lot of people already know the existence of task manager, which is a staple Windows utility that allows users to visually see what is currently running on their Windows system through a GUI (Graphic User Interface). Not only this, but it can also show users what programs aren't responding and allow them to end the process or see details on how much a CPU a process is utilizing. 

Task Manager is a useful tool when troubleshooting or performing analysis on an endpoint device. However, it does have its drawbacks such as not showing a Parent-Child process view. 
![Pasted image 20240412183916](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a148fbec-e395-448c-a04f-08a2c8bdd2e2)

![Pasted image 20240412184248](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/75b375b3-d34f-4540-b841-48fc30ac594c)

For example, System (Parent) to smss.exe (Child) 
You can't tell the relationship through Task manager but you can through other tools such as Process Hacker and Process Explorer.
![Pasted image 20240412184019](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/437bd7da-ae31-464e-8e85-231194cfa3ef)

![Pasted image 20240412184022](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/707a3a51-a22e-43bf-99a7-47e812927528)

*** 
## SYSTEM 

This is the first Windows process I will talk about, "System." The PID (process Identifier) for System is **ALWAYS** 4. 

The official definition from Windows Internals 6th Edition: "_The System process (process ID 4) is the home for a special kind of thread that runs only in kernel mode a kernel-mode system thread. System threads have all the attributes and contexts of regular user-mode threads (such as a hardware context, priority, and so on) but are different in that they run only in kernel-mode executing code loaded in system space, whether that is in Ntoskrnl.exe or in any other loaded device driver. In addition, system threads don't have a user process address space and hence must allocate any dynamic storage from operating system memory heaps, such as a paged or nonpaged pool._"

Here we have the "properties" of the "System" process through Process Explorer

![Pasted image 20240412182840](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/36e19bb2-ada5-4a48-aea6-a0515c6f1f83)

As you can see, it doesn't give us much. In contrast, if we look at the "Properties" of the "System" process through Process Hacker we get this.

![Pasted image 20240412184909](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4a9a71f1-9072-4f54-bbc2-35f09e44c3b3)

which gives us more information. We now can see that the Parent of System is System Idle Process (0), the image path is C:\Windows\system32\ntoskrnl.exe (NT OS Kernel), and it also show us this is verified to be Microsoft Windows.

Some unusual behavior we want to look out for when it comes to the "System" process would be things such:

- A parent process (aside from System Idle Process (0))
- Multiple instances of System. (Should only be one instance) 
- A different PID. (Remember that the PID will always be PID 4)
- Not running in Session 0
***
