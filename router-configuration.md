# CLI router configuration

## Configure router password
```bash
ssh-keygen -f "/root/.ssh/known_hosts" -R "192.168.1.1" # removing old entry after a firmware upgrade
ssh root@192.168.1.1
passwd
```
[OpenWRT Luci installation](https://openwrt.org/docs/guide-user/luci/luci.essentials)
```bash
opkg update
opkg install luci
```

## Configure wifi
```bash
uci set wireless.radio0.country='DE'
uci set wireless.radio0.legacy_rates='1'
uci set wireless.default_radio0.ssid='5g'
uci set wireless.default_radio0.encryption='psk2'
uci set wireless.default_radio0.key='password'
uci set wireless.radio1.country='DE'
uci set wireless.radio1.legacy_rates='1'
uci set wireless.default_radio1.ssid='2g'
uci set wireless.default_radio1.encryption='psk2'
uci set wireless.default_radio1.key='password'
```
[Enabling Wireless Networks](http://trac.gateworks.com/wiki/OpenWrt/wireless#EnablingaWirelessRadio)
```bash
uci delete wireless.radio0.disabled
uci delete wireless.radio1.disabled
uci commit
/etc/init.d/network restart
```
## Configure OpenVPN
### [Base configuration](https://openwrt.org/docs/guide-user/services/vpn/openvpn/client)

```bash
opkg update
opkg install openvpn-openssl
# Configure firewall
uci del_list firewall.@zone[1].device="tun0"
uci add_list firewall.@zone[1].device="tun0"
uci commit firewall
service firewall restart
# Save VPN client profile
cat << "EOF" > /etc/openvpn/vpnclient.conf
PASTE_CONTENT_FROM_openvpn.ovpn
EOF
 
# Fix permissions
chmod "u=rw,g=,o=" /etc/openvpn/vpnclient.conf
 
# Install packages
opkg update
opkg install openvpn-openssl
 
# Configure VPN client
sed -i -e "
/^user/s/^/#/
\$a user nobody
/^group/s/^/#/
\$a group nogroup
/^dev/s/^/#/
\$a dev $(uci get firewall.@zone[1].device | sed -e "s/^.*\s//")
" /etc/openvpn/vpnclient.conf
service openvpn restart
# Save username/password credentials
cat << "EOF" > /etc/openvpn/vpnclient.auth
YOUR_VPN_USERNAME
YOUR_VPN_PASSWORD
EOF
 
# Fix permissions
chmod "u=rw,g=,o=" /etc/openvpn/vpnclient.auth
 
# Configure VPN client
sed -i -e "
/^auth-user-pass/s/^/#/
\$a auth-user-pass /etc/openvpn/vpnclient.auth
/^redirect-gateway/s/^/#/
\$a redirect-gateway def1 ipv6
" /etc/openvpn/vpnclient.conf
service openvpn restart

# Send ca.crt, client.crt and client.key to router
exit
cd /to_folder_containig_files/
scp client.crt root@192.168.1.1:/etc/openvpn/
scp client.key root@192.168.1.1:/etc/openvpn/
scp ca.crt root@192.168.1.1:/etc/openvpn/
ssh root@192.168.1.1
service openvpn restart
```
### [Configure OpenVPN Killswitch](https://openwrt.org/docs/guide-user/services/vpn/openvpn/extra)
```bash
uci -q delete firewall.vpn
uci set firewall.vpn="zone"
uci set firewall.vpn.name="vpn"
uci set firewall.vpn.input="REJECT"
uci set firewall.vpn.output="ACCEPT"
uci set firewall.vpn.forward="REJECT"
uci set firewall.vpn.masq="1"
uci set firewall.vpn.mtu_fix="1"
uci add_list firewall.vpn.device="tun0"
uci del_list firewall.@zone[1].device="tun0"
uci -q delete firewall.lan2vpn
uci set firewall.lan2vpn="forwarding"
uci set firewall.lan2vpn.src="lan"
uci set firewall.lan2vpn.dest="vpn"
uci set firewall.@forwarding[0].enabled="0"
uci commit firewall
service firewall restart
 
cat << "EOF" > /etc/openvpn/killswitch.sh
#!/bin/sh
if pgrep openvpn
then
uci set firewall.lan2wan.enabled="1"
/etc/init.d/openvpn stop &
else
uci set firewall.lan2wan.enabled="0"
/etc/init.d/openvpn start &
fi
/etc/init.d/firewall restart &
EOF
chmod "u=rwx,g=rx,o=rx" /etc/openvpn/killswitch.sh
```

## [Access DSL router through this one](https://simplebeian.wordpress.com/2014/03/12/accessing-your-modem-from-openwrt-router/)
```bash
uci del network.wan.proto
uci set network.wan.proto='static'
uci set network.wan.ipaddr='ip_in_dsl_router_ip_pool'
uci set network.wan.netmask='255.255.255.0'
uci set network.wan.metric='0'

uci add luci ifstate # =cfg090295
uci set luci.@ifstate[-1].interface='wan'
uci set luci.@ifstate[-1].ifname='eth0.2'
uci set luci.@ifstate[-1].bridge='false'
uci commit
/etc/init.d/network restart
```
*Don't forget to drop ip_in_dsl_router_ip_pool address in DSL router* 

*Reboot router for changes to take effect*
