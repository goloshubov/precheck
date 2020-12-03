adhoc.yml

source_groupname=source (default)
target_groupname=target (default)
source_list=[sourcehostA, sourcehostB, ...]
target_list=[targethostA, targethostB, ...]
tcp_ports=[22,443, ...]
udp_ports=[53, ...]
run_listeners=False
do_cleanup=True
do_curl=False
curl_protocol="https"
curl_opts="-k -m5"
curl_ports=[443]
do_ping=False
```

$ ansible-playbook -i ./hosts precheck.yml -v
$ ansible-playbook -i ./hosts precheck.yml -v -e do_ping=True
$ ansible-playbook -i ./hosts precheck.yml -v -e '{tcp_ports: [80,443]}'
$ ansible-playbook -i ./hosts precheck.yml -v -e do_curl=True
$ ansible-playbook -i ./hosts1 -i ./hosts2 precheck.yml -v -e do_curl=True -e source_groupname=source1 -e target_groupname=target2
$ ansible-playbook -i ./hosts precheck.yml -v -e do_curl=True -e curl_protocol=http -e '{curl_ports: [80]}'
$ ansible-playbook -i ./hosts precheck.yml -v -e '{tcp_ports: [53,123]}' -e '{udp_ports: [53,123]}' -e do_curl=True -e curl_protocol=http -e '{curl_ports: [80,8080]}'
$ ansible-playbook -i ./hosts precheck.yml -v -b -e '{tcp_ports: [53,123]}' -e run_listeners=True
$ ansible-playbook precheck.yml -i './hosts' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22,53]}'
$ ansible-playbook precheck.yml -i '' -e '{source_list: [localhost]}' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22,53]}'
$ ansible-playbook precheck.yml -i ./hosts -e '{source_list: [sourcehostA, sourcehostB]}' -e '{target_list: [192.168.1.100,192.168.2.200]}' -e '{tcp_ports: [22]}'


$ cat /path/targetdata
192.168.1.1,192.168.1.2,192.168.1.3;123,53,8080
192.168.1.4,192.168.1.5;9090,10050
192.168.1.6;22

$ cat targetdata | while IFS=';' read -r hosts ports; do ansible-playbook -i ./sourcehosts precheck.yml -e "{target_list: [ $hosts ]}" -e "{tcp_ports: [ $ports ]}"; done
