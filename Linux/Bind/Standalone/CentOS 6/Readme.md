#Installation#
```
yum install bind bind-utils bind-chroot
chkconfig --add bind
chkconfig --level 345 bind on
cd /var/named/chroot/
mv /var/named/
```
#Configuration#
move data files to chroot directory
```
find /var/named/ -mindepth 1 -maxdepth 1 -not -iname "chroot" -exec mv {} /var/named/chroot{} \;
mv /etc/named* /var/named/chroot/etc/
ln -sf /var/named/chroot/etc/named.conf /etc/named.conf
```

reset permissions
```
chown -R named:named /var/named/chroot/var/named/
chown -R named:named /var/named/chroot/etc/
```

#Partitioning#
Whether or not you're going to be using your BIND service for both Internal and External DNS, it's good practice to go ahead and partition it for that, it's very easy. For partitioning, you'll create an ACL with your internal addresses (plus localhost). You wrap your zone files within "views", an internal view and an external view. The different views use your ACL to determine which zone files to read from for a DNS query response based on the client's IP address. You can review the sample config from my own DNS server to see it in action.

#Integration with DHCP#
If you have DHCPD running on a linux box and wish to push DDNS to your BIND server, there is a little bit more config to do. First you should refer to DHCP's configuration folder on how to do the first half of the configuration.
```
vim /var/named/chroot/etc/named.conf
```

paste this line near the top, outside of any declaration block
```
include "/var/named/chroot/etc/rndc.key";
```
Now for each zone that will receive DDNS updates from Bind, put this as their allow-update setting
```
allow-update { key rndc-key; };
```
Now cycle both BIND and DHCP and you should be good to go.