This guide provides a step-by-step walkthrough for setting up a home automation server on Windows 11 using Docker Desktop and Dockhand.

1. BIOS Configuration: AMD-V Activation
To run Docker and virtualized environments efficiently on a Ryzen 3 4300U, you must enable hardware virtualization.

Restart your computer and enter the BIOS/UEFI (usually by pressing F2, Del, or Esc).

Navigate to the Advanced or CPU Configuration menu.

Locate SVM Mode (Secure Virtual Machine) or AMD-V and set it to Enabled.

Save and Exit.

2. System Identity (Windows 11)
Set a clear hostname for your machine to make it easily identifiable on your network.

Go to Settings > System > About.

Click Rename this PC.

Suggested names: VCellColony or MustelidMainframe.

Restart Windows to apply the changes.

3. Network: DHCP Reservation (Technicolor Router)
To ensure your server's IP address remains constant, configure a fixed IP via your Technicolor router.

Access your router's web interface (typically 192.168.1.1).

Find the Local Network or DHCP settings.

Locate your device in the list and set the Lease Duration to "Static" or "Fixated" to ensure the IP never expires.

4. Docker Desktop Installation
Docker provides the containerization environment needed for your apps.

Download: Get the installer from the Official Docker Desktop Page.

Prerequisites: Ensure WSL2 (Windows Subsystem for Linux) is installed and enabled, as it is the high-performance backend for Docker on Windows 11.

Configuration: * During installation, ensure the WSL2 option is checked.

Once installed, open Docker Desktop and go to Settings > General.

Enable Start Docker Desktop when you log in to ensure services are available after a reboot.

5. Deploying Dockhand
Dockhand is used to manage your container stacks. Create a folder for your project and use the following docker-compose.yaml file:

YAML
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
Run the command docker-compose up -d in your terminal to start the service.

6. Accessing Dockhand
Open your browser and navigate to http://localhost:3000.

Add a new environment pointing to your localhost to begin managing containers.

7. Deploying Home Assistant via Dockhand
Inside the Dockhand UI, create a new stack and paste the following configuration for Home Assistant:

YAML
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
      - TZ=Europe/Berlin # Replace with your actual Timezone
Note: Ensure you define the ${TZ} variable or replace it directly with your timezone (e.g., Europe/Berlin).

8. Final Setup & Future Stacks
Home Assistant: Once deployed, open http://localhost:8123 to complete your smart home setup.

Expanding: You can continue adding stacks through Dockhand, such as Pi-hole for network-wide ad-blocking.