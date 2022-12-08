 The following configuration allows for running a secure network where tor is used as a transparent proxy for all the hosts in the LAN to access the internet, for extra security we disable ipv6 as well as any protocols other than tcp and udp (only port 53 allowed), only the firewall is allowed to directly contact the internet. 
 
 For peace of mind we will only use open source software to build this network: kvm (with virt-manager), mullvad-vpn, opnsense and tor.

 Warning: if the guest OS is compromised and rooted an attacker may change the default gateway to discover mullvad's ip address, in order to prevent this you must apply firewall rules in your host os to the virtual nat network interface for restricting access to opnsense's upstream (usually 192.168.100.1), allowing only 192.168.100.5 (opnsense's static ip) to use it.

 Please report any issues with the configuration in the issues tab and they will be fixed as soon as possible.

Host OS configuration:
```
- Host: Linux distro preferably debian or rhel based

- Install Mullvad App
 - Settings:
  - Launch App On Startup - ON
  - Auto Connect: ON
  - Lan Sharing: ON
  - Block Ads, Trackers, Adult Content....: OFF
  - Enable IPV6: OFF
  - Kill Switch: ON
  - Lockdown Mode: ON
  - Tunnel Protocol: WireGuard
  - Wireguard Settings: 
   - Port: 53
   - Obfuscation: ON (UDP-Over-TCP)
   - UDP Over TCP Port: 443
   - Enable MultiHop: ON
   - IP Version: 4
   - MTU: Default
  - Use Custom DNS Server: OFF

- Install virt-manager
- Configuring the network on virt-manager
 - edit > connection details 
  - delete default network
  - make new network called default
  - Nat mode
  - Leave 192.168.100.0/24 as the nat network
  - pass to physical device wg-mullvad 
```

OpnSense Configuration:
```
- install opnsense on a vm
- configure LAN on vtnet0 with upstream as 192.168.100.1
- assign 192.168.100.5 as the ip address
- leave ipv6 unconfigured
- update opnsense from the web gui
 
- go to system > firmware > plugins and install os-tor
- go to services > tor > configuration
 - Socks Proxy ACL 
  - enable: check 
  - protocol: IPV4
  - network: 0.0.0.0
  - action: accept
 - General > Advanced Mode 
  - listen interfaces: lan
  - enable: check
  - enable transparent proxy: check
  - transparent ip pool: 10.64.0.0/16

- firewall > nat > port forward 
 - new rule 
  - interface: LAN
  - version: IPV4
  - Protocol: TCP
  - source/invert: check
  - source: this firewall
  - source port range: any - any
  - destination: any
  - destination ports: any
  - redirect target ip: single host or network 127.0.0.1
  - redirect target port: other 9040
  - filter rule association: pass

- firewall > nat > port forward
 - new rule 
  - interface: LAN
  - version: IPV4
  - Protocol: UDP
  - source/invert: check
  - source: this firewall
  - source port range: any - any
  - destination: any
  - destination ports: DNS
  - redirect target ip: single host or network 127.0.0.1
  - redirect target port: other 9053
  - filter rule association: pass

- firewall > nat > port forward
 -new rule
  - interface: LAN
  - version: IPV4/IPV6
  - Protocol: ANY
  - Destination: ANY

- firewall > rules > LAN
 - delete "Default allow LAN IPv6 to any rule"
 - edit "Default allow LAN to any rule"
  - version: ipv6+ipv4
  - destination: LAN net
 - new rule
  - Action: block
  - Interface: LAN
  - Direction: in
  - Version: ipv4/ipv6
  - Protocol: any
  - Source/invert: check
  - Source: this firewall
```

Guest OS Configuration:
```
- for windows do it through the gui network manager (same way: address, netmask, gateway)
- for other OS google how to do static network configuration
- increment the ip address if you're running more than one guest
- edit the following file and add the contents below: /etc/network/interfaces then reboot the machine 

auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.168.100.25
netmask 255.255.255.0
gateway 192.168.100.5
```
