## Initial enumeration

### Identifying hosts
When we are inside a network, there are several ways we can identify hosts on that network. We can `PASSIVELY` identify hosts using network monitoring tools such as `wireshark` and `tcpdump`, then `ACTIVELY` validate our findings using tools like `nmap` and `fping`:

#### passive host discovery

##### tcpdump
the following command tells `tcpdump` to listen on `eth0` interface and write the data to `file.pcap` for later analysis using `wireshark` 
```shell
sudo tcpdump -i eth0 -w file.pcap
```
##### pktmon.exe
if you are on a Windows host you may find `pktmon.exe`, which is a packet monitoring tool native to windows 10. First you need to capture some traffic and store it in a file, and then you can convert the file form etl to pcap `REQUIRES ADMIN PRIVILAGES`:
```powershell
ipconfig /all

pktmon list

pktmon start -c --comp <id> --pkt-size 0 -s 100 -f cap.etl

pktmon etl2pcap cap.etl --out cap.pcap
```

##### responder
You can also use `responder` which is used to poison `LLMNR`, `NBT-NS` and `MDNS` requests and responses. we just want it to `listen` to the network. We can use the following command:
```shell
sudo reponder -I eth0 -A 
```
This will give us an output similar to this:
```shell
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 143.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 143.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 143.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 143.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 143.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 143.0, ignoring
[Analyze mode: MDNS] Request by 10.19.103.8 for 154.0, ignoring
```
you can use `awk` to get only the IP addresses like:
```shell
grep 'Request by' responder.txt | awk '{print $6}' | sort -u
```
--- 
#### active host discovery
##### fping
we can use `fping` to send ICMP packets to a list of hosts instead of one:
```shell
fping -asgq CIDR

172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.200
172.16.5.225
172.16.5.238
172.16.5.240

     510 targets
       9 alive
     501 unreachable
...
```

- -a show live targets
- -s print status
- -g generate a target list from CIDR
- -q don't show per-target results

##### nmap
no introduction needed...
```shell
sudo nmap -v -A -iL hosts.txt -oN portscan.nmap
```
---
### Identifying Users
We can enumerate users using [kerbrute](https://github.com/ropnop/kerbrute) :
```shell
kerbrute userenum -d <DOMAIN> --dc <DC_IP> user_list -o output_list
```
This is a good approach to enumerate users as `kerbrute` sends one UDP frame to the KDC and unless kerberos logging is enabled, there won't be any logs of your requests.
