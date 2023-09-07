#H1 NTT-lab project breakdown
STEPS:
1. starting from zero.
  *starting with WAN-Cloud and the WAN-Switch already established and configured in GNS3.
   *We added a FortiGate firewall, two switchs and a Windows 10 workstation to the Network.
     *We then provided connections for the network. 
3. Set up a virtual LAN interface utilizing PuTTY. The CLI inputs were as follows.
   a. conf sys int
   b. edit port2
   c. set allowaccess ping http https ssh
   d. set ip 10.128.0.1/24
   e. end
4. we then verified the configuration utilizing the following CLI input.
   a. show sys int port2
4.Configured the DHCP server for the LAN interface:
  a. conf sys dhcp server 
  b. edit 1
  c. set default-gateway 10.128.0.1
  d. set netmask 255.255.255.0
  e. set interface port2
  f. config ip-range
  g. edit 1
  h. set start-ip 10.128.0.100
  i. set end-ip 10.128.0.199
  j. next
  k. end
  l. next
  m. end
5. verified the configuration DHCP server utilizing the following command.
  a. show sys dhcp server 1
6. setting up a Win10 workstation and verifing it has leased a DHCP address from the LAN network.
   a. input ipconfig /all
  verify
  --a. valid ip range: 10.128.0.[100-199]/24
  --b. gateway: 10.128.0.1
  --c. DHCP server: 10.128.0.1
7. Tested the network connectivity by pinging remote destinations.
  --a. ping 10.128.0.1 (LAN) should be successful
  --b. ping 8.8.8.8 (WAN) should fail 
  --c. ping google.com (DNS) should fail
8. connecting to the GUI utilizing the Win10 browser.
  --a. in the browser input "http://10.128.0.1/ it will pull up a login interface.
  --b. in put your admin username and password from when logged in to the firewall CLI.
  --c.select "later" when prompted you will be directed to the dashboard.
9. Make system changes on the dashboard.
  --*Hostname = firewall
  --*timezone = GMT -6:00 Central Time (US & Canada)
  --*setup device as local NTP server = enabled
  --*list on interfaces = port2, port4
  --*idle timeout = 60
  --*auto file system check = enabled
10. Backup the configuration.
11. Reboot the firewall.
12. Completing the Network setup. before starting backup the configuration one more time.
  a. configure network interfaces
    10.128.0.0/24 = LAN network
    10.128.99.0/24 = Guest network
    10.128.10.0/24 = DMZ network
  b. edit port1
      alias = WAN
      role = WAN 
  c. edit port2
      alias = LAN
      role = LAN
  d. edit port3
      alias = GUEST
      role = LAN
      ip/network mask = 10.128.99.1/24
      create address object matching subnet = enabled
      administrative access = ping, ssh
      dhcp server = enabled
      dns server = same as interface ip
    expand advance
      ntp server = local
  e. edit port4
      alias = DMZ
      role = DMZ
      ip/network mask = 10.128.10.1/24
      create address object matching subnet = enabled
      administrative access = ping
13. Enable DNS and configure DNS
  a. enable dns under: system > Feature visibility
      dns database = enabled
  b. configure DNS firewall setting under: Network > DNS
      dns server = specify
      primary dns server = 8.8.8.8
      secondary dns server = 1.1.1.1
14. configure network dns under: Network > DNS Servers
  a. LAN DNS
      Create New
      interface = LAN(port2)
      mode = recursive
  b. GUEST DNS
      Create New
      interface = GUEST(port3)
      mode = recursive
  c. DMZ DNS
      Create New
      interface = DMZ(port4)
      mode = recursive 
15. Creat service objects
  a. LAN services
      Create New > Service Group
      name = LAN-services-group
  b. DMZ services
      Create New > Service Group
      name = DMZ-services-group
      members = ALL_ICMP, FTP, RDP, SSH, Web access
16. Configure firewall Rules 
  a. LAN-to-WAN policy
      Create New
      name = LAN-to-WAN
      incoming interface = LAN
      outgoing interface = WAN
      source = port2 address
      destination = ALL
      service =  ALL
      NAT = enabled
  b. DMZ-to-WAN policy
      Create New
      name = DMZ-to-WAN
      incoming interface = DMZ
      outgoing interface = WAN
      source = port4 address
      destination = ALL
      service = ALL
      NAT = enabled
  c. LAN-to-DMZ policy
      Create New
      name = LAN-to-DMZ
      incoming interface = LAN
      outgoing interface = DMZ
      source = port2 address
      destination = port4 address
      service = DMZ-services-group
      NAT = disabled
  d. DMZ-to-LAN policy
      Create New
      name = DMZ-to-LAN
      incoming interface = DMZ
      outgoing interface = LAN
      source = port4 address
      destination = port2 address
      service = LAN-services-group
      NAT = disabled
  e. WAN-to-DMZ policy
      Create New
      name = WAN-to-DMZ
      incoming interface = WAN
      outgoing interface = DMZ
      source = ALL
      Destination = port4 address
      service = DMZ-services-group
      NAT = disabled
17. Backup the firewall configuration
