# edgerouter-lite-config

See https://kriegsman.io/2016/01/configuring-a-ubiquiti-edgerouter-lite-for-kpn/

Step 1: 
Configure ethernet to 192.168.1.2/24 with gateway 192.168.1.1

```
configure
 
set interfaces ethernet eth1 address 192.168.2.254/24
set interfaces ethernet eth1 description "eth1 - LAN"
set interfaces ethernet eth1 duplex auto
set interfaces ethernet eth1 speed auto
 
set service dhcp-server disabled false
set service dhcp-server hostfile-update disable
set service dhcp-server shared-network-name LAN authoritative enable
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24 default-router 192.168.2.254
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24 dns-server 8.8.8.8
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24 dns-server 8.8.4.4
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24 lease 86400
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24 start 192.168.2.50
set service dhcp-server shared-network-name LAN subnet 192.168.2.0/24 start 192.168.2.50 stop 192.168.2.200
 
set service dns forwarding cache-size 150
set service dns forwarding listen-on eth1
set service dns forwarding name-server 8.8.8.8
set service dns forwarding name-server 8.8.4.4
set service dns forwarding options listen-address=192.168.2.254
 
commit
save
exit
```

Step 2:
```
configure
 
set firewall all-ping enable
set firewall broadcast-ping disable
set firewall ipv6-receive-redirects disable
set firewall ipv6-src-route disable
set firewall ip-src-route disable
set firewall log-martians enable
set firewall receive-redirects disable
set firewall send-redirects enable
set firewall source-validation disable
set firewall syn-cookies enable
 
set firewall name WAN_IN default-action drop
set firewall name WAN_IN description "WAN to Internal"
set firewall name WAN_IN enable-default-log
set firewall name WAN_IN rule 10 action accept
set firewall name WAN_IN rule 10 description "Allow established/related"
set firewall name WAN_IN rule 10 log enable
set firewall name WAN_IN rule 10 protocol all
set firewall name WAN_IN rule 10 state established enable
set firewall name WAN_IN rule 10 state invalid disable
set firewall name WAN_IN rule 10 state new disable
set firewall name WAN_IN rule 10 state related enable
 
set firewall name WAN_IN rule 20 action drop
set firewall name WAN_IN rule 20 description "Drop invalid state"
set firewall name WAN_IN rule 20 log enable
set firewall name WAN_IN rule 20 protocol all
set firewall name WAN_IN rule 20 state established disable
set firewall name WAN_IN rule 20 state invalid enable
set firewall name WAN_IN rule 20 state new disable
set firewall name WAN_IN rule 20 state related disable
 
set firewall name WAN_LOCAL default-action drop
set firewall name WAN_LOCAL description "WAN to router"
set firewall name WAN_LOCAL enable-default-log
set firewall name WAN_LOCAL rule 10 action accept
set firewall name WAN_LOCAL rule 10 description "Allow established/related"
set firewall name WAN_LOCAL rule 10 log disable
set firewall name WAN_LOCAL rule 10 protocol all
set firewall name WAN_LOCAL rule 10 state established enable
set firewall name WAN_LOCAL rule 10 state invalid disable
set firewall name WAN_LOCAL rule 10 state new disable
set firewall name WAN_LOCAL rule 10 state related enable
 
set firewall name WAN_LOCAL rule 20 action drop
set firewall name WAN_LOCAL rule 20 description "Drop invalid state"
set firewall name WAN_LOCAL rule 20 log disable
set firewall name WAN_LOCAL rule 20 protocol all
set firewall name WAN_LOCAL rule 20 state established disable
set firewall name WAN_LOCAL rule 20 state invalid enable
set firewall name WAN_LOCAL rule 20 state new disable
set firewall name WAN_LOCAL rule 20 state related disable
 
commit
save
exit
```
Step 3:
```
configure
 
delete interfaces ethernet eth0 address
 
set interfaces ethernet eth0 description "eth0 - FTTH"
set interfaces ethernet eth0 duplex auto
set interfaces ethernet eth0 speed auto
set interfaces ethernet eth0 mtu 1512
 
set interfaces ethernet eth0 vif 6 description "eth0.6 - Internet"
set interfaces ethernet eth0 vif 6 mtu 1508
 
set interfaces ethernet eth0 vif 6 pppoe 0 user-id FB7490@xs4all.nl
set interfaces ethernet eth0 vif 6 pppoe 0 password xs4all
set interfaces ethernet eth0 vif 6 pppoe 0 default-route auto
set interfaces ethernet eth0 vif 6 pppoe 0 name-server auto
set interfaces ethernet eth0 vif 6 pppoe 0 idle-timeout 180
set interfaces ethernet eth0 vif 6 pppoe 0 mtu 1500
 
set interfaces ethernet eth0 vif 6 pppoe 0 firewall in name WAN_IN
set interfaces ethernet eth0 vif 6 pppoe 0 firewall local name WAN_LOCAL
 
set system name-server 8.8.8.8
set system name-server 8.8.4.4
 
commit
save
exit
```
Step 4: 
Make sure to update to a version that supports this
```
configure
 
set system offload ipv4 forwarding enable
set system offload ipv4 pppoe enable
set system offload ipv4 vlan enable
 
commit
save
exit
```

Step 5: 

```
configure
 
set service nat rule 5010 description "KPN Internet"
set service nat rule 5010 log enable
set service nat rule 5010 outbound-interface pppoe0
set service nat rule 5010 protocol all
set service nat rule 5010 source address 192.168.2.0/24
set service nat rule 5010 type masquerade
 
commit
save
exit
```
