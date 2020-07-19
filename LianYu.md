# Try Hack Me - Lian_Yu
A beginner level security challenge

---
## Objectives
1. Deploy the VM and Start the Enumeration.  
2. What is the Web Directory you found?  
3. What is the file name you found?  
4. What is the FTP Password?  
5. What is the file name with SSH password?  
6. user.txt  
7. root.txt  
---
## Recon, Scanning, and Enumeration
Once the VM has been launched I am going to start scanning the box.  
Command: `nmap -T4 -sC -sV -oN nmap\scan.txt <ipaddress>`
Command Breakdown:
```
	-T4: speed of nmap scan is 4/5 (personal preference of mine)

    -sC: Scan with default NSE scripts. Considered useful for discovery and safe

	-sV: Attempts to determine the version of the service running on port

    -oN: Normal output to the file normal.file
```
NMAP Results:
```NMAP
Host is up (0.12s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38570/tcp6  status
|   100024  1          42067/tcp   status
|   100024  1          53744/udp   status
|_  100024  1          59897/udp6  status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Based on our NMAP results we are able to see that thereis a web service that is open. So lets the IP address of our box and put it into our browswer and see what comes up.  

![homePage](images/homepage.png "homePage")  
<br>
The homepage that we are precented with gives us a little back store on who Oliver Queen but that is about it.  
So lets check out the source code of this page and see if we are able to find any additional clues.  
Their doesn's seem to be anything usefull on this page.
![homePageSourceCode](images/homePageSourceCode.png "homePageSourceCode")  

How about we fire up GoBuster and see if there is anything creeping below the surface of this site.
Command: `gobuster dir -u http://<ip> -t 40 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`

Command Breakdown:
```
    dir: Specify that you want to search for dictories.

    -u: Full URL (including scheme), or base domain name.

    -t: Declare the amount of threds you want to use.

	-w: Path to the wordlist used for brute forcing (use – for stdin).
```
![gbIsland](images/gbIsland.png "gbIsland")
It seems that where able to locat another directory. So lets go back to our broswer and see if we are able to navagate to it.  

![island](images/island.png "island")  
The `island` urls is accessible by our browser and it seem that they where not expecting us.  
It seems that our next hit is to got to Lian_Yu and we are going to need a code word in order to get in. Let's check out the source code to see it we can find this code word.  
I think that we have found our code word and it looks like the tried to hit it plan site. Cleaver!
![islandSouceCode](images/islandSouceCode.png "islandSouceCode")  

