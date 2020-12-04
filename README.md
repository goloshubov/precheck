Параметры:

```ini
source_groupname=source (default)
target_groupname=target (default)
source_list=[sourcehostA, sourcehostB, ...]
target_list=[targethostA, targethostB, ...]
do_ping=True  (default)
tcp_ports=[22,443, ...]
udp_ports=[53, ...]
do_nmap_udp=False
run_listeners=False
do_cleanup=True  (True by default when run_listeners=True)
curl_protocol="http" (default)
curl_opts="-k -m5" (default)
curl_ports=[80,...]
negative=False 
```

Примеры:
\
A. роли в плейбуках - adhoc.yml)
\
B. плейбук precheck.yml:
```bash
$ ansible-playbook -i ./hosts precheck.yml -v
$ ansible-playbook -i ./hosts precheck.yml -v -e '{tcp_ports: [80,443]}'
$ ansible-playbook -i ./hosts precheck.yml -v -e '{curl_ports: [80]}'
$ ansible-playbook -i ./hosts1 -i ./hosts2 precheck.yml -v -e source_groupname=source1 -e target_groupname=target2 -e '{curl_ports: [80]}'
$ ansible-playbook -i ./hosts precheck.yml -v -e curl_protocol=https -e '{curl_ports: [443]}'
$ ansible-playbook -i ./hosts precheck.yml -v -e '{tcp_ports: [53,123]}' -e '{udp_ports: [53,123]}' -e curl_protocol=https -e '{curl_ports: [8443,8123]}'
$ ansible-playbook -i ./hosts precheck.yml -v -b -e '{tcp_ports: [53,123]}' -e run_listeners=True
$ ansible-playbook precheck.yml -i './hosts' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22,53]}'
$ ansible-playbook precheck.yml -i '' -e '{source_list: [localhost]}' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22,53]}'
$ ansible-playbook precheck.yml -i ./hosts -e '{source_list: [sourcehostA, sourcehostB]}' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22]}'

$ cat /path/targetdata
192.168.1.1,192.168.1.2,192.168.1.3;123,53,8080
192.168.1.4,192.168.1.5;9090,10050
192.168.1.6;22
$ cat targetdata | while IFS=';' read -r hosts ports; do ansible-playbook -i ./sourcehosts precheck.yml -e "{target_list: [ $hosts ]}" -e "{tcp_ports: [ $ports ]}"; done
```
