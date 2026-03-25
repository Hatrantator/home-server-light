This guide provides a step-by-step walkthrough for setting up a home automation server on Windows 11 using Docker Desktop and Dockhand.

## Table of Contents
1. [BIOS Configuration](#1-bios-configuration-amd-v-activation)
2. [System Identity](#2-system-identity-windows-11)
3. [Network: DHCP Reservation](#3-network-dhcp-reservation-technicolor-router)
4. [Docker Desktop Installation](#4-docker-desktop-installation)
5. [Deploying Dockhand](#5-deploying-dockhand)
6. [Accessing Dockhand](#6-accessing-dockhand)
7. [Deploying Home Assistant](#7-deploying-home-assistant-via-dockhand)
8. [Final Setup & Future Stacks](#8-final-setup--future-stacks)

---

## 1. BIOS Configuration: AMD-V Activation
To run Docker and virtualized environments efficiently on a **Ryzen**, you must enable hardware virtualization.

* **Enter BIOS/UEFI**: Restart your computer and press `F2`, `Del`, or `Esc`.
* **Navigate**: Go to the **Advanced** or **CPU Configuration** menu.
* **Enable SVM**: Locate **SVM Mode** (Secure Virtual Machine) or **AMD-V** and set it to **Enabled**.
* **Finish**: Save and Exit.

---

## 2. System Identity (Windows 11)
Set a clear hostname for your machine to make it easily identifiable on your network.

* Navigate to **Settings** > **System** > **About**.
* Select **Rename this PC**.
* **Suggested names**: `VCellColony` or `MustelidMainframe`.
* **Restart** Windows to apply the changes.

---

## 3. Network: DHCP Reservation (Technicolor Router)
To ensure your server's IP address remains constant, configure a fixed IP via your Technicolor router.

1.  Access your router's web interface (typically `192.168.1.1`).
2.  Find the **Local Network** or **DHCP** settings.
3.  Locate your device in the list and set the **Lease Duration** to **"Static"** or **"Fixated"** to ensure the IP never expires.

---

## 4. Docker Desktop Installation
Docker provides the containerization environment needed for your apps.

* **Download**: Get the installer from the [Official Docker Desktop Page](https://www.docker.com/products/docker-desktop/).
* **Prerequisites**: Ensure **WSL2** (Windows Subsystem for Linux) is installed and enabled, as it is the high-performance backend for Docker on Windows 11.
* **Configuration**: 
    * During installation, ensure the **WSL2** option is checked.
    * Open Docker Desktop and go to **Settings** > **General**.
    * Enable **Start Docker Desktop when you log in** to ensure services are available after a reboot.

---

## 5. Deploying Dockhand
Dockhand is used to manage your container stacks. Create a folder for your project and use the following `docker-compose.yaml` file:

```yaml
services:
  dockhand:
    image: fnsys/dockhand:latest
    container_name: dockhand
    restart: unless-stopped
    user: "0:0"
    ports:
      - "3000:3000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - dockhand_data:/app/data

volumes:
  dockhand_data:
```
Run the command `docker-compose up -d` in your terminal to start the service.

---

## 6. Accessing Dockhand
* Open your browser and navigate to `http://localhost:3000`.
* Add a new **environment** pointing to your **localhost** to begin managing containers.

---

## 7. Deploying Home Assistant via Dockhand
Inside the Dockhand UI, create a **new stack** and paste the following configuration for **Home Assistant**:
```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
    environment:
      - TZ=${TZ} # Replace with your actual Timezone
```
Ensure you define the `${TZ}` variable or replace it directly with your actual timezone (e.g., `Europe/Berlin`).

---

## 8. Final Setup & Future Stacks
* **Home Assistant:** Once deployed, open `http://localhost:8123` to complete your smart home setup.
* **Expanding:** You can continue adding stacks through Dockhand, such as **Pi-hole** for network-wide ad-blocking.
```yaml
# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      # DNS Ports
      - "53:53/tcp"
      - "53:53/udp"
      # Default HTTP Port
      - "80:80/tcp"
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "443:443/tcp"
      # Uncomment the below if using Pi-hole as your DHCP Server
      #- "67:67/udp"
      # Uncomment the line below if you are using Pi-hole as your NTP server
      #- "123:123/udp"
    environment:
      # Set the appropriate timezone for your location from
      # https://en.wikipedia.org/wiki/List_of_tz_database_time_zones, e.g:
      TZ: 'Europe/London'
      # Set a password to access the web interface. Not setting one will result in a random password being assigned
      FTLCONF_webserver_api_password: 'correct horse battery staple'
      # If using Docker's default `bridge` network setting the dns listening mode should be set to 'ALL'
      FTLCONF_dns_listeningMode: 'ALL'
    # Volumes store your data between container upgrades
    volumes:
      # For persisting Pi-hole's databases and common configuration file
      - './etc-pihole:/etc/pihole'
      # Uncomment the below if you have custom dnsmasq config files that you want to persist. Not needed for most starting fresh with Pi-hole v6. If you're upgrading from v5 you and have used this directory before, you should keep it enabled for the first v6 container start to allow for a complete migration. It can be removed afterwards. Needs environment variable FTLCONF_misc_etc_dnsmasq_d: 'true'
      #- './etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      # See https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
      # Required if you are using Pi-hole as your DHCP server, else not needed
      - NET_ADMIN
      # Required if you are using Pi-hole as your NTP client to be able to set the host's system time
      - SYS_TIME
      # Optional, if Pi-hole should get some more processing time
      - SYS_NICE
    restart: unless-stopped
```
