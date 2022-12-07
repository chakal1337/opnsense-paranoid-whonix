Host OS configuration
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
  - pass to physical device wg-mullvad 
```
