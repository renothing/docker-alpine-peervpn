# Docker image running PeerVPN

## Usage

Pull the image from Dockerhub.

    docker pull renothing/peervpn

### Running the image directly

The default `ENTRYPOINT` will generate a configuration file for PeerVPN
(unless one exists already) based on supplied environment variables and 
then run the `peervpn` binary.

The following environment variables are supported. Note that some default
values are of limited use.

| Variable | Default |
|----------|---------|
| NETWORKNAME | PEERVPN$RANDOM |
| PSK | PSK$RANDOM |
| INITPEERS | example.com 7000 |
| ENABLETUNNELING | yes |
| INTERFACE | peervpn0 |
| IFCONFIG4 | 172.16.254.$(expr $RANDOM % 256)/24 |
| IFCONFIG6 | fe80::1034:56ff:fe78:$(expr $RANDOM % 10000)/64 |
| UPCMD | your init cmd here |
| LOCAL | 0.0.0.0 |
| PORT | 7000 |
| ENABLEIPV4 | yes |
| ENABLEIPV6 | no |
| ENABLENDP | yes |
| ENABLERELAY | no |

The example below will run a VPN between two containers. Both containers must
configure different UDP ports (7001 and 7002) as they are on the same host. 
In the example below the IP address of the host running Docker is 10.0.2.15.
```
docker run --name=vpn1 -p 7001:7001/udp --cap-add=NET_ADMIN \
        -e NETWORKNAME=mynet -e PSK=mykey -e PORT=7001 \
        -e INITPEERS='10.0.2.15 7002' -e IFCONFIG4='172.16.1.1/24' -d \
        renothing/peervpn
    
docker run --name=vpn2 -p 7002:7002/udp --cap-add=NET_ADMIN \
        -e NETWORKNAME=mynet -e PSK=mykey -e PORT=7002 \
        -e INITPEERS='10.0.2.15 7001' -e IFCONFIG4='172.16.1.2/24' -d \
        renothing/peervpn
```
use dhclient for autoscaling with the following upcmd:

peervpn-dnsmasq.conf
```
interface dnsmasq0
ifconfig4 10.0.2.1/24
upcmd dnsmasq -i dnsmasq0 --dhcp-range=10.0.2.2,10.0.2.254,255.255.255.0,12h
port 5678
```

peervpn-dhclient.conf
```
initpeers 127.0.0.1 5678
interface dhclient0
upcmd dhclient -nw dhclient0
```

  for docker:
```
#for init peer or any node you want run dhcp
UPCMD=dnsmasq -i $INTERFACE --dhcp-range=10.1.1.7,10.1.1.250,255.255.255.0,infinite
#for other peer
UPCMD=dhclient -nw ${INTERFACE}
```
Apparently, DHCP will work over the VPN. So, you could have a static head node (in this example defined in peervpn-dhclient.conf with initpeers 127.0.0.1 5678) that serves up DHCP addresses.

So, you would only need to specify a static address for one node (so dnsmasq can bind to an address) and each other nodes would need to know at least the head node's (or another node's) address.

ref: https://github.com/peervpn/peervpn/issues/10#issuecomment-176987373

### Use as a base image

This image can also be used as a base image. `COPY` your own PeerVPN configuration
file and overwrite the `ENTRYPOINT`.

### Running image via docker-compose

```
version: '3'

services:
    peervpn:
        image: renothing/peervpn
        network_mode: "host"
        cap_add:
            - NET_ADMIN
        restart: "always"
        environment:
            NETWORKNAME: srvnet
            INTERFACE: "srvnet0"
            PSK: thisisapresharedkey
            LOCAL: "0.0.0.0"
            PORT: 7000
            INITPEERS: "1.2.3.4 7000"
            IFCONFIG4: "10.8.0.1/24"
            ENABLERELAY: "yes"
            ENABLEIPV6: "no"
```

## Authors

* renothing (Fork)
* Thomas Leister <thomas.leister@mailbox.org> (Fork)
* Markus Juenemann <markus@juenemann.net> (Original author)
