# Usage #

## Adding rules ##

First, rules accounting traffic must be created.

To gather traffic statistics for network 192.168.0.0/24 passing thru the router and traffic statistics to WWW server (installed on router) for the same network as before use the following commands.

```
# iptables -A FORWARD -m account --aaddr 192.168.0.0/24 --aname mynetwork
# iptables -A INPUT -p tcp --dport 80 -m account --aaddr 192.168.0.0/24 --aname myserver --ashort
# iptables -A OUTPUT -p tcp --dport 80 -m account --aaddr 192.168.0.0/24 --aname myserver --ashort
```

In the first rule we gather traffic statistics for network 192.168.0.0/24 passing thru the router.

In the second rule we gather traffic statistics to WWW server (installed on router) for network 192.168.0.0/24. Because of --ashort parameter module will gather only total statistics (without each protocol statistics).

In the third rule we gather traffic statistics from WWW server (installed on router) to network 192.168.0.0/24. Because table 'myserver' already exists will be used again.

'''Note: To use existing table in many rules (like in third rule above), check whether network address/netmask is the same in all rules. If network address/netmask differs, new rule won't be created).'''

After executing above commands you will see new entries in the procfs /proc/net/ipt\_account/ directory:
```
# ls -laR /proc/net/ipt_account/
/proc/net/ipt_account/:
total 0
dr-xr-xr-x    2 root     root            0 Apr 2 17:21 .
dr-xr-xr-x    4 root     root            0 Apr 2 17:21 ..
-rw-r--r--    1 root     root            0 Apr 2 17:21 mynetwork
-rw-r--r--    1 root     root            0 Apr 2 17:21 myserver                 
```

## Viewing statistics ##

To view gathered statistics just cat appropriate file.

```
# cat /proc/net/ipt_account/mynetwork
ip = 192.168.0.0 bytes_src = 8009371355 7979197249 25417729 4756377 0 packets_src = 14844174 14600134 186011 58029 0 bytes_dest = 17568766197 17540073337  25092213 3600647 0 packet
s_dest = 17462235 17236701 182662 42872 0 time = 0
ip = 192.168.0.1 bytes_src = 702 0 702 0 0 packets_src = 9 0 9 0 0 bytes_dest = 0 0 0 0 0 packets_dest = 0 0 0 0 0 time = 12
ip = 192.168.0.2 bytes_src = 133164609 133071748 92441 420 0 packets_src = 2548317 2547888 422 7 0 bytes_dest = 7331211760 7331210800 540 420 0 packets_dest = 4928417 4928404 6 7 0 time = 1
ip = 192.168.0.3 bytes_src = 0 0 0 0 0 packets_src = 0 0 0 0 0 bytes_dest = 0 0 0 0 0 packets_dest = 0 0 0 0 0 time = 123124
ip = 192.168.0.4 bytes_src = 168821 0 168821 0 0 packets_src = 2043 0 2043 0 0 bytes_dest = 180 0 180 0 0 packets_dest = 2 0 2 0 0 time = 12
ip = 192.168.0.5 bytes_src = 147593 0 147593 0 0 packets_src = 1324 0 1324 0 0 bytes_dest = 0 0 0 0 0 packets_dest = 0 0 0 0 0 time = 12
...
```

Each row in that file contains traffic statistics for one IP. First row contains sum of all traffic statistics in whole table. In each row you can find six fields.

|'''Field'''|'''Description'''|
|:----------|:----------------|
|IP         |IP of the host   |
|bytes\_src |statistics in bytes for "outgoing" traffic of that host. Field is followed by five numbers. The first number is the total, the second one  is TCP, the third one UDP, the fourth one is ICMP and finally the fifth one is traffic for all other protocols|
|packets\_src|same as above but in packets instead of bytes|
|bytes\_dest|statistics in bytes for "incomming" traffic of that host. Field is followed by five numbers. The first number is the total, the second one  is TCP, the third one UDP, the fourth one is ICMP and finally the fifth one is traffic for all other protocols|
|packets\_dest|same as above byt in packets instead of bytes|
|time       |time when last update to specified row was made. It's in seconds from now|

When the table is created with --ashort switch, the output is slightly different. For each field in the row (bytes\_src, packets\_src, bytes\_dest, pakcets\_dest) you will only see total statistics.

```
# cat /proc/net/ipt_account/myserver
ip = 192.168.0.0 bytes_src = 12309123 packets_src = 123145 bytes_dest = 3252355 packets_dest = 242132 time = 0
...
```

'''Note: if you do not need protocol (TCP/UDP/ICMP/others) statistics, please use --ashort switch. It will result in smaller memory allocation.'''

## Time field ##

Time field is by default updated when either incomming or outgoing traffic is accounted.

## Deciding what to show ##

You can define which rows will be shown and which won't. You can decide whether show only rows with any "outgoing" traffic accounted (non-zero values of _**src columns, values of**_dst columns doesn't matter):

```
echo "show=src" > /proc/net/ipt_account/mynetwork
```

or only rows with any "incomming" traffic accounter (non-zero values of _**dst columns, values of**_src columns doesn't matter):

```
echo "show=dst" > /proc/net/ipt_account/mynetwork
```

You can also combine these filters and show rows with any type of traffic accounted (non-zero values of _**src columns '''or''' non-zero values of**_dst columns):

```
echo "show=src-or-dst" > /proc/net/ipt_account/mynetwork
```

or with both types of traffic accounted (non-zero values of _**src columns '''and''' non-zero values of**_dst columns):

```
echo "show=src-and-dst" > /proc/net/ipt_account/mynetwork
```

'''Note: Instead of "show=src-or-dst" you can write "show=dst-or-src". Also intead of "show=src-and-dst" you can write "show=dst-and-src".'''

## Reseting counters ##

You can quickly reset (zero) all counters in the table using following command:

```
# echo "reset" > /proc/net/ipt_account/mynetwork
```

Module support reset-after-read feature. To enable this feature enter the following command:

```
# echo "reset-on-read=yes" > /proc/net/ipt_account/mynetwork
```

With this feature enabled, each read on /proc/net/ipt\_account/table/mynetwork will automaticaly reset all counters in the table.

'''Note: Instead of "reset-on-read=yes" you can write just "reset-on-read".'''

To disable reset-after-read feature enter the following command:

```
# echo "reset-on-read=no" > /proc/net/ipt_account/mynetwork
```

## Saving and loading counters ##

The counters inside table can be freely set. For example, entering the following command will change counters for 192.168.0.251 host.

```
# echo "ip = 192.168.0.251 bytes_src = 1 2 3 4 5 packets_src = 6 7 8 9 0 bytes_dest = 1 2 3 4 5 packets_dest = 6 7 8 9 0 time = 0" > /proc/net/ipt_account/mynetwork    
```

If you have table created with --ashort switch change counters like shown below.

```
# echo "ip = 192.168.0.251 bytes_src = 1 packets_src = 2 bytes_dest = 3 packets_dest = 4 time = 0" > /proc/net/ipt_account/myserver
```

'''Note: time field is ignored, but it must be given.'''

This feature can be used to save counter values before router reboot, and than restore them after reboot.

```
# cat /proc/net/ipt_account/myserver > myserver.save                    
...
# while read line; do echo $line > /proc/net/ipt_account/myserver; done < myserver.save
...                     
```

'''Note: Counters must be loaded in row-by-row order. Below command won't work.'''

```
# cat myserver.save > /proc/net/ipt_account/myserver
```