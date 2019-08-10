CLI configuration:

```bash
ssh-keygen -f "/root/.ssh/known_hosts" -R "192.168.1.1"
passwd
```
[OpenWRT Luci installation](https://openwrt.org/docs/guide-user/luci/luci.essentials)
```bash
opkg update
opkg install luci
```
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
