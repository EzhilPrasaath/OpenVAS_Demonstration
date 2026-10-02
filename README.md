# OpenVAS (GVM) Local Installation & Setup Guide on Kali Linux

This guide provides a step-by-step walkthrough for installing **OpenVAS (GVM)** — a full-featured network vulnerability scanner supported by Greenbone — on Kali Linux.

It includes:

- System preparation and installation
- Initial GVM setup
- PostgreSQL collation version mismatch fixes
- Manual PostgreSQL database creation
- Administrator account creation
- GVM service management
- Installation verification
- Accessing the Greenbone Security Assistant (GSA)

---

## 1. System Preparation & Installation

Before installing GVM, update your Kali Linux system to reduce the possibility of dependency conflicts.

### Update and upgrade the system

```bash
sudo apt update && sudo apt full-upgrade -y
```

### Install the OpenVAS/GVM package

```bash
sudo apt install openvas -y
```

### Run the initial GVM setup

```bash
sudo gvm-setup
```

The setup process may take some time because GVM needs to configure its database, services, certificates, scanner components, and vulnerability feeds.

---

## 2. Resolving PostgreSQL Collation Mismatch

Sometimes `gvm-setup` may fail while creating or initializing the `gvmd` database after a `glibc` or system-library update.

A common error is:

```text
database has a collation version mismatch
```

This happens when PostgreSQL detects that the collation version stored when a database was created differs from the currently installed system collation version.

### Refresh the collation version

Run:

```bash
sudo -u postgres psql -c "ALTER DATABASE postgres REFRESH COLLATION VERSION;"
```

Then:

```bash
sudo -u postgres psql -c "ALTER DATABASE template1 REFRESH COLLATION VERSION;"
```

### Re-run the GVM PostgreSQL database creation script

```bash
sudo runuser -u postgres -- /usr/share/gvm/create-postgresql-database
```

If the script completes successfully, continue with the administrator-account setup.

> **Note:** Do not manually delete PostgreSQL databases unless you are certain they are disposable. Existing GVM installations may contain configuration and scan-related data.

---

## 3. Creating the Administrator Account

If the GVM database was recreated manually, the administrator account created during the original `gvm-setup` process may no longer exist.

You can create a new administrator account explicitly:

```bash
sudo runuser -u _gvm -- gvmd --create-user=admin --password='MySecurePassword123!'
```

### Use your own strong password

Replace:

```text
MySecurePassword123!
```

with a strong password of your choice.

For example:

```bash
sudo runuser -u _gvm -- gvmd --create-user=admin --password='YOUR_STRONG_PASSWORD'
```

> **Security:** Do not commit a real password to GitHub. If this command is included in a public repository, use a placeholder such as `YOUR_STRONG_PASSWORD`.

---

## 4. Starting and Restarting GVM Services

If the Greenbone web daemon (`gsad`) is stuck or an old process is occupying port `9392`, terminate the orphaned `gsad` process before restarting the services.

### Stop stuck `gsad` processes

```bash
sudo killall gsad
```

If no `gsad` process is running, you may see a message indicating that no process was found. That is not necessarily a problem.

### Restart the GVM services

```bash
sudo systemctl restart gvmd ospd-openvas gsad
```

### Check the service status

You can verify the individual services with:

```bash
sudo systemctl status gvmd
sudo systemctl status ospd-openvas
sudo systemctl status gsad
```

Press `q` to exit the `systemctl status` screen.

---

## 5. Verifying the Installation

Use the GVM diagnostic tool to check whether the installation is configured correctly:

```bash
sudo gvm-check-setup
```

The tool checks important components such as:

- GVM services
- PostgreSQL configuration
- Database connectivity
- Certificates
- Scanner configuration
- Vulnerability feeds
- Required permissions

Wait for the diagnostic script to finish and review any reported errors or warnings.

A successful check should indicate that the GVM installation is ready to use.

---

## 6. Accessing the Greenbone Security Assistant

Once the GVM services are running, open a web browser on the Kali machine and navigate to:

```text
https://127.0.0.1:9392
```

You may receive a browser warning because the local GVM installation commonly uses a certificate that is not trusted by the browser.

Proceed to the GVM web interface if you trust the local installation.

Log in with:

```text
Username: admin
Password: YOUR_STRONG_PASSWORD
```

---

## 7. Useful GVM Commands

### Check GVM setup

```bash
sudo gvm-check-setup
```

### Restart the manager

```bash
sudo systemctl restart gvmd
```

### Restart the OpenVAS scanner

```bash
sudo systemctl restart ospd-openvas
```

### Restart the web interface

```bash
sudo systemctl restart gsad
```

### Restart all three services

```bash
sudo systemctl restart gvmd ospd-openvas gsad
```

### Check service status

```bash
sudo systemctl status gvmd
sudo systemctl status ospd-openvas
sudo systemctl status gsad
```

### Check whether port 9392 is listening

```bash
sudo ss -lntp | grep 9392
```

---

## 8. Troubleshooting

### Problem: `gvm-setup` fails

Run:

```bash
sudo gvm-check-setup
```

Look for the first reported error and resolve that issue before repeatedly running the setup command.

---

### Problem: PostgreSQL collation mismatch

If you see:

```text
database has a collation version mismatch
```

try:

```bash
sudo -u postgres psql -c "ALTER DATABASE postgres REFRESH COLLATION VERSION;"
sudo -u postgres psql -c "ALTER DATABASE template1 REFRESH COLLATION VERSION;"
```

Then:

```bash
sudo runuser -u postgres -- /usr/share/gvm/create-postgresql-database
```

---

### Problem: Port 9392 is already in use

Check which process is using the port:

```bash
sudo ss -lntp | grep 9392
```

If `gsad` is the process holding the port:

```bash
sudo killall gsad
```

Then restart it:

```bash
sudo systemctl restart gsad
```

---

### Problem: GVM services are not running

Check:

```bash
sudo systemctl status gvmd
sudo systemctl status ospd-openvas
sudo systemctl status gsad
```

Then restart them:

```bash
sudo systemctl restart gvmd ospd-openvas gsad
```

Finally, run:

```bash
sudo gvm-check-setup
```

---

## 9. Quick Installation Summary

For a normal installation, the basic sequence is:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install openvas -y
sudo gvm-setup
sudo gvm-check-setup
```

If PostgreSQL reports a collation mismatch:

```bash
sudo -u postgres psql -c "ALTER DATABASE postgres REFRESH COLLATION VERSION;"
sudo -u postgres psql -c "ALTER DATABASE template1 REFRESH COLLATION VERSION;"
sudo runuser -u postgres -- /usr/share/gvm/create-postgresql-database
```

Create the administrator account if required:

```bash
sudo runuser -u _gvm -- gvmd --create-user=admin --password='YOUR_STRONG_PASSWORD'
```

Restart the services:

```bash
sudo systemctl restart gvmd ospd-openvas gsad
```

Verify:

```bash
sudo gvm-check-setup
```

Then open:

```text
https://127.0.0.1:9392
```

---

## 10. Security Notes

- Use a strong, unique GVM administrator password.
- Never commit real passwords, API keys, certificates, or private keys to GitHub.
- Use `YOUR_STRONG_PASSWORD` or another placeholder in documentation.
- Keep Kali Linux and GVM packages updated.
- Run vulnerability scans only against systems you own or have explicit authorization to test.
- Review `gvm-check-setup` output carefully if the installation reports warnings or errors.

---

## License / Disclaimer

This guide is intended for **authorized security testing, vulnerability assessment, and educational use**.

Only scan systems and networks for which you have explicit permission.
