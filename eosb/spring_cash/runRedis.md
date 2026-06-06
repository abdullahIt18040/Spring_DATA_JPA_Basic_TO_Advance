### Option 1: Run Redis with WSL (Recommended)
```
Redis officially supports Linux, so using the Windows Subsystem for Linux (WSL) is the easiest and most reliable approach.

Open PowerShell as Administrator:

wsl --install
if install before then 
wsl

Restart your computer if prompted.
Open Ubuntu from the Start menu.

Install Redis:

sudo apt update
if not install before then 
sudo apt install redis-server -y

Start Redis:

sudo service redis-server start

Verify it's running:

redis-cli ping

Expected output:

PONG

You can learn more from the official Redis docs: Redis Installation Guide
```
