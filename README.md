# Mini-NAS with Copyparty & Termux

Lightweight homemade NAS running on Android using Termux, Copyparty and Cloudflare Tunnel.

## Features

- Local private storage
- Public file sharing
- Remote access without port forwarding
- Low power consumption
- SSH remote management

## Prerequisites
* An Android device.
* [Termux](https://f-droid.org/pt_BR/) installed (Ensure you use the F-Droid version, not the deprecated Google Play version).

## 1. Environment Setup

```bash
termux-setup-storage
 
pkg update && pkg upgrade -y

pkg install python termux-api && python -m ensurepip && python -m pip install --user -U copyparty && { grep -qE 'PATH=.*\.local/bin' ~/.bashrc 2>/dev/null || { echo 'PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && . ~/.bashrc; }; }
echo $?

pkg install ffmpeg && python3 -m pip install --user -U pillow
```
## 2. Configuration Files
We will create two separate configuration files to separate our private local network from our public-facing tunnel.

Local Server Configuration (Secure)
This instance is strictly for local network access.

Create the configuration file:

```bash
nano .copyparty.conf
```

Paste the following basic configuration (adjust names, paths, and passwords as needed):

```bash
[global]
ban-pw: 6,60,1440
ban-404: 20,60,1440
xff-hdr: cf-connecting-ip
name: Your_Local_Server_Name
e2dsa, e2ts, z, qr

[accounts]
your_username: your_strong_password

# Use a forward slash (/) at the beginning for custom endpoint names
[/Local_Storage]
/storage/emulated/0/Your_Local_Path
accs:
  rwdma: your_username

```

Public Server Configuration (Internet Facing)
Create a second file named .public.conf Copy the structure from above, but point it to a different storage directory (like an external SD card) and avoid storing sensitive personal data here.

## 3. Launching the Servers
You will need to open 3 separate terminal sessions in Termux (swipe left-to-right from the edge of the screen and click "New Session").

Session 1: Start the Local Server

```bash
copyparty -c .copyparty.conf
```
Your local NAS is now running.

Session 2: Start the Public Server
Run the public instance on a separate port (8080):

```bash
copyparty -c .public.conf -p 8080
```

Session 3: Expose the Public Server via Cloudflare
Create a secure tunnel pointing to the local port we just opened:

```bash
cloudflared tunnel --url http://127.0.0.1:8080
```
Cloudflared will generate a public URL in the terminal. You can now access your segregated public server from anywhere in the world using that link, while your sensitive data remains locked safely in the local instance.

## 4. Extra Tips

If your device is far away or difficult to use directly, you can manage it remotely using OpenSSH.

Install and enable the SSH server:

```bash
pkg install openssh

sshd

sv-enable sshd

passwd
# Set your password
```
To connect from another computer or device on the same network:

```ssh username@device-ip -p 8022```

Example:

```ssh u0_a409@10.0.0.0 -p 8022```

You can get your current username using:

```whoami```

Just a practical guide on how to have a pocket Server.

