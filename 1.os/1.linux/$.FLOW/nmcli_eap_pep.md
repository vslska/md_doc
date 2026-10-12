
#### Создание подключения eap pep

```bash
nmcli connection add type wifi con-name "name" ifname wlan0 ssid "SSID" \
  wifi-sec.key-mgmt wpa-eap \
  802-1x.eap peap \
  802-1x.identity "login" \
  802-1x.password "password" \
  802-1x.phase2-auth mschapv2

```
