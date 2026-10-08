# Metasploitable 2 in Docker (Kali Linux)

This guide provides step-by-step instructions for downloading, installing, and running a vulnerable **Metasploitable 2** environment inside a Docker container on **Kali Linux** for penetration testing practice.

---

## 📋 Prerequisites

Before starting, ensure your Kali Linux package database is updated:
```bash
sudo apt update
```

---

## 🚀 Installation & Setup

### 1. Install and Enable Docker
If Docker is not already installed on your Kali system, install it and enable the background daemon service:

```bash
# Install Docker
sudo apt install docker.io -y

# Start and enable the Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. Pull the Metasploitable 2 Image
Download the community-maintained Metasploitable 2 Docker image:

```bash
sudo docker pull tleemcjr/metasploitable2
```

### 3. Run the Container
Launch the container in detached mode (`-d`). 

> 💡 **Note on Port 2222:** Kali Linux often runs an SSH server on port `22` natively. To prevent conflicts, the container's SSH port (`22`) is mapped to the host's port `2222`. A `tail` command is appended to keep the container running indefinitely in the background.

```bash
sudo docker run -d \
  --name metasploitable2 \
  -p 80:80 \
  -p 21:21 \
  -p 2222:22 \
  -p 23:23 \
  -p 3306:3306 \
  tleemcjr/metasploitable2 \
  bash -c "/bin/services.sh && tail -f /dev/null"
```

---

## 🛠️ Management & Verification

### Check Container Status
Verify that the container is actively running and see its port mappings:
```bash
sudo docker ps
```

### Stopping and Starting the Environment
When you are done practicing, you can manage the container without deleting it:

* **To stop the container:**
  ```bash
  sudo docker stop metasploitable2
  ```
* **To start the container again:**
  ```bash
  sudo docker start metasploitable2
  ```

---

## 🔍 Troubleshooting

### ❌ Error: "address already in use" (Port Conflict)
If you see an error indicating port `22` or another port is already bound, it means a native service on your Kali host is using it.
* **Fix:** Change the host mapping in the `docker run` command (e.g., change `-p 22:22` to `-p 2222:22`).

### ❌ Error: "Conflict. The container name is already in use"
If the container crashes or is stopped but you cannot rerun the setup command because the name is taken:
* **Fix:** Remove the dead container before running it again:
  ```bash
  sudo docker rm metasploitable2
  ```

---

## 🛡️ Mitigation & Security Best Practices

Metasploitable 2 is **intentionally vulnerable** and highly insecure. Exposing it incorrectly can allow unauthorized users on your local network (or the internet) to compromise your host system. Implement these mitigations to stay secure:

### 1. Bind Ports to Localhost Only (Crucial)
By default, `-p 80:80` binds port 80 to `0.0.0.0`, meaning anyone on your local Wi-Fi or Ethernet network can attack your container. 
* **Mitigation:** Force Docker to only expose ports to your local machine by prepending `127.0.0.1:` to your port flags:
  ```bash
  sudo docker run -d --name metasploitable2 \
    -p 127.0.0.1:80:80 \
    -p 127.0.0.1:21:21 \
    -p 127.0.0.1:2222:22 \
    -p 127.0.0.1:23:23 \
    -p 127.0.0.1:3306:3306 \
    tleemcjr/metasploitable2 \
    bash -c "/bin/services.sh && tail -f /dev/null"
  ```

### 2. Isolate via Custom Docker Network
Instead of using the default bridge network, isolate your lab environment.
* **Mitigation:** Create a dedicated network for your penetration testing labs:
  ```bash
  sudo docker network create pentest-net
  ```
  Then append `--network pentest-net` when running your containers to prevent them from interacting with your default host infrastructure.

### 3. Avoid Running as Root (Container Escape Prevention)
If a containerized service is exploited, an attacker might look for container breakout vulnerabilities to gain root access to your Kali host machine.
* **Mitigation:** Never run this container with the `--privileged` flag. Ensure your Docker daemon configurations are kept up to date to patch known runtime escape vulnerabilities.

### 4. Stop the Container When Done
Leaving a vulnerable server running in the background is a constant risk.
* **Mitigation:** Always run `sudo docker stop metasploitable2` as soon as your testing session is finished.

---

## 🎯 Verification Scan (Reconnaissance)

To confirm all services are up and ready to target, run an aggressive version detection scan using **Nmap** from your Kali host terminal:

```bash
nmap -p 21,23,80,2222,3306 -sV 127.0.0.1
```

* **Web UI Access:** Open a browser on your Kali machine and navigate to `http://127.0.0.1` or `http://localhost`.
* **Default Credentials:** `msfadmin` / `msfadmin`
