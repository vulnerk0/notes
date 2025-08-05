	## Host Discovery
more information about host discovery can be found [HERE]([https://nmap.org/book/host-discovery-strategies.html](https://nmap.org/book/host-discovery-strategies.html))
When you gain access to the internal network, you might want to view all the available hosts on that network. You can use Nmap to discover hosts with this command:
```shell
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```
NOTE: this command will work if no firewall rules prevent it and IDS is disabled
- -sn : disables port scanning
- -oA : saves the output to all formats with the file name starting with tnet
However, if you have a list of hosts and want to check for live ones you can use this command:
```shell
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```
- -iL : specifies input file
If the IPs are next to each other you can specify them using the '-' character like `10.10.10.1-11`.
You can learn why nmap marked the IP as alive is by using the `--reason` switch:
```shell
sudo nmap 10.129.2.18 -sn -oA host -PE --reason 
```
- -PE : tell nmap to use ICMP echo requests and ping the host
By default nmap first pings the target with an ARP packet, we can disable that by using the `--disable-arp-ping` like :
```shell
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```
This is an interesting page; it show the default ttl for OSs [binbert](https://www.binbert.com/blog/2009/12/default-time-to-live-ttl-values/) 
## Host and port scanning
More information about port scanning can be found [HERE](https://nmap.org/book/man-port-scanning-techniques.html)
After discovering all the hosts on the network, we want to further enumerate them. We want the following information:
- Open ports and its services
- Service versions
- Information that the services provided
- Operating system

These are all the states that a port can be in
![[Pasted image 20250627165834.png]]
### Discovering Open TCP Ports
By default nmap will scan the top 1000 ports, we can manipulate this behavior with a couple of switches:
- -p- : scan all 65535 ports
- -p xx,xxx,xx : scan the specified ports
- -p xx-xxx : scan the specified range
- --top-ports=x : scan the top X ports
- -F : top 100 ports
### Filtered Ports
There are two reasons for a port to be displayed as filtered:
- The server firewall **rejects** the packet
- The server firewall **drops** the packet
we can see in the below example that the server **drops** the packet:
```shell
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
```

nmap sent a packet which got dropped then sent another one to make sure that this is no accident. The other example shows a firewall rejecting our packets:
```shell
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
```
In this case the scanner sent the first SYN packet then received a rejection from the server with a type 3 and error code 3.
### Discovering Open UDP Ports `-sU`
The UDP ports are sometimes forgotten thus giving us some hope of finding hidden services on the server. Because UDP is a stateless protocol and does not require a three way hand shake like TCP does, we do not get any acknowledgment which means longer timeouts. Another disadvantage is that we often don't get a response back because nmap sends empty datagrams `0x00` thus is the port is displayed as open, we only got a response because the application is configured to do so.
## Saving the results
For more information visit the official website [nmap.org](https://nmap.org/book/output.html)
It's important that we save the results for further inspection. We can save the result of an nmap scan using the -o option, but we need to append one more character to it:
- N : The normal format by nmap
- G : greppable format
- X : XML document
- A : All of the above
one awesome way to present the result is by saving them in xml format and then using `xsltproc` to convert the xml to html like:
```shell
xsltproc target.xml -o target.html
```
## Scripting engine
More information about Nmap Scripting Engine can be found [Here]([https://nmap.org/nsedoc/index.html](https://nmap.org/nsedoc/index.html))
the scripts in nmap are written in lua and can provide additional functionality. Here are the categories of the scripts with a total of 14:
![[Pasted image 20250628063305.png]]
there are a couple of switches tied to scripts:
- --script \<category\> : choose a specific category
- --script \<script1\>, \<script2\> : choose specific scripts
-  -sC : use default scripts
- -A : for Aggressive, uses the following switches `-sC -sV -O --traceroute`
## Performance
Performance is important when conducting a scan for the whole network. Nmap provides some switches to enhance the performance:
- --initial-rtt-timeout : sets the initial time to wait for the packet
- --max-rtt-timeout : sets the max time to wait for the packet
- --max-retries : the number of packets sent to the same port/host for identification
if you know the bandwidth of the network, you can tell nmap to send X number of packets simultaneously with:
- --min-rate : tell nmap to send X number of packets in parallel
You also have the `-T` switch which combines all of the above, good for black box testing:
- `-T 0` / `-T paranoid`
- `-T 1` / `-T sneaky`
- `-T 2` / `-T polite`
- `-T 3` / `-T normal`
- `-T 4` / `-T aggressive`
- `-T 5` / `-T insane`
for more information about the exact switches used, visit [This Page](https://nmap.org/book/performance-timing-templates.html)
## Firewall and IDS/IPS Evasion
To read more about IDS/IPS evasion refer to [this page](https://nmap.org/book/subvert-ids.html)
Nmap has multiple ways of evading firewall rules and IDS/IPS. These systems may be configured to **drop** or **reject** the packet, in case the firewall dropped the packet no response will be received from the server. However, if the firewall rejects the packet, the server will send us a packet with a RST flag, these packets may contain nothing or contain one of the following ICMP error codes:
- Net Unreachable
- Net Prohibited
- Host Unreachable
- Host Prohibited
- Port Unreachable
- Proto Unreachable
### TCP ACK scan
The TCP ACK `-sA` scan method is much harder for firewalls to filter because the firewall can't determine if the connection was established -with a SYN packet- from the external network or the internal network
### Detect IDS/IPS
Unlike firewalls, detecting IDS and IPS can be a little bit tricky because we need to further examine the behavior of the target. First of all, you should have multiple VPSs or multiple VPN servers so if one gets blocked you won't be interrupted, IDS systems are invented to help sys admins protect the network by inspecting each packet between the hosts, if a malicious packet was found the administrator is notified. IPS however is complementary to IDS in the sense that it automatically blocks the IP associated with the malicious packet. Detecting these to systems will make us more careful and stealthy in our approach.
### Decoys
As the name suggests, nmap will disguise the packets by changing the IP header to a random IP. Thus they are decoys, then nmap will insert our IP into a position in the list and iterate between them, to use decoys use the `-D RND:X` while `RND` means random and `X` is our IP. We can also provide a list like `-D xx.xx.xx.x,xx.xx.xx.x`
```shell
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```
The spoofed packets are often filtered by ISPs and routers, we can bypass this by specifying our VPS IP addresses and use them with the "IP ID" manipulation in the IP headers, read more about "IP ID" manipulation [Here](). We can also manually specify the source IP with `-S`. This an example from HTB nmap module:
#### Testing the firewall rules
```shell-session
sudo nmap 10.129.2.28 -n -Pn -p445 -O

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:23 CEST
Nmap scan report for 10.129.2.28
Host is up (0.032s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.14 seconds
```
#### Scan by using a different source IP
```shell-session
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:16 CEST
Nmap scan report for 10.129.2.28
Host is up (0.010s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Linux 2.6.32 - 2.6.35 (94%), Linux 2.6.32 - 3.5 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.11 seconds
```
### DNS Proxying
Another fundamental way is to specify a DNS server by either using `--source-port <src>` or `--dns-server <ns>,<ns>`, usually the company trusts its own DNS server more than the outside ones, if the IDS is not configured properly we can use TCP port 53 as our source port and the target will see that the packets are coming from the DNS server. The following is an example from HTB nmap module:
#### Normal SYN-Scan
```shell-session
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 22:50 CEST
SENT (0.0417s) TCP 10.10.14.2:33436 > 10.129.2.28:50000 S ttl=41 id=21939 iplen=44  seq=736533153 win=1024 <mss 1460>
SENT (1.0481s) TCP 10.10.14.2:33437 > 10.129.2.28:50000 S ttl=46 id=6446 iplen=44  seq=736598688 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```
#### SYN-Scan from DNS port
```shell-session
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).

PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```
Now that we know that the firewall accepts TCP port 53 we will most likely be able to connect to that port via `ncat` with the `--source-port` switch like `ncat -nv --source-port 53 10.129.2.28 50000` 

## Side notes
### Exercise Answers
Q2 Medium lab : `sudo nmap -sS --reason -sC -D RND:5 -sV -Pn -n --disable-arp-ping 10.129.2.48`
the `--reason` switch is the key
Q3 Hard lab : `sudo nmap -sS -T5 --source-port 53 -sV -Pn -n --disable-arp-ping 10.129.114.121`
the key here is to connect to the service at port 50000 using ncat like `ncat -nv --source-port 53 IP 50000` - kill any process that uses port 53 like dnsmasq 
### Connect scan
The connect scan `-sT` is highly accurate duo to the full three-way handshake however it is not stealthy nor fast. But, in some cases, the fire wall will block incoming packets and allow outgoing packets, in this case the connect scan might be able to bypass that fire wall rule.
### manual service identification
Once we identify that a port is open/filtered we can try to connect to it using ncat like:
```
sudo ncat -nv --source-port 53 10.129.114.201 50000
```
you notice here that I used one of the bypasses which is DNS proxying