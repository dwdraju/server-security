## Probing Ports

### Nmap
Employ nmap to check if port is open or not.
```
nmap [ip]
```
* Open: Port has a service or daemon listening
```
SYN -> SYN/ACK
```
* Closed: Don’t have a daemon listening behind the port but is open
```
SYN -> RST
```
* Filtered: Firewall is preventing access to the IP address that is scanning the system
```
SYN -> ICMP error
```


### Confusing Port Scanner with Iptables
Truncate the response to confuse sophisticated port scanning
```
-A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
-A INPUT -p tcp -j REJECT --reject-with tcp-reset
-A INPUT -j REJECT --reject-with icmp-proto-unreachable
```

#### Drop VS Reject
* Drop on public interfaces, reject on the internal interfaces(ICMP is vital to path MTU discovery)
* Drop better for security, client can't know firewall is blocking the connection
* Less resource utilization but reject might suffer DDOS attack

DROP rather than REJECT is to avoid giving away information about which ports are open, however, discarding packets gives away exactly as much information as the rejection.

More on [www.chiark.greenend.org.uk/~peterb/network/drop-vs-reject](www.chiark.greenend.org.uk/~peterb/network/drop-vs-reject)

### Port Knocking with knockd
```
$ apt-get install knockd
```
Config file at */etc/knockd.conf*
```
[options]
		UseSyslog

[openSSH]
		sequence = sequence = 6:tcp 1450:udp 8156:udp 22045:tcp 23501:udp 24691:tcp
		seq_timeout = 5
		command = /sbin/iptables -I INPUT -s %IP% -p tcp—dport 22 -j
	ACCEPT
		tcpflags = syn
		seq_timeout = 10
		cmd_timeout = 15

[closeSSH]
		sequence = 3011,6145,7298
		seq_timeout = 5
		command = /sbin/iptables -D INPUT -s %IP% -p tcp—dport 22 -j
	ACCEPT
		tcpflags = syn

[options]
		LogFile = /var/log/knockd.log
		PidFile = /var/tmp/knockd/file	
```

Default is: 7000, 8000, and 9000 to open up SSH access and ports 9000, 8000, 7000 to close access. Change the configuration and restart knockd service:

```
$ systemctl restart knockd.service
```
Now, open the port for connection:
```
$ knock [options] <host> <port[:proto]> <port[:proto]> <port[:proto]>
$ knock [IP] 6:tcp 1450:udp 8156:udp 22045:tcp 23501:udp 24691:tcp
```


