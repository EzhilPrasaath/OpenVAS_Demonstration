# Metasploitable 2 Installation on Kali Linux Using Docker

This guide covers installing Docker, running the Metasploitable 2 container, checking its services, and connecting to its legacy SSH service from Kali Linux.

> **Safety:** Metasploitable 2 is intentionally vulnerable. Keep the published ports bound to `127.0.0.1`. Do not expose this lab to the Internet or an untrusted network.

## 1. Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
```

Start Docker and enable it at boot:

```bash
sudo systemctl enable --now docker
```

Check Docker:

```bash
sudo systemctl status docker
```

## 2. Download the Metasploitable 2 Image

```bash
sudo docker pull tleemcjr/metasploitable2
```

Verify the image is available:

```bash
sudo docker images
```

## 3. Remove an Existing Container (If Needed)

Docker cannot create a second container with a name already in use. Check existing containers:

```bash
sudo docker ps -a
```

If a container named `metasploitable2` already exists and you want to replace it, remove it:

```bash
sudo docker rm -f metasploitable2
```

This removes the container, not the downloaded image.

## 4. Start the Container

Run the following command as one block:

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

The host ports are bound to localhost only.

| Kali host port | Container port | Service |
|---:|---:|---|
| 21 | 21 | FTP |
| 2222 | 22 | SSH |
| 23 | 23 | Telnet |
| 80 | 80 | HTTP |
| 3306 | 3306 | MySQL |

SSH uses host port `2222` to avoid conflicting with Kali's own SSH service on port `22`.

## 5. Verify the Container

```bash
sudo docker ps
```

Check startup logs if needed:

```bash
sudo docker logs metasploitable2
```

If the container is not running, inspect its state:

```bash
sudo docker ps -a
```

## 6. Scan the Services

Run Nmap against the local lab:

```bash
sudo nmap -sV -p 21,2222,23,80,3306 127.0.0.1
```

Note that the SSH service is exposed on host port `2222`, not `22`.

Open the HTTP service in a browser:

```text
http://127.0.0.1
```

Or test it from the terminal:

```bash
curl http://127.0.0.1
```

## 7. Connect to SSH

The old SSH server may offer only legacy host-key algorithms. If the normal connection fails with `no matching host key type found`, try enabling RSA host keys for this connection only:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa \
    -o PubkeyAcceptedAlgorithms=+ssh-rsa \
    -p 2222 msfadmin@127.0.0.1
```

Common Metasploitable 2 credentials are:

```text
Username: msfadmin
Password: msfadmin
```

If SSH then reports a key-exchange error, you can try this additional legacy option for this lab connection:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa \
    -o PubkeyAcceptedAlgorithms=+ssh-rsa \
    -o KexAlgorithms=+diffie-hellman-group14-sha1 \
    -p 2222 msfadmin@127.0.0.1
```

These options apply only to the command; do not weaken your global SSH configuration. Legacy algorithms are less secure and should be used only for this isolated training target.

## 8. Start Metasploit

In a separate Kali terminal:

```bash
msfconsole
```

For example, search for modules related to a service version found during your scan:

```text
search vsftpd 2.3.4
```

Validate the target and module details before testing. Only test this lab or systems for which you have explicit authorization.

## 9. Stop, Restart, or Remove the Lab

Stop the container:

```bash
sudo docker stop metasploitable2
```

Start the existing container again:

```bash
sudo docker start metasploitable2
```

Remove the container:

```bash
sudo docker rm -f metasploitable2
```

The image remains available after removing the container. To remove the image too:

```bash
sudo docker rmi tleemcjr/metasploitable2
```

## 10. Troubleshooting

### Container name is already in use

Remove or rename the existing container before creating another one:

```bash
sudo docker rm -f metasploitable2
```

Then rerun the command in **Section 4**.

### A host port is already in use

Check listening ports:

```bash
sudo ss -tulpn | grep -E ':(21|22|23|80|2222|3306)\b'
```

Change the host-side port if necessary. For example, `-p 127.0.0.1:8080:80` makes the web service available at `http://127.0.0.1:8080`.

### SSH still fails

Confirm the container is running and the port mapping exists:

```bash
sudo docker ps
```

Then review the container logs:

```bash
sudo docker logs metasploitable2
```

The exact SSH compatibility options depend on the algorithms offered by the server.

## References

- Docker image: https://hub.docker.com/r/tleemcjr/metasploitable2
- Rapid7 Metasploitable 2 documentation: https://docs.rapid7.com/metasploit/metasploitable-2/
- Rapid7 Metasploitable 2 Exploitability Guide: https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/
