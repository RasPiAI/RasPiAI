### Version 0.1 (pre-release), 2024-02-24 (first version)

To build a RasPiAI from scratch with this script, you need a Raspberry Pi 4B or 5 with 8 GB of RAM.

## How to prepare the Raspberry Pi: 🥧

1. Installing **Raspberry Pi OS**:
   - Download Raspberry Pi OS Lite as an operating System from [Raspberry PI](https://www.raspberrypi.com/software/)
    - The BerryPlexus can be reached via SSH, username: RasPiAI, password: RasPiAI$.

2. Raspberry Pi OS Lite **updaten und upgraden**:
   - `sudo apt update && sudo apt upgrade`
   - `sudo reboot`
      
3. Enlarge the partition of the Mico-SD card:
    - `sudo raspi-config`
    - at "6 Advanced Options" -> "A1 Expand Filesystem"

4. Setting the regional setting, timezone and WLAN:
     - `sudo raspi-config`
     - at "5 Localisation Options" -> "L1 Locale", "L2 Timezone" and "L4 WLAN Country"
     - choose the settings that suit your country
     - reboot the system
  
5. Installing **Docker**:
   - `curl -sSL https://get.docker.com -o install-docker.sh`
   - `bash install-docker.sh`

6. Inclusion of the current user in the Docker user group:
   - `sudo usermod -aG docker $USER`
   - `newgrp docker`

7. Verify the Docker installation with:
   - `docker run hello-world`
  
8. Installing **Docker Compose**:
    - Download [Asset docker-compose-linux-armv7](https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-armv7) for Raspberry Pi 4 with `wget https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-armv7 -O docker-compose`
    - `chmod +x docker-compose`
    - `sudo mv docker-compose /usr/local/bin`

9. Install a Texteditor like **Vim**:
   - `sudo apt install vim`
  
## How to make your Raspberry Pi to an Acess Point: 📡

1. Installation of required programme
   - `sudo apt install dnsmasq hostapd iptables`
   - `sudo apt install dhcpcd5`

2. Stop the packages from running
   - `sudo systemctl stop hostapd`
   - `sudo systemctl stop dnsmasq` 

3. Configuring the WLAN interface
    - `sudo nano /etc/dhcpcd.conf`
    - Delete the contents of the file and replace them with the following lines of code:
```
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```
4. Restart DHCP Client Daemon
    - `sudo systemctl restart dhcpcd`
    
5. Check whether the Ethernet interface (eth0) and the WLAN adapter (wlan0) are working and present.
    - `ip l`

6. Set up DHCP server and DNS cache
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

7. Check DHCP server and DNS cache and put into operation
   - `dnsmasq --test -C /etc/dnsmasq.conf` -> "syntax check OK"
   - `sudo systemctl restart dnsmasq`
   - `sudo systemctl status dnsmasq` -> You should see "active (running)" and "enabled" in green.  
   - `sudo systemctl enable dnsmasq`

8. Set up WLAN-AP host
   - `sudo nano /etc/hostapd/hostapd.conf`
   - You can replace ssid with your own username and wpa_passphrase with your own password.
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

9. Changing the read rights to the file
   - `sudo chmod 600 /etc/hostapd/hostapd.conf`

10. Check WLAN-AP host configuration and put into operation
   - `sudo hostapd -dd /etc/hostapd/hostapd.conf` -> With "Ctrl + C" you can end the running hostapd instance if required.
   - If the following messages appear, everything is in the green zone:
```
...
wlan0: interface state COUNTRY_UPDATE->ENABLED
...
wlan0: AP-ENABLED
...
```

11. So that the "hostapd" starts as a daemon in the background
    - `sudo nano /etc/default/hostapd`
    - Remove the # in front of "DAEMON_CONF=""
    - Then add the following parameters:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

12. Start-up of "hostapd"
    - `sudo systemctl unmask hostapd`
    - `sudo systemctl start hostapd`
    - `sudo systemctl enable hostapd`
   
13. Check status of "hostapd
    - `sudo systemctl status hostapd` -> You should see "active (running)" and "enabled" in green.

14. Check that the WLAN interface is not managed by another network management tool
    - `sudo systemctl stop NetworkManager`
    - `sudo systemctl stop wpa_supplicant`

14.  Configure routing and NAT for the Internet connection
     - `sudo nano /etc/sysctl.conf`
     - remove the # symbol before the following line of code `net.ipv4.ip_forward=1`
     - `sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`
     - `sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`
     - `sudo nano /etc/rc.local`
     - Here we enter the following code line before the line with "exit 0": `iptables-restore < /etc/iptables.ipv4.nat`
   
15. Reboot the Raspberry Pi
    -`sudo reboot`

16. Check WLAN, router and DHCP/DNS function
    - `sudo systemctl status hostapd` ->  You should see "active (running)" and "enabled" in green.
    -` ps ax | grep hostapd`
    - `sudo systemctl status dnsmasq`
    - `ps ax | grep dnsmasq`

### Troubleshooting: 

There is a possibility that the Raspberry Pi will lose its ability to function as an access point after a restart. This means that WLAN is no longer provided. If this is the case, you must proceed as follows: 
   - Check the status of the Raspberry Pi with `iw dev wlan0 info`. If the status shows "managed" and not "AP", then the problem exists.
   - Configure automatic restart of hostapd. To do this, "always" must be added to "Restart=" under `sudo nano /lib/systemd/system/hostapd.service` or `sudo nano 
     /etc/systemd/system/hostapd.service`.
   - Deactivate the NetworkManager. To do this, execute the commands `sudo systemctl stop NetworkManager` and `sudo systemctl disable NetworkManager`.
   - Activate and start hostapd again to reconfigure the services. To do this, you must enter the following in sequence: `sudo systemctl stop hostapd`, `sudo systemctl unmask 
     hostapd`, `sudo systemctl enable hostapd` and `sudo systemctl start hostapd`. 
   - Reboot the Raspberry Pi with `sudo reboot`.  

## How to install ollama 🦙 and open-webui together using docker🐳:

1. Running Ollama in a Docker container
   - `docker run -d --name ollama -p 11434:11434 -v ollama_volume:/root/.ollama ollama/ollama:latest`    
2. Check if Ollama is running or not:
   - `docker ps`
> CONTAINER ID: 0a5142e31b9c  IMAGE: 0a5142e31b9c  COMMAND: ollama/ollama:latest  CREATED: 1 minute ago  STATUS:Up 1 minute              PORTS: 0.0.0.:11434->11434/tcp,  :::11434->11434/tcp  NAMES: ollama      
   - `curl http://localhost:11434`
> Ollama is running

3. Running open-webui
   - `git clone https://github.com/ollama-webui/ollama-webui`
   - `cd ollama-webui`
   - `vim docker-compose.yml` -> [docker-compose.yml](https://github.com/BerryPlexus/BerryPlexus/blob/main/docker-compose.yml)
   - `docker stop ollama`
   - `docker compose up -d`
  
### Troubleshooting: 

There is a possibility that problems may occur when docker compose is executed despite the docker container being stopped. If this is the case, the Ollama container must be deleted. 
   - `rm ollama`
  
## How to configure the Firewall: 🔥

1. Allow incoming connections on port 80
   - `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`
  
2. Save Firewall Rules
   - `sudo apt-get install iptables-persistent`
   - `sudo iptables-save | sudo tee /etc/iptables/rules.v4 > /dev/null `

## How to set up an automatic login: 🤖

1. `sudo nano /etc/systemd/system/getty@tty1.service.d/autologin.conf`

2. Paste the following content into the file and save and close the file at the end.
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin RasPiAI --noclear %I 38400 linux
```
3. `sudo systemctl daemon-reload`

4.` sudo systemctl enable getty@tty1.service`

5.` sudo reboot`


## Resources at your fingertips: ⌨️

1. [Raspberry Pi Cheatsheet](https://opensource.com/sites/default/files/gated-content/raspberry_pi_cheatsheet_from_opensource.com_.pdf)
2. [Docker Cheatsheet](https://collabnix.com/docker-cheatsheet/)
3. [VIM Cheatsheet](https://devhints.io/vim)



- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...Configure routing and NAT for the Internet connection


- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
