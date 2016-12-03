# Networking for Web Developers

## From Ping to HTTP

### Ping
The `ping` utility sends an echo request to a computer's operating system.

### Netcat
The `nc` (or netcat) utility is used for just about anything under the sun involving TCP, UDP, or UNIX-domain sockets.  It can open TCP connections, send UDP packets, listen on arbitrary TCP and UDP ports, do port scanning, and deal with both IPv4 and IPv6. Common uses include: simple TCP proxies, shell-script based HTTP clients and servers, network daemon testing, a SOCKS or HTTP ProxyCommand for ssh and much, much more.
##### Client-Server Model
It is quite simple to build a very basic client/server model using nc. On one console, start `nc` listening on a specific port for a connection. For example:
```
nc -l 1234
```
nc is now listening on port 1234 for a connection.  On a second console (or a second machine), connect to the machine and port being listened on:
```
nc 127.0.0.1 1234
```
There should now be a connection between the ports. Anything typed at the second console will be concatenated to the first, and vice-versa. After the connection has been set up, `nc` does not really care which side is being used as a server and which side is being used as a client. The connection may be terminated on either side using an EOF (`^D`). This is an example of a plain TCP client/server.  
The port range that a normal (non-root) user can listen on is 1024 through 65535. But if you use root access (including sudo) then you can listen on ports down to 1.
##### Data Transfer
The example in the previous section can be expanded to build a basic data transfer model.  Any information input into one end of the connection will be output to the other end, and input and output can be easily captured in order to emulate file transfer.
Start by using `nc` to listen on a specific port, with output captured into a file:
```
nc -l 1234 > filename.out
```
Using a second machine, connect to the listening `nc` process, feeding it the file which is to be transferred:
```
nc host.example.com 1234 < filename.in
```
After the file has been transferred, the connection will close automatically.
##### Talking to Servers
It is sometimes useful to talk to servers “by hand” rather than through a user interface.  It can aid in troubleshooting, when it might be necessary to verify what data a server is sending in response to commands issued by the client. For example, to retrieve the home page of a website:
```
printf "GET / HTTP/1.0\r\n\r\n" | nc host.example.com 80
```
Note that this also displays the headers sent by the web server. They can be filtered, using a tool such as sed(1), if necessary.  
More complicated examples can be built up when the user knows the format of requests required by the server. As another example, an email may be submitted to an SMTP server using:
```
nc [-C] localhost 25 << EOF
HELO host.example.com
MAIL FROM:<user@host.example.com>
RCPT TO:<user2@host.example.com>
DATA
Body of email.

QUIT
EOF
```

### Other Stuff
The HTTP layer is implemented by programs such as web browsers and servers, while the TCP is implemented in the operating system. A program listening on a TCP port is waiting for another program to connect to that port. Once that happens, the programs can send data back and forth.  
The `lsof` linux utility lists open files, including network sockets (listening or connected). The following command will make it listen to internet sockets specifically: `sudo lsof -i`

## Names and Addresses

### Definitions
*Host:* A machine on the internet that might host services.  
*Endpoints:* The two machines or programs communicating over the connection.  

### Host and Dig
The `host` command is a utility for looking up records in the DNS. See the IP addresses of Udacity with `host -t a www.udacity.com`. The `dig` utility shows the same information but in a form more readily processed by scripts. Try `dig www.udacity.com`.

### DNS and HTTP
Domain names are essential not only for the DNS to map the domain name to an IP address, but also for other HTTP features such as cookies and SSL. A single web server can handle requests for multiple sites which it tells apart using the domain name in the `Host` request header. Apache calls this a *virtual host configuration* while nginx calls this having *multiple server blocks*.  
When a web app sets a cookie, it does so for a particular domain name, and further requests to that domain will get that cookie sent back.  
SSL encryption certificates are issued for particular domains. By encrypting the traffic between browser and server, it prevents networks in the middle from reading private data. It also lets the user's browser verify that the site they're getting data from is actually the site that they expect.

