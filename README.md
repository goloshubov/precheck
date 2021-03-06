Precheck helps with bulk ICMP/TCP/UDP/HTTP(S) availability checks. For example, it can be used as a predeploy / prerequisite check, to see if all necessary firewall rules were set and communication is possible.

How it works. Each host from 'source' inventory group tries communicating with each host from 'target' inventory group using ping (ICMP), netcat (TCP/UDP), nmap (UDP), curl (HTTP/HTTPS). Some cases without using inventory are also possible - in examples below.






```ini
Variables (options):
#source and target hosts. can be defined through inventory groups
source_groupname=source (default)
target_groupname=target (default)
#or host lists (or combination of both) - IP addresses and/or domain names.   (command line option: -e '{target_list: [host1,host2,...]}' )
source_list=[sourcehostA, sourcehostB, ...]
target_list=[targethostA, targethostB, ...]
#Notice that netcat listeners (run_listeners) can't be used when target_list is defined.
do_ping=True  (default)
#tcp udp (command line option: -e '{tcp_ports: [port1,port2,...]}' )
tcp_ports=[22,443, ...]
udp_ports=[53, ...]
do_nmap_udp=False
#listeners
run_listeners=False
do_cleanup=True  (True by default when run_listeners=True)
#curl
curl_protocol="http" (default)
curl_opts="-k -m5" (default)
curl_ports=[80,...]
negative=False
```

Usage:
\
A. roles in playbooks (example - adhoc.yml)
\
B. precheck.yml playbook:

Define 'source' and 'target' groups, and all needed variables ([all:vars] section) in the inventory file and run the play (ICMP ping check is enabled by default):
```bash
$ ansible-playbook -i ./hosts precheck.yml -v
```
Or use extra variables instead, or combination of both:
- tcp port check, TCP 80,443
```bash
$ ansible-playbook -i ./hosts precheck.yml -v -e '{tcp_ports: [80,443]}'
```
- curl check, http (80)
```bash
$ ansible-playbook -i ./hosts precheck.yml -v -e '{curl_ports: [80]}'
```
- the same as above but with custom host groups (and from different inventories)
```bash
$ ansible-playbook -i ./hosts1 -i ./hosts2 precheck.yml -v -e source_groupname=source1 -e target_groupname=target2 -e '{curl_ports: [80]}'
```
- curl check, https (443)
```bash
$ ansible-playbook -i ./hosts precheck.yml -v -e curl_protocol=https -e '{curl_ports: [443]}'
```
- a combination
```bash
$ ansible-playbook -i ./hosts precheck.yml -v -e '{tcp_ports: [53,123]}' -e '{udp_ports: [53,123]}' -e curl_protocol=https -e '{curl_ports: [8443,8123]}'
```
- tcp port check with running listeners (nc -l) on target hosts (ssh access on target hosts is required)\
```bash
$ ansible-playbook -i ./hosts precheck.yml -v -b -e '{tcp_ports: [53,123]}' -e run_listeners=True
```
- no target group in inventory ('source' group -> target hosts from command line).
```bash
$ ansible-playbook precheck.yml -i './hosts' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22,53]}'
```
- no inventory at all, local run (localhost -> target hosts)
```bash
$ ansible-playbook precheck.yml -i '' -e '{source_list: [localhost]}' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22,53]}'
```
- using only specific source hosts from inventory and custom target hosts from command line (source hosts must be defined in inventory, source group doesn't matter). A sort of host limit for source hosts.
```bash
$ ansible-playbook precheck.yml -i ./hosts -e '{source_list: [sourcehostA, sourcehostB]}' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22]}'
```
- batch run (in the example below it is a list of target hosts with specific ports)
```bash
$ cat /path/targetdata
192.168.1.1,192.168.1.2,192.168.1.3;123,53,8080
192.168.1.4,192.168.1.5;9090,10050
192.168.1.6;22
```
```bash
$ cat targetdata | while IFS=';' read -r hosts ports; do ansible-playbook -i ./sourcehosts precheck.yml -e "{target_list: [ $hosts ]}" -e "{tcp_ports: [ $ports ]}"; done
```
\
To make plays output more readable add display_skipped_hosts = no into the default section of ansible.cfg

