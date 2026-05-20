# Mini-NAS with Copyparty & Termux

This guide provides a step-by-step process for turning an unused Android device into a secure Mini-NAS using [Copyparty](https://github.com/9001/copyparty) and Termux. 

To mitigate risks, this setup uses a dual-server architecture:
1. **Local Server:** Highly secure, restricted to your local network (static IP), intended for sensitive personal files.
2. **Public Server:** Exposed to the internet via a Cloudflare tunnel, intended only for non-sensitive data sharing. Storage is segregated using external SD cards.

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

## 4. Extra tips

If your phone is at a distance from you, or has any problem that makes it difficult to use without connecting it to a computer, I recommend that you use OpenSSH for local and remote generation of the same device, for this I will leave a mini explanation of how to do this, I hope you like it, thank you.

```bash
pkg install openssh
sshd
sv-enable sshd
passwd # enter your password
```
On your computer or other device that allows you to connect, type the following command ``` ssh u0_a409@10.0.0.0 -p 8022 ``` remember to use the IP of your device and the user collects himself with the ``` whoami ``` command running on the device that provides the server 

That and just a basic server, which I've been using for a while and decided to share how the configuration I used was, I highly recommend reading the copyparty project wiki because and a project of extra quality and complexity, I'm happy if I helped you, and remember, read it.

by Christopher


## Future Roadmap
