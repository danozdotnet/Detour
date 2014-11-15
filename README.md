![Detour Icon](../images/icon.png?raw=true)

Detour
======

Easily route devices on your LAN through different VPNs and Interfaces on your [EdgeMAX](http://www.ubnt.com/edgemax/edgerouter-lite/) router.

Detour was created so I could easily switch some of my media devices to the US versions of Netflix and Amazon Prime (I live in Canada).  A friend uses it to route his AppleTV through Canada, USA, or Europe so he can bypass the geographic restrictions of NHL GameCenter.  It can also be used to selectively route different devices through a different WAN connection or VPN.

###Screenshot
Below is a screenshot of the Web Interface once it's been added to my home screen and launched as an app.
![Detour Screenshot](../images/screenshot.png?raw=true)

##How It Works
Detour is a PHP script that allows you to easily manage which route to the internet a particular device on your network will take.  It does this by adding the IP addresses of these devices to an address group.  You then create firewall rules to route the members of these groups through the different interfaces.

##Installation and Setup
First you need to create the interface to route the data through.  If this is a VPN you need to create the VPN client.  If this is just another WAN port, you probably don't need to do anything.  Next you have to setup a routing table and firewall rule to route data through that interface based on an address group.  Lastly, add the modify firewall to your LAN interface.

####Adding a PPTP VPN Client Interface
Create the VPN Client.  Here we're using *pptpc0* as the interface name.

	edit interfaces pptp-client pptpc0
	set server-ip **VPN-SERVER-IP.COM**
	set user-id **USERNAME**
	set password **PASSWORD**
	set description "USA VPN"
	set default-route none
	set require-mppe
	exit

Enable NAT masquerade for the interface.  Here we are using rule number 5004.  You can use the next rule number available on your system, as long as it's greater than 5000.

	edit service nat rule 5004
	set description "Masquerade for pptpc0"
	set outbound-interface pptpc0
	set type Masquerade
	exit

####Creating a Routing Table for the New Interface
If you already have a table 1 use the next available table number.

	set protocols static table 1 interface-route 0.0.0.0/0 next-hop-interface pptpc0

####Creating the Firewall Rules
First we need to create the address group.
	
	set firewall group address-group vpn_usa

Next we create the firewall modify rule to route the data through the table above if the source IP address is in the address group.  

	edit firewall modify detour rule 10 <- Increment this number for additional interfaces
	set description "Detour route to US VPN pptpc0"
	set source group address-group vpn_usa    <- Use the group name you created above
	set modify table 1       <- Use the table number you used above
	exit

####Add the Firewall Modify rule to your LAN Interface
You only need to do this once.  Here were usign *eth0* as the LAN interface.  Replace that with your LANs interface or bridge name.

	set interfaces ethernet eth0 firewall in modify detour

####Commit and Save

	commit
	save

##Installing Detour

	cd /config
	curl -Lk https://github.com/TravisCook/Detour/archive/master.tar.gz | tar xz
	mv Detour-master detour
	cd detour
	sudo ./install.sh
	
####Configuring Detour
There are two configuration files for Detour.  Enter your address group names into the group_list.conf file.  In our example above that would be *vpn_usa*.

	vi group_list.conf

Enter the IP addresses and device names for the device's you'd like to use with Detour.  These IP addresses should be statically assigned to each device.

	vi ip_list.conf


