# Mininet Walkthrough
## Part 1: Everyday Mininet Usage
First, a (perhaps obvious) note on command syntax for this walkthrough:

$ preceeds Linux commands that should be typed at the shell prompt
mininet> preceeds Mininet commands that should be typed at Mininet’s CLI,
#preceeds Linux commands that are typed at a root shell prompt
In each case, you should only type the command to the right of the prompt (and then press return, of course!)

### Install Mininet
For now, there should be no OpenFlow packets displayed in the main window.

Note: Wireshark is installed by default in the Mininet VM image. If the system you are using does not have Wireshark and the OpenFlow plugin installed, you may be able to install both of them using Mininet’s install.sh script as follows:
```bash
farhan@Arctic:~/Github$ git clone git://github.com/mininet/mininet
farhan@Arctic:~/Github/mininet$ util/install.sh -w
farhan@Arctic:~/Github/mininet$ util/install.sh -a
farhan@Arctic:~$ sudo apt install net-tools
```

### Interact with Hosts and Switches
Start a minimal topology and enter the CLI:
```bash
farhan@Arctic:~$ sudo mn
*** Creating network
*** Adding controller
*** Adding hosts:
h1 h2 
*** Adding switches:
s1 
*** Adding links:
(h1, s1) (h2, s1) 
*** Configuring hosts
h1 h2 
*** Starting controller
c0 
*** Starting 1 switches
s1 ...
*** Starting CLI:
mininet> 
```

Display Mininet CLI commands:
```bash
mininet> help

Documented commands (type help <topic>):
========================================
EOF    gterm  iperfudp  nodes        pingpair      py      switch
dpctl  help   link      noecho       pingpairfull  quit    time  
dump   intfs  links     pingall      ports         sh      x     
exit   iperf  net       pingallfull  px            source  xterm 
```

Display nodes:
```bash
mininet> nodes
available nodes are: 
c0 h1 h2 s
```

Display links:
```bash
mininet> nodes
available nodes are: 
c0 h1 h2 s1
mininet> net
h1 h1-eth0:s1-eth1
h2 h2-eth0:s1-eth2
s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
c0
```

Display links:
```bash
mininet> net
h1 h1-eth0:s1-eth1
h2 h2-eth0:s1-eth2
s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
c0
```

Dump information about all nodes:
```bash
mininet> dump
<Host h1: h1-eth0:10.0.0.1 pid=19090> 
<Host h2: h2-eth0:10.0.0.2 pid=19092> 
<OVSSwitch s1: lo:127.0.0.1,s1-eth1:None,s1-eth2:None pid=19097> 
<Controller c0: 127.0.0.1:6653 pid=19083> 
```

You should see the switch and two hosts listed.

If the first string typed into the Mininet CLI is a host, switch or controller name, the command is executed on that node. Run a command on a host process:
```bash
mininet> h1 ifconfig -a
h1-eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.1  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::bc11:59ff:feba:15dd  prefixlen 64  scopeid 0x20<link>
        ether be:11:59:ba:15:dd  txqueuelen 1000  (Ethernet)
        RX packets 44  bytes 6293 (6.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13  bytes 1006 (1.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Note that only the network is virtualized; each host process sees the same set of processes and directories. For example, print the process list from a host process:
```bash
mininet> h1 ps -a
    PID TTY          TIME CMD
   2250 tty2     00:02:12 Xorg
   2313 tty2     00:00:00 gnome-session-b
  19077 pts/0    00:00:00 sudo
  19078 pts/0    00:00:00 mn
  19118 pts/1    00:00:00 controller
  20176 pts/2    00:00:00 ps
```

### Test connectivity between hosts
Now, verify that you can ping from host 0 to host 1:
```bash
mininet> h1 ping -c 1 h2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=90.4 ms

--- 10.0.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 90.391/90.391/90.391/0.000 ms
2
```
