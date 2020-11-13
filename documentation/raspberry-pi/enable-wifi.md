Copy `wpa_supplicant.conf` and then enter:
```
wpa_cli -i wlan0 reconfigure
sudo rfkill unblock all
sudo ip link set wlan0 up
```
