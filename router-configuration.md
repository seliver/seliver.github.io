# H1 CLI configuration:

## Configuring password
```bash
ssh-keygen -f "/root/.ssh/known_hosts" -R "192.168.1.1" # removing old entry after a firmware upgrade
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
[OpenWRT OpenVPN](https://openwrt.org/docs/guide-user/services/vpn/openvpn/client)
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