### IP Addresses
There are two major versions of the IP protocol in use today: IPv4 and IPv6. IPv4 addresses are made of 4 octets/bytes (8 bit values, ranging from 0 to 255) separated by dots.

### Binary-Decimal Conversions
Highest decimal number represented with a given number of bits:
```
>>> 2**10
1024
```
Number of bits needed to represent a decimal number:
```
>>> from math import log
>>> log(1024, 2)
10.0
```

## Addressing and Networks

### Netblocks and Subnets
Computers that are on the same network block (home, business, school network) usually have IP addresses that all share a particular prefix. Computers on the same network block can communicate with each other without going through a router.  
A network with a longer prefix has less of the 32 bits in IPv4 addresses left to distinguish particular hosts. The network prefix length is chosen when the network is setup.  
For instance, a network with a 24 bits prefix has 8 bits left for the host part of addresses. Conventionally, this is written as `{prefix}/24`. Eg: `198.51.100.0/24`. One can't tell how long the prefix is just by looking at the addresses.  
The top and bottom addresses of a network block are reserved, and the first address is usually the router. So a network has a 24 bits prefix and 8 bits left for host address, there are 256 - 3 = 253 host addresses left for hosts.  
A subnet mask is a binary number made of 1s on the left and 0s on the right, to indicate the length of the prefix. The mask for a `/24` network would be `11111111111111111111111100000000` or `255.255.255.0`. The decimal subnet mask for a `/16` network would be `255.255.0.0`, `255.252.0.0` for a `/14` network, and so on.  
Addresses don't belong to hosts as much as they belong to interfaces on those hosts. For example, a laptop might have Ethernet and WiFi interfaces, as well as a loopback interface for talking to itself. It might also have tunnel and VM interfaces. You can find out which interfaces your machine has with `ip addr show`.

### Routers and Default Gateways
A router is a device that connects two different IP networks and acts as a Gateway. While most hosts might have only one interface, a router will have two or more. A host on a local network knows about a default gateway, which is a router connected towards the rest of the internet. Computers connected to the same switch or WiFi access point are normally local to each other, and can send packets to one another without going through a different network. You can print a machine's default gateway with `ip route show default`.  
A home network is typically comprised of several computers: a desktop, laptop, a few smart phones, etc. All of these access the internet through a router, which acts as their default gateway. The router connects to their ISP and gets a single public IP address. It then assigns private addresses to all the devices in the home network. Private IP addresses come off of one of 3 reserved IP address netblocks: `10.0.0.0/8`, `172.16.0.0/12` or `192.168.0.0/16`. The most common private IP addresses found on home routers are in the block `192.168.0.0/24` (a subset of the `192.168.0.0/16` block) with a default gateway of `192.168.0.1`.

## Protocol Layers

### Abstraction Layers and Protocols
| Level       | Protocols     |
|-------------|---------------|
| Application | HTTP SSH NTP  |
| Transport   | TCP UDP       |
| Internet    | IP            |
| Hardware    | Ethernet WiFi |

### TCP Connection Establishment
At the start, the client knows the server's IP address and port but the server knows nothing about the client. The client has to send a message to the sever containing - among other things - it's IP address and port number. TCP also keeps track of the data sent by each endpoint. It ensures that the recipient of a data packet has received the packet, and that it receives sequences of packets in order, even if the underlying network re-orders them. This is done by putting a sequence number on each packet and having endpoints send an acknowledgement indicating the reception of a particular sequence number. If a packet goes missing, the sender will notice there is no acknowledgement and will re-transmit it. Sequence numbers start out at a random number, so they're unlikely to get confused between one connection and another. See *Connection Establishment* video in *Protocol Layers* lesson.  
Packets exchanged between a machine acting as a client and a server can be checked with the `tcpdump` utility. In one terminal, get `tcpdump` to monitor exchanges with example.net.
```
sudo tcpdump -n host example.net
```
In another terminal, send an HTTP request to example.net.
```
printf 'HEAD / HTTP/1.1\r\nHost: example.net\r\n\r\n' | nc example.net 80
```

