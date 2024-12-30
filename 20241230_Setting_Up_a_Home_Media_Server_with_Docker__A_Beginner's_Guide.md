Setting up a home media server allows you to centralize, manage, and stream your media library efficiently—all from the comfort of your home. This guide walks you through configuring a powerful home media server using Docker with a sample setup.

---

## Why a Home Media Server?

A home media server offers numerous benefits:
- **Centralized Library**: Store all your movies, TV shows, and other media in one place.
- **Customizable Setup**: Tailor your setup to your preferences with open-source tools.
- **Personalized Access**: Stream your media from any device within your network or even remotely.

---

## Prerequisites

Before diving in, ensure you have:
1. A home server, such as an old PC or NAS, running a Linux-based OS.
2. Docker and Docker Compose installed on your server.
3. Basic knowledge of terminal commands to navigate and execute commands.
4. Sufficient storage to house your media library.

---

## Step-by-Step Setup

### Prepare Your Server

Update your system and ensure Docker and Docker Compose are installed and running. Familiarize yourself with your server’s directory structure, as this will help keep the configuration files and media files organized.

### Organize Your Project Directory

Create a structured directory for your server project. Include subdirectories for configurations, downloads, and media storage. This organization ensures easier management and avoids clutter.

### Define Your Docker Setup

Set up a `docker-compose.yml` file that includes the services you’ll use for your media server. In this guide, we use the following tools:

- **qBittorrent**: A torrent client for managing downloads.
- **Prowlarr**: A tool for managing and aggregating torrent trackers.
- **Sonarr**: A TV show management and automation tool.
- **Radarr**: A movie management and automation tool.
- **Jellyfin**: A media server application to organize and stream your content.

```yaml
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    volumes:
      - ./configs/qbittorrent:/config
      - downloads:/downloads # Central downloads directory
    ports:
      - 8080:8080 # qBittorrent WebUI
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
    networks:
      - default

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:develop
    container_name: prowlarr
    hostname: prowlarr
    ports:
      - 9696:9696 # Prowlarr WebUI
    volumes:
      - ./configs/prowlarr:/config
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
    networks:
      - default

  sonarr:
    image: lscr.io/linuxserver/sonarr
    container_name: sonarr
    hostname: sonarr
    ports:
      - 8989:8989 # Sonarr WebUI
    volumes:
      - ./configs/sonarr:/config
      - downloads:/downloads # Central downloads directory
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
    networks:
      - default

  radarr:
    image: lscr.io/linuxserver/radarr
    container_name: radarr
    hostname: radarr
    ports:
      - 7878:7878 # Radarr WebUI
    volumes:
      - ./configs/radarr:/config
      - downloads:/downloads # Central downloads directory
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
    networks:
      - default

  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    hostname: jellyfin
    ports:
      - 8096:8096 # Jellyfin WebUI
    volumes:
      - ./configs/jellyfin:/config
      - downloads:/media # Access media files from downloads directory
    environment:
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
    networks:
      - default

volumes:
  downloads:

networks:
  default:
    driver: bridge
```

Configure the services to share volumes (such as downloads and media) and expose the necessary ports for accessing their WebUIs.

### Start Your Media Server

Run Docker Compose to launch the services. This will download the required images and initialize the containers. Ensure that each service starts without issues by checking their logs.

---

## Access and Configure Your Applications

Once your containers are running, you can access the WebUIs for each service in your web browser using your server’s IP address and the respective ports. Configure each application based on your preferences:

- In qBittorrent, set up your download preferences and directories.
- In Prowlarr, add and manage trackers to streamline your downloads.
- In Sonarr and Radarr, configure your TV shows and movie libraries, respectively.
- In Jellyfin, organize your media collection and set up user profiles for streaming.

Ensure all applications point to the shared directories, such as the downloads folder, to enable seamless integration between tools.

---

## Tips for Optimal Performance

1. Use external storage drives or RAID setups to ensure sufficient space and redundancy for your media.
2. Set up a VPN if you plan to access your media server remotely, ensuring secure connections.
3. Explore additional tools like Bazarr for managing subtitles or Portainer for monitoring your Docker containers.
4. Regularly update your Docker images to benefit from the latest features and security patches.

---

## Wrapping Up

By following this guide, you’ve successfully set up a home media server that centralizes and automates your media management and streaming needs. With tools like Jellyfin, Sonarr, and Radarr working together, you’ll enjoy a smooth and efficient experience. Feel free to customize this setup further based on your unique requirements. Happy streaming!