Let's take a look at the hit that TryHackMe gives us.  
![Hint2](images/Hint2.png "Hint2")  
Our hit is "In numbers". After digging around on the internet a bit I was able to find the  [SecLists](https://github.com/danielmiessler/SecLists) repo maintained by Daniel Miessler, Jason Haddix, and g0tmi1k.  
So I quickly did a git clone of the repository and decided to use one of the Fuzzing word list in combnation with GoBuster.  
Commad: `gobuster dir -u http://$ip/island -t 40 -w /usr/share/dirbuster/wordlists/SecLists/Fuzzing/4-digits-0000-9999.txt` 
<br>

![2100](images/2100.png "2100")  
It seems that there is an addittional directory.  
Now we are going to navgate to that site and see if we can find the next clue.  
It would seem that the only thing that we have on this site is just a YouTube clip explaning more about Oliver Queen.  

<br>

![ytClip](images/ytClip.png "ytClip")  
Just like the previous sties that we had found we are going to check the source code and see what we are able to find. 

![ticket](images/ticket.png "ticket")  
At first glance it does not seem like anything in this codes is out of place but the more I looked at it the more the comment seemed out of place.  
So I went back to our dear friend GoBuster and had it checked for `.ticket` extentions.  
Command: `gobuster dir -u http://$ip/island/2100 -T 40 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x .ticket`  
![gbTicket](images/gbTicket.png "gbTicket")  
<br>
When we go to open up the `green_arrow.ticket` in our browser we are given what looks like it could be an encoded password.  
![greenArrowURL](images/greenArrowURL.png "greenArrowURL")  
Now if I had to guess I would say that this some form a base encoded string. Let's head over to [CyberCheft](https://github.com/danielmiessler/SecLists) and see kind of resutls we will get when we run it though each base version.  
<br>
The only one that I was able to any readiable results from was `Base58`.  
It is possible that we now have a username and password combonation.. but to what it the exact question.
![decodedPass](images/decodedPass.png "decodedPass")  

If we look back at our `nmap` scans that when did at the begining of this challene, remember that we had both an FTP and an SSH port open.  
Port 21 our FTP port  
Port 22 is our SSH port  
## Compromise
---
So we are going to use that username and password that we found and try to log into each port with them. Starting with the FTP.  
Command: `ftp <ip address>`
![ftp](images/ftp.png "ftp")  

Looks like we are able to log into FTP with the credentials that we found earlier.
Now if we use our `ls` command we will be able to see if there is anything that it being stored around here.  
Seems like we have a couplet of images that are just laying around. Let's go ahead and [copy](https://kb.globalscape.com/KnowledgebaseArticle10407.aspx) them ot our local computer so can check them out later.  
Command: `mget *.*`  

No that our images are downloaded lets see if their is anything use around.  
Let's go up a directory using the `cd ..` commade.  
It seem that there are more users on this server then just the one that we had found.  
I tried to change into that directory but was unsuccessful, so we will return to it later.  
![slade](images/slade.png "slade")  

We are now going to take a look at the images that we downloaded while logged into the FTP server.  
The first image that we are going to look into is `Leave_me_alone.png`.  
When we use the command of `file Leave_me_alone.png` you can see that the results come back and say that the file type is data and not png.  
![fileLeave](images/fileLeave.png "fileLeave")  

This leads me to believe that there could be something wrong with this file.  
We can use  hexeditor to view the header information of out image and see if it is correct.  
Here is a nice [cheat sheet](https://digital-forensics.sans.org/media/hex_file_and_regex_cheat_sheet.pdf) that breaks down what the header should be for an png image.  

If you compare the header of the image to the cheet sheet you can see that the header for this png file is not correct.  
So we are going to changed it to `89 50 4E 47`, save the changes and then try and open the image.  
![hex](images/hex.png "hex")  
 Once we try and open up the `Leave_me_alone.png` image you will need that it actually opens up this time around.

![zipPass](images/zipPass.png "zipPass")  
Since this image has so kindly provided us with a password we need to checked out the other imaged and see it they are hidding anything as well.  
Command: `steghide info aa.jpg`  
![steghideInfo](images/steghideInfo.png "steghideInfo")  
Now that we have extracted the zip file from our image lets go ahead and unzip that file as well.  
Command: `unzip ss.zip`  

It looks like two files where extracted. Lets cat out the content of each file and see what awaits us.
![passwd.txt](images/passwd.txt.png "passwd.txt")  
Well this does not look like a password at all but it does provide us with a nice little warning about what waits for us when we finally get to Lian Yu.  

This time we are going to take a look at our next file.  
It seems that the content of this file could be yet another password.  
So we have a password but what it the username to go along with it an most importantly do we need to log into.  

![shado](images/shado.png "shado")  

There is only one other port avaiable that we could try and log into this time around. We are going to try and log into our SSH port this time.  
Command: `ssh <username>@<ipaddress>`

At this moment in the challenge it helps if you know anything about the TV show Arrow. I was able to pull on my knowlege of the show and figure out the username for SSH by a litte bit of luck and some well placed hints.  
The first hint would be the name of the other directory that we saw when logged into under the FTP account.  
Our second hing would have been the image that we had to correct the header on.  
So I toke the name of that character and used it in our SSH command and combined it with the content of the shado file and then ... BOOM!!  
![LianYu](images/LianYu.png "LianYu")  
As soon as I got logged in I used the `ls` command and I was given out `user.txt` file and then I used the `cat` command to receive the content of that file as well.  
Just like that we were able to find our user flage.  
![user.txt](images/user.txt.png "user.txt")  
## Privilege escalation
---

Now that we have found our user flag the next thing that we will need to do is try and find a to escalation our privileges from the slade user to a root user.  
Lets see what we are able to run on this box that does not require the root password.  
Command: `Sudo -l`  
![sudo](images/sudo.png "sudo")  
It looks like we are able to run the `pkexec` file without being prompted for a root password.  
While researching possible ways to exploite pkexec I came across an article [here]("https://unix.stackexchange.com/questions/421853/one-prompt-pkexec-two-command") and it gave an an idea to see if I could call a bash shell from pkexec.  
Command: `sudo /usr/bin/pkexec /bin/bash`  
![rootShell](images/rootShell.png "rootShell")  
<br>
I was able to successfully gain a root shell and now we need to find out `root.txt` file.
So lets poke around in our root directory and see if we are able to find it.  
Using our `ls /root` command we are able to find the root.txt file.  
Now we will need to receive the content of that file.  
![root](images/root.png "root")  
<br>

# Congratulations you have just completed the Lian Yu challenge!