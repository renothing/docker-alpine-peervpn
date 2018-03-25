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
| ENABLEIPV6 | yes |
| ENABLERELAY | no |

The example below will run a VPN between two containers. Both containers must
configure different UDP ports (7001 and 7002) as they are on the same host. 
In the example below the IP address of the host running Docker is 10.0.2.15.

    docker run --name=vpn1 -p 7001:7001/udp --cap-add=NET_ADMIN \
        -e NETWORKNAME=mynet -e PSK=mykey -e PORT=7001 \
        -e INITPEERS='10.0.2.15 7002' -e IFCONFIG4='172.16.1.1/24' -d \
        renothing/peervpn
    
    docker run --name=vpn2 -p 7002:7002/udp --cap-add=NET_ADMIN \
        -e NETWORKNAME=mynet -e PSK=mykey -e PORT=7002 \
        -e INITPEERS='10.0.2.15 7001' -e IFCONFIG4='172.16.1.2/24' -d \
        renothing/peervpn

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
