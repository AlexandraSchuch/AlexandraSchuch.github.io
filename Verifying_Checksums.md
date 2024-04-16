# Verifying Checksums of Download Files


## What is a checksum?

A checksum is a unique fixed string that's generated through the combination of a hashing algorithm and data. In this case, a file is hashed to create a checksum. 

![Pasted image 20240416144120](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a0dcd505-fbea-44ca-85eb-78fb29ae57b0)

We can use checksums to verify the integrity of a file and detect errors due to data transmission corruption by comparing two (the original and ours) sets of hashes to see if they match. 

If the file has altered in any way, then the hashes will not match.

![Pasted image 20240416144042](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/e577abc6-80a5-438b-bc74-7dd562a2b959)

***
## Utilizing Checksum

Using VirtualBox as an example

![Pasted image 20240415132701](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/5bd0831b-14f4-487c-8c80-6ca5c662c038)

We Click Download

![Pasted image 20240415132713](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/4712c857-e2f4-4ab8-abb5-7ec4ca285c6f)

We choose the operating system that pertains to us. In my case, it's a Windows OS 

![Pasted image 20240415132740](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/a9baf301-70b5-4e25-b346-a24fb3a36e25)

While its downloading, lets open up the "SHA256 Checksums"

![Pasted image 20240415132805](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/ba6fec09-4631-4707-acca-fc7b1785e1fa)

We see this page, which lists all the SHA256 Checksums for all the different download versions of VirtualBox. We can use the SHA256 Checksum to verify that the VirtualBox file that we just downloaded has not been altered in anyway.

![Pasted image 20240416142359](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/6d8b16c5-6a60-4a6a-9f34-77e01e54cf2f)

We open our file explorer and head over to our downloads directory

![Pasted image 20240415135904](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/cbd5a8a1-f9e4-44a1-8047-51686c19a254)

We then right click the downloads directory and open PowerShell

![Pasted image 20240415141606](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/f7c76147-0895-44f3-b3e1-03539cba18c8)

Now we run the command 
<br>
`Get-FileHash .\(VIRTUALBOX PATH)`
<br> 
In my case its was

<pre><code>Get-FileHash .\VirtualBox-7.0.14-161095-Win.exe <button aria-label="copy" data-title-succeed="Copied!" data-original-title="" title=""><i class="far fa-clipboard"></i> </button></pre></code>

![Pasted image 20240415141902](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/12e85150-2041-49de-b64e-5cd12d621583)

and our results are something like so, where we see the SHA256 of the VirtualBox file.

![Pasted image 20240415142018](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/515dac9d-83e1-4c23-9a61-8dec92b8b9ee)

We will copy the Hash and paste it into the find tool (CTRL + F) on the Checksum web page we opened earlier and see if we get a match between the two SHA256 hashes.

![Pasted image 20240415143129](https://github.com/AlexandraSchuch/alexandraschuch.github.io/assets/144488134/c708e406-57d6-4703-a36f-138000ef47b6)

and WE DO! Look at that! It matches perfectly. We now know that the VirtualBox download hasn't been altered in anyway during transit and it's safe to fully download.
