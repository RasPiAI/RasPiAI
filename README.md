### Version 0.1 (pre-release), 2024-02-24 (first version)

## How to prepare the Raspberry Pi: :pie:

1. Installing **Raspberry Pi OS**:
   - Download Raspberry Pi OS Lite as an operating System from [Raspberry PI](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2023-12-11/2023-12-11-raspios-bookworm-arm64-lite.img.xz?_gl=1*ub0mk4*_ga*MTk1NDg2OTQzMC4xNzA3ODE5Nzc2*_ga_22FD70LWDS*MTcwODc3NTIyMC4yLjEuMTcwODc3NTMwOC4wLjAuMA..)
    - The BerryPlexus can be reached via SSH, username: BerryPlexus, password: BerryPlexus$.

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

## How to install ollama 🦙 and open-webui together using docker:🐳

## Resources at your fingertips: ⌨️

1. `[Raspberry Pi Cheatsheet](https://opensource.com/sites/default/files/gated-content/raspberry_pi_cheatsheet_from_opensource.com_.pdf)`
2. `[Docker Cheatsheet](https://collabnix.com/docker-cheatsheet/)`
3. `[VIM Cheatsheet](https://devhints.io/vim)`



- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
