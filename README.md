### Version 0.1, 2024-05-20 (first version)

To build a RasPiAI from scratch with this script, you need a Raspberry Pi 4B or 5 with 8 GB of RAM.

## How to prepare the Raspberry Pi: 🥧

1. **Installing Raspberry Pi OS:**
   - Download Raspberry Pi OS Lite as an operating System from [Raspberry PI](https://www.raspberrypi.com/software/)
   - The RasPiAI can be accessed via SSH, username: RasPiAI, password: RasPiAI$. You can choose your own username and password if you like.
     
2. **Raspberry Pi OS Lite updaten und upgraden:**
   - `sudo apt update && sudo apt upgrade`
   - `sudo reboot`
      
3. **Enlarge the partition of the Mico-SD card:**
    - `sudo raspi-config`
    - at "6 Advanced Options" -> "A1 Expand Filesystem"

4. **Setting the regional setting, timezone and WLAN:**
     - `sudo raspi-config`
     - at "5 Localisation Options" -> "L1 Locale", "L2 Timezone" and "L4 WLAN Country"
     - choose the settings that suit your country
     - reboot the system
  
5. **Installing Docker:**
   - `curl -sSL https://get.docker.com -o install-docker.sh`
   - `bash install-docker.sh`

6. **Inclusion of the current user in the Docker user group:**
   - `sudo usermod -aG docker $USER`
   - `newgrp docker`

7. **Verify the Docker installation:**
   - `docker run hello-world`
  
8. **Delete Docker container:**
   - `docker ps`
   - `docker rm` -> enter the Container ID of the "hello-world" container
  
## How to make your Raspberry Pi to an Acess Point: 📡

1. **Installation of required programmes:**
   - `sudo apt install dnsmasq hostapd iptables`
   - `sudo apt install dhcpcd5`

2. **Stop the packages from running:**
   - `sudo systemctl stop hostapd`
   - `sudo systemctl stop dnsmasq` 

3. **Configuring the WLAN interface:**
    - `sudo nano /etc/dhcpcd.conf`
    - Delete the contents of the file and replace them with the following lines of code:
```
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```
4. **Restart DHCP Client Daemon:**
    - `sudo systemctl restart dhcpcd`
    
5. **Check whether the Ethernet interface (eth0) and the WLAN adapter (wlan0) are working and present:**
    - `ip l`

6. **Set up DHCP server and DNS cache:**
   - `sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf_alt`
   - `sudo nano /etc/dnsmasq.conf `
   - The following minimum configuration must be entered here: 
  
```
# DHCP server active for WLAN interface
interface=wlan0

# DHCP server not active for existing network
no-dhcp-interface=eth0

# IPv4-Adressbereich und Lease-Time
dhcp-range=192.168.4.2,192.168.4.50,255.255.255.0,24h
```

7. **Check DHCP server and DNS cache and put into operation:**
   - `dnsmasq --test -C /etc/dnsmasq.conf` -> "syntax check OK"
   - `sudo systemctl restart dnsmasq`
   - `sudo systemctl status dnsmasq` -> You should see "active (running)" and "enabled" in green.  
   - `sudo systemctl enable dnsmasq`

8. **Set up WLAN-AP host:**
   - `sudo nano /etc/hostapd/hostapd.conf`
   - You can replace **ssid** with your own username and **wpa_passphrase** with your own password.
```
interface=wlan0
driver=nl80211
ssid=RasPiAI
hw_mode=g
channel=7
wmm_enabled=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=RasPiAI$
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

9. **Changing the read rights to the file:**
   - `sudo chmod 600 /etc/hostapd/hostapd.conf`

10. **Check WLAN-AP host configuration and put into operation:**
   - `sudo hostapd -dd /etc/hostapd/hostapd.conf` -> With "Ctrl + C" you can end the running hostapd instance if required.
   - If the following messages appear, everything is in the green zone:
```
...
wlan0: interface state COUNTRY_UPDATE->ENABLED
...
wlan0: AP-ENABLED
...
```

11. **Let "hostapd" start as a daemon in the background:**
    - `sudo nano /etc/default/hostapd`
    - Remove the # in front of "DAEMON_CONF=""
    - Then add the following parameters:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

12. **Start-up of "hostapd":**
    - `sudo systemctl unmask hostapd`
    - `sudo systemctl start hostapd`
    - `sudo systemctl enable hostapd`
   
13. **Check status of "hostapd":**
    - `sudo systemctl status hostapd` -> You should see "active (running)" and "enabled" in green.

14. **Check that the WLAN interface is not managed by another network management tool:**
    - `sudo systemctl stop NetworkManager`
    - `sudo systemctl stop wpa_supplicant`

14.  **Configure routing and NAT for the Internet connection:**
     - `sudo nano /etc/sysctl.conf`
     - remove the # symbol before the following line of code `net.ipv4.ip_forward=1`
     - `sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`
     - `sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`
     - `sudo nano /etc/rc.local`
     - Here we enter the following code line before the line with "exit 0": `iptables-restore < /etc/iptables.ipv4.nat`
   
15. **Reboot the Raspberry Pi:**
    -`sudo reboot`

16. **Check WLAN, router and DHCP/DNS functions:**
    - `sudo systemctl status hostapd` ->  You should see "active (running)" and "enabled" in green.
    - `ps ax | grep hostapd`
    - `sudo systemctl status dnsmasq` ->  You should see "active (running)" and "enabled" in green.
    - `ps ax | grep dnsmasq`

### Troubleshooting: 

There is a possibility that the Raspberry Pi will lose its ability to function as an access point after a restart. This means that WLAN is no longer provided. If this is the case, you must proceed as follows: 
   - Check the status of the Raspberry Pi with `iw dev wlan0 info`. The problem exists when there is "managed" next to the type instead of "AP".
   - Configure automatic restart of hostapd. To do this, "always" must be added to "Restart=" under `sudo nano /lib/systemd/system/hostapd.service` or `sudo nano 
     /etc/systemd/system/hostapd.service`.
   - Restart hostapad with the following commands: `sudo systemctl unmask hostapd`, `sudo systemctl start hostapd`, `sudo systemctl enable hostapd`.      
   - Reboot the Raspberry Pi with `sudo reboot`.
   - Check the status of the Raspberry Pi again with `iw dev wlan0 info`. You should see "AP" next to the type. The WLAN with the name RasPiAI should now be active.

## How to install ollama 🦙 and open-webui:

**Install Ollama & open-webui together:**
   - `docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama`    

## How to configure the Firewall: 🔥

1. **Allow incoming connections on port 80:**
   - `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`
  
2. **Save Firewall Rules:**
   - `sudo apt-get install iptables-persistent`
   - `sudo iptables-save | sudo tee /etc/iptables/rules.v4 > /dev/null `

## How to set up an automatic login: 🤖

1. `sudo mkdir -p /etc/systemd/system/getty@tty1.service.d/`
   
2. `sudo nano /etc/systemd/system/getty@tty1.service.d/autologin.conf`

3. Paste the following content into the file and save and close the file at the end.
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin RasPiAI --noclear %I 38400 linux
```

4. `sudo systemctl daemon-reload`

5. `sudo systemctl enable getty@tty1.service`

6. `sudo reboot`

## Recommended LLM models 👍:

The models listed here are primarily aimed at those users who are looking for small LLM models that can communicate in other languages as well as English.

- [Phi-3 Mini ]([https://ollama.com/library/phi3]) - for general communication
- [Starcoder2 ]([https://ollama.com/library/starcoder2]) - for coding

## Starting your LLMs 🍾:

- In a browser of your choice, enter the IP address **http://192.168.4.1:3000/**.
- Sign up with any user name and email address. The first person to log in after start-up is automatically given administrator rights. This person can now authorise other users. 

## Resources at your fingertips: ⌨️

1. [Raspberry Pi Cheatsheet](https://opensource.com/sites/default/files/gated-content/raspberry_pi_cheatsheet_from_opensource.com_.pdf)
2. [Docker Cheatsheet](https://collabnix.com/docker-cheatsheet/)
3. [VIM Cheatsheet](https://devhints.io/vim)
