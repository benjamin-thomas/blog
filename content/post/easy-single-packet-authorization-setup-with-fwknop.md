---
title: "Easy Single Packet Authorization Setup With fwknop"
date: 2017-07-24T17:31:34+02:00
description: For linux servers
tags: [Admin]
---

## Step 1: Install on the client and server
```console
apt-get install fwknop-client
apt-get install fwknop-server

pacman -S fwknop # client + server
```

## Step 2: Generate a symetric key with the fwknop client

```console
fwknop --key-gen
KEY_BASE64: J74NzkCQuxRxm66XAnY1kNHYFPIVdL9bPCkyObSxUfU=
HMAC_KEY_BASE64: UkMsk1sD59asme5z+YAdJ7r376xq1iZKftKbGj0LVOY1KQSQvglZMp0eW3vQDcmZLnK76is4E99/JAp8Krw3hQ==
```

## Step 3: Add the keys on the server
```console
grep -B3 HMAC_KEY_BASE64 /etc/fwknop/access.conf
SOURCE                  ANY
REQUIRE_SOURCE_ADDRESS  Y
KEY_BASE64              J74NzkCQuxRxm66XAnY1kNHYFPIVdL9bPCkyObSxUfU=
HMAC_KEY_BASE64         UkMsk1sD59asme5z+YAdJ7r376xq1iZKftKbGj0LVOY1KQSQvglZMp0eW3vQDcmZLnK76is4E99/JAp8Krw3hQ==
```

## Step 4: Add the keys on the client
```console
grep -B5 HMAC_KEY_BASE64 ~/.fwknoprc
[my_server]
ALLOW_IP            1.1.1.1
ACCESS              tcp/22
SPA_SERVER          2.2.2.2
KEY_BASE64          J74NzkCQuxRxm66XAnY1kNHYFPIVdL9bPCkyObSxUfU=
HMAC_KEY_BASE64     UkMsk1sD59asme5z+YAdJ7r376xq1iZKftKbGj0LVOY1KQSQvglZMp0eW3vQDcmZLnK76is4E99/JAp8Krw3hQ==
```

## Step 5: Test the setup

### Launch the server in the foreground
```console
fwknopd -f
```

### Knock with the client
```console
fwknop -n my_server
```

## Step 6: Setup the firewall

```console
apt-get install iptables-persistent
```

```console
cat /etc/iptables/rules.v4
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]

# Keep state.
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Loop device.
-A INPUT -i lo -j ACCEPT

# ssh
-A INPUT -s 1.1.1.1 -p tcp --dport 22 -j ACCEPT # will be removed later

# Allow PING from remote hosts.
-A INPUT -p icmp --icmp-type echo-request -j ACCEPT
COMMIT
```

```console
cat /etc/iptables/rules.v6
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]
COMMIT
```

```console
iptables-apply /etc/iptables/rules.v4
ip6tables-apply /etc/iptables/rules.v6
```

```console
systemctl restart netfilter-persistent.service
```
Note: `netfilter-persistent.service` exists before the installation of `iptables-persistent`, but will not do anything without that package installed.

## Step 7: Test manually

At this point, the knockd server is still running in the foreground...

### On the server
```console
nc -v -l -p 2222
```

### On the client

#### Port is closed
```console
nmap -n -Pn 2.2.2.2 -p 2222
Starting Nmap
Nmap scan report for 2.2.2.2
Host is up.

PORT     STATE    SERVICE
2222/tcp filtered EtherNetIP-1
```

#### Knocking...
```console
fwknop -n my_server -A tcp/2222
```

#### Server's port is now open
```console
nmap -n -Pn 2.2.2.2 -p 2222

Starting Nmap
Nmap scan report for 2.2.2.2
Host is up (0.020s latency).

PORT     STATE SERVICE
2222/tcp open  EtherNetIP-1
```

## Step 8: Restart

Test manually once more, and if all is good after a restart, go to step 9.

```console
grep START /etc/default/fwknop-server
START_DAEMON="yes"
```

## Step 9: Last restart

Remove the hard coded IP in /etc/iptables/rules.v4 and restart one last time

### And we're done!
```console
fwknop -n my_server && ssh 2.2.2.2
```
