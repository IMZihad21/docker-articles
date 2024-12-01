Hosting applications on a Virtual Private Server (VPS) using Docker and Nginx is a powerful, scalable, and flexible approach. This article walks you through setting up Docker, configuring an Nginx reverse proxy using Nginx Proxy Manager, managing your Docker containers with Portainer, and deploying a sample application. This article is based on **Debian** based OS system.

---

## Step 1: Install Docker on Your VPS

### 1. Update System Packages:

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Cleanup system for fresh Docker install:

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### 3. Install using the [official apt repository](https://docs.docker.com/engine/install/debian/#install-using-the-repository):

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 4. Configure Docker as a non-root user and verify:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER

docker --version
```

---

## Step 2: Create a Docker Network

To allow seamless communication between containers, create a shared Docker network named `nginx_proxy`:

```bash
docker network create nginx_proxy
```

---

## Step 3: Set Up Nginx Proxy Manager

Nginx Proxy Manager simplifies reverse proxy configuration with a web-based interface.

### 1. Prepare Directory:

```bash
mkdir -p ~/managers/nginx && cd ~/managers/nginx
```

### 2. Create a Docker Compose File: Save the following content in `docker-compose.yml`:

```yaml
services:
  app:
    container_name: nginx
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
networks:
  default:
    name: nginx_proxy
    external: true
```

### 3. Deploy Nginx Proxy Manager:

```bash
docker compose up -d
```

### 4. Access the Admin Panel: Navigate to `http://<your-vps-ip>:81` and log in with the default credentials:

- Email: `admin@example.com`
- Password: `changeme`

Update your credentials immediately.

---

## Step 4: Set Up Portainer for Docker Management

Portainer provides an intuitive interface to manage Docker containers.

### 1. Prepare Directory:

```bash
mkdir -p ~/managers/portainer && cd ~/managers/portainer
```

### 2. Create a Docker Compose File: Save the following content in `docker-compose.yml`:

```yaml
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:latest
    restart: unless-stopped
    ports:
      - '9000:9000'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer_data:/data
networks:
  default:
    name: nginx_proxy
    external: true
```

### 3. Deploy Portainer:

```bash
docker compose up -d
```

### 4. Access the Portainer UI: Navigate to `http://<your-vps-ip>:9000` and follow the setup wizard.

---

## Step 5: Configure Nginx Proxy Manager UI with a Subdomain

1. Log in to NPM by opening `http://<your-vps-ip>:81` in your browser.
2. Add a new Proxy Host: Enter a subdomain you control for accessing the NPM UI (e.g., npm.example.com) in the Domain Name field.
3. Forward Hostname/IP: Enter the Portainer container's hostname, which in this case is `nginx`.
4. Set the Forward Port to 81.
5. Check Enable Websockets.
6. Enable SSL.
7. Check Enable SSL, use the Let's Encrypt certificate, add your email address, agree to the terms, and check Force SSL to ensure all traffic uses HTTPS.
8. Save the Proxy Host.
9. Access NPM securely via the subdomain by visiting `https://npm.example.com`.

**Optional**: After this, you can disable port 80 and 81 which uses http so that only https request using 443 ports can pass through securely. This is mainly for restricting unsecured ports.

---

## Step 6: Configure Portainer with Nginx Proxy Manager

Using NPM, we will set up a reverse proxy to access Portainer via a domain or subdomain.

1. Log in to NPM: Open `https://npm.example.com` or `http://<your-vps-ip>:81` if port is open and log in.
2. Add a New Proxy Host:
3. Domain Name: Enter a domain or subdomain you control (e.g., portainer.example.com).
4. Forward Hostname/IP: Enter the Portainer container's hostname, which in this case is `portainer`.
5. Forward Port: Set the port to 9000.
6. Enable Websockets: Check this option.
7. SSL Configuration: Check Enable SSL. Use the Let's Encrypt certificate (add your email).
8. Save the Proxy Host: After saving, the reverse proxy will be active.
9. Access Portainer via Domain: Visit `https://portainer.example.com`.

---

## Conclusion

By following these steps, you’ve deployed a robust hosting environment on your VPS using Docker. Nginx Proxy Manager allows easy reverse proxy setup, while Portainer provides seamless container management. This setup ensures scalability and secure access for hosting multiple applications.