# kioptrix-full-walkthrough
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