### TCP Flags
Packet records produced by `tcpdump` have a section called Flags that appears right after the address and port information. This section has one or more letters or dots inside square brackets, such as `[S]`, `[S.]`, `[.]`, `[P.]`, and `[F.]`. The original TCP packet format has six flags. Two more optional flags have since been standardized, but they are much less important to the basic functioning of TCP. For each packet, `tcpdump` will show which flags are set on that packet. Flag meanings are as follows:
  * SYN (synchronize) `[S]`: This packet is opening a new TCP session and contains a new initial sequence number `seq`.
  * FIN (finish) `[F]`: This packet is used to close a TCP session normally. The sender is saying that they are finished sending, but they can still receive data from the other endpoint.
  * PSH (push) `[P]`: This packet is the end of a chunk of application data, such as an HTTP request.
  * RST (reset) `[R]`: This packet is a TCP error message; the sender has a problem and wants to reset (abandon) the session.
  * ACK (acknowledge) `[.]`: This packet acknowledges that its sender has received data from the other endpoint. Almost every packet except the first SYN will have the ACK flag set.
  * URG (urgent) `[U]`: This packet contains data that needs to be delivered to the application out-of-order. Not used in HTTP or most other current applications.

## Big Networks

### Traceroute
Every data packet traveling on a network has a time to live attribute, which is reduced by 1 each time that packet hits a router. So if routers are misconfigured, such that packets flow around in an infinite loop, packets will eventually expire, preventing network overload. When a packet expires, the router that last received it sends a tiny error message back to the original sender.

### Bandwidth-Delay Product
The bandwidth-delay product, measured in bits, is the product of bandwidth (bits/s) and latency (s). This corresponds to the amount of data that can be in transit through a connection at any time.

### Middleboxes 1: Firewalls
Firewalls are devices that network operators can use to filter traffic that's coming into or leaving their network. A firewall is one example of a class of network devices called middleboxes — devices that inspect, modify, or filter network traffic. Other examples of middleboxes include intrusion detection systems and load balancers. Technically, it's only a middlebox if it's a separate device from the client or server. Server-side "firewalls" like Linux iptables aren't middleboxes.  
The most common configuration for a firewall is to drop any incoming traffic except traffic to (host, port) pairs that are supposed to be receiving connections from the Internet. This lets the network administrator be sure that other machines on the network - like backend databases or administrative systems - aren’t going to get direct attacks from outside.  
But firewalls can cause trouble for application developers. If you're trying to test or deploy a network app and there's a firewall between your server and the user, that firewall can potentially interfere with your app or block it completely. In order to deploy an application on a particular server and port, it helps to know what kind of firewall might be between you and your user. One of the reasons that many non-Web applications use HTTP as a transport is that HTTP is often unblocked at firewalls even when other ports are blocked.

### Middleboxes 2: Proxies and NAT
With Network Address Translation, or NAT, several devices can access Internet resources through a single public IP address, with the NAT device using port numbers to match up connections on the inside and outside.  
For end-users, NAT devices overlap with firewalls. Typical home routers can act as both a NAT and a simple firewall, often having the ability to block or filter at a very basic level. At a larger scale, ISPs and other organizations have deployed NAT devices for their whole customer networks, called carrier-grade NAT. This is very common for mobile networks, and also for ISPs in the developing world, where there never were anywhere near enough addresses allocated for the number of users.  
With NAT, your website can see requests from the same IP address that actually come from different users on different computers. Another way that can happen is through the use of web proxies. Whereas a NAT works at the IP level, rewriting packets, a web proxy works at the HTTP level, taking queries from browsers and sending them out to web servers. Many organizations use web proxies for caching, including some ISPs. From the standpoint of a web developer or site operator, traffic from a busy proxy looks much the same as traffic from a busy NAT: queries for many users, on many actual computers, are funneled through a single public IP address.
