# Disable selinux
setenforce 0

vi /etc/sysconfig/selinux
	- disabled

# Enable IP forwarding
vi /etc/sysctl.conf
	- net.ipv4.ip_forward = 1

# BTRFS Compression
chattr -R +c /etc/ /home /var /opt
btrfs fi defrag -r -c /

# Network Configuration
# WAN
PEERDNS=no

# Package Section
yum remove firewalld
yum install strongswan quagga bind-chroot bind-utils httpd mod_ssl dhcp tcpdump tmux screen iproute iptables-services iptables-utils ntp ddclient linux-igd iptraf-ng radvd ndisc6

# Services
systemctl start iptables.service
systemctl enable iptables.service

# iptables:
# em1 = LAN
# em2 = WAN
iptables -A FORWARD -i em1 -j ACCEPT
iptables -A FORWARD -o em1 -j ACCEPT
iptables -t nat -A POSTROUTING -o em2 -j MASQUERADE

service iptables save

# Bind9-chroot
systemctl stop named.service
systemctl disable named.service
systemctl start named-chroot.service
systemctl enable named-chroot.service

# DHCPD
systemctl enable dhcpd.service
systemctl start dhcpd.service

# NTP
systemctl enable ntpd.service
systemctl start ntpd.service

# upnpd
Theres a bug in the RPM for upnpd and you can fix it by doing the steps below 
Refernce: https://bugzilla.redhat.com/show_bug.cgi?format=multiple&id=903740

vi /etc/sysconfig/upnpd
EXTIFACE="em2"
INTIFACE="em1"
vi /usr/lib/systemd/system/upnpd.service
	ExecStart=/usr/sbin/upnpd -f $EXTIFACE $INTIFACE
systemctl stop upnpd.service
systemctl disable upnpd.service
systemctl enable upnpd.service
systemctl start upnpd.service
