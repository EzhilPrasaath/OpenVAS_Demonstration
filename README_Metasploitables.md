# Metasploitable 2 Docker Setup on Kali Linux

## 1. Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
```

## 2. Start and Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Check Docker:

```bash
sudo systemctl status docker
```

## 3. Pull the Metasploitable 2 Image

```bash
sudo docker pull tleemcjr/metasploitable2
```

Verify:

```bash
sudo docker images
```

## 4. Run Metasploitable 2

```bash
sudo docker run -it --rm   --name metasploitable2   -p 80:80   -p 21:21   -p 22:22   -p 23:23   -p 3306:3306   tleemcjr/metasploitable2   /bin/services.sh && bash
```

### Exposed Ports

| Port | Service |
|------|---------|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 80 | HTTP |
| 3306 | MySQL |

## 5. Verify the Container

Open another terminal:

```bash
sudo docker ps
```

The container should appear as:

```text
metasploitable2
```

## 6. Find the Container IP

```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' metasploitable2
```

Because the ports are mapped to the Kali host, you can access the services through:

```text
127.0.0.1
```

## 7. Test the Target

Run an Nmap scan:

```bash
sudo nmap -sC -sV -p- 127.0.0.1
```

Open the web service:

```text
http://127.0.0.1
```

## 8. Default Credentials

```text
Username: msfadmin
Password: msfadmin
```

## 9. Stop the Container

```bash
sudo docker stop metasploitable2
```

Because the container was started with `--rm`, it will be removed automatically when stopped.

## 10. Start Again

Run the same Docker command again:

```bash
sudo docker run -it --rm   --name metasploitable2   -p 80:80   -p 21:21   -p 22:22   -p 23:23   -p 3306:3306   tleemcjr/metasploitable2   /bin/services.sh && bash
```

> **Warning:** Metasploitable 2 is intentionally vulnerable. Use it only in an isolated lab environment and never expose it to the Internet or an untrusted network.
