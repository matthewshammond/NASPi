# Setup Pi and NASPi Hardware
## Update and Upgrade Pi
```
sudo apt update
sudo apt full-upgrade -y
```
## Install script to control fan
```
sudo apt install -y vim git curl pigpio python3-pigpio python3-smbus unattended-upgrades apt-transport-https
git clone https://github.com/geekworm-com/x-c1.git 
cd x-c1
sudo chmod +x *.sh
sudo bash install.sh
```
## Add alias for soft shutdown
```
echo "alias xoff='sudo /usr/local/bin/x-c1-softsd.sh'" >> ~/.bashrc 
```
## Configure unattended-upgrades
```
sudo dpkg-reconfigure --priority=low unattended-upgrades
```
## Restart Pi
`sudo reboot`

# Clone respository
- Clone repository to home directory
```
git clone https://github.com/matthewshammond/NASPi.git
```

- Move plex from NASPi directory to HOME directory
```
mv NASPi/plex .
```

# Setup Plex Media Server
## Add plex key and repository to sources list
```
echo deb [signed-by=/usr/share/keyrings/plex-archive-keyring.gpg] https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
curl https://downloads.plex.tv/plex-keys/PlexSign.key | gpg --dearmor | sudo tee /usr/share/keyrings/plex-archive-keyring.gpg >/dev/null
```

## Update sources and install plex media server
```
sudo apt update
sudo apt install -y plexmediaserver
```

## Configure audio to work through HDMI
- edit /boot/config.txt
```sudo vim /boot/config.txt```

- change dtoverlay to read
```vc4-fkms-v3d```

## Install window display
### Install the X Window System (X11)
- Install xserver-xorg 
```sudo apt install -y --no-install-recommends xserver-xorg```

- Install xinit
```sudo apt install -y --no-install-recommends xinit```

- Install x11-xserver-utils 
```sudo apt install -y --no-install-recommends x11-xserver-utils```

### Install Chromium and kiosk dependencies
- Install chromium-browser
```sudo apt install chromium-browser```

- Install the kiosk dependencies
```sudo apt install matchbox-window-manager xautomation unclutter```

### Configure xinit to start on boot
- Add following command to .bashrc
```echo "xinit /home/pi/kiosk -- vt$(fgconsole)" >> ~/.bashrc```



# Setup RASAP to Create Travel Router
- Install RASPAP
```curl -sL https://install.raspap.com | bash```

## Configure Network Settings
- Stop network services
```
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```

- Modify DHCPCD
```sudo vim /etc/dhcpcd.conf```
```
interface wlan0
    static ip_address=10.13.71.1/24
    nohook wpa_supplicant
```
```sudo systemctl restart dhcpcd```

- Modify HOSTAPD
```sudo nano /etc/hostapd/hostapd.conf```
```
interface=wlan0
driver=nl80211

hw_mode=a
channel=36
ieee80211n=0
wmm_enabled=0
macaddr_acl=0
ignore_broadcast_ssid=0

auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
beacon_int=100

ssid=Discovery
wpa_passphrase=password
country_code=US
```

```
sudo vim /etc/default/hostapd
DAEMON_CONF="/etc/hostapd/hostapd.conf"
sudo vim /etc/init.d/hostapd
DAEMON_CONF=/etc/hostapd/hostapd.conf
```

- Modify DNSMASQ
```sudo vim /etc/dnsmasq.conf```
```
interface=wlan0       # Use interface wlan0
server=1.1.1.1       # Use Cloudflare DNS
dhcp-range=10.13.71.10,10.13.71.20,12h # IP range and lease time
```

- Modify SYSCTL
```
sudo vim /etc/sysctl.conf
net.ipv4.ip_forward=1
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```

- Forward all traffic form wlan1 to wlan0
```sudo iptables -t nat -A POSTROUTING -o wlan1 -j MASQUERADE```

- Save rules to be used on reboot
```sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"```

- Edit RC.local to set iptables on reboot
```sudo vim /etc/rc.local```
- Above `exit 0` add:
```iptables-restore < /etc/iptables.ipv4.nat```

## Finalize Configuration
```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
sudo service dnsmasq start
```

## Reboot
```sudo reboot```

## Defaults:
```
IP address: 10.3.141.1
Username: admin
Password: secret
DHCP range: 10.3.141.50 — 10.3.141.255
SSID: raspi-webgui
Password: ChangeMe
```
