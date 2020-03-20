## Ubuntu LTS (18.04) Multi User Remote Desktop Server

This is a fork of [danielguerra69/ubuntu-xrdp](https://github.com/danielguerra69/ubuntu-xrdp) with some customizations, tagged as `rcarmo/ubuntu-xrdp`.

The original message had a full implementation of multi-user `xrdp` with `xorgxrdp` with copy/paste and audio support via `pulseaudio`, plus the ability to reconnect to previous sessions. I've added `MP3` and `AAC` support to save audio bandwidth when using the standard Microsoft Remote Desktop clients (may not work reliably - I'm still experimenting, mostly with Mac and iOS).

## Confirmed Working:

* Multi-display support on Mac clients (tested with three monitors on the client, RDP works across all of them on all clients)
* High-bandwidth audio with the Mac App Store Remote Desktop Client (no settings to pick audio bandwidth)
* (Garbled) compressed audio with Jump Dekstop for macOS

## Usage

Start the rdp server
(WARNING: `--shm-size 1g` is required to ensure Firefox/Chrome doesn't crash)

```bash
docker run -d --name uxrdp --hostname terminalserver --shm-size 1g -p 3389:3389 -p 2222:22 rcarmo/ubuntu-xrdp
```
> *Note*: if you already have an RDP server on 3389 change `-p <my-port>:3389`.  `-p 2222:22` is for `ssh` access (`ssh -p 2222 ubuntu@<docker-ip> )

Connect with your remote desktop client to the `docker` server. If your client doesn't provide a native authentication prompt, you will get a graphical one inside the RDP session. Use the `Xorg` session configuration (leave as is), then enter your username and password.

## Sample User

There is a sample user with `sudo` rights:

```
Username: ubuntu
Password: ubuntu
```

You can set a `PASSWORDHASH` externally:

First create a password hash

```bash
openssl passwd -1 'newpassword'
```

Run the container with your hash:

```bash
docker run -d -e PASSWORDHASH='$1$Cm8EQjXg$7dJeRsw6TLvgxsl3.pBRZ1'
```

You can change your password inside the RDP session from the terminal:

```bash
passwd
```

## Adding New Users

No configuration is needed for new users. Just do

```bash
docker exec -ti uxrdp adduser mynewuser
```

After this the new user can login.

## Adding New Services

To make sure standard processes are working `supervisor` is installed.
The location for adding services to start automatically is `/etc/supervisor/conf.d`

### Example: Add `mysql` as a service

```bash
apt-get -yy install mysql-server
echo "[program:mysqld] \
command= /usr/sbin/mysqld \
user=mysql \
autorestart=true \
priority=100" > /etc/supervisor/conf.d/mysql.conf
supervisorctl update
```

## Volumes

This image uses two volumes:
1. `/etc/ssh/` holds the `sshd` host keys and config
2. `/home/` holds the `ubuntu/` default user home directory

When bind-mounting `/home/`, make sure it contains a folder `ubuntu/` with proper permissions, otherwise login will block/fail:

```
mkdir -p ubuntu
chown 999:999 ubuntu
```

## Installing Additional Packages During Build

The Dockerfile has support for the build argument ADDITIONAL_PACKAGES to install additional packages during build. 

Either pass that in with `--build-arg` during `docker build` or add it 
as `args` in your `docker-compose.override.yml` and run `docker-compose build`.

## Using `docker-compose`

```bash
git clone https://github.com/rcarmo/ubuntu-xrdp.git
cd ubuntu-xrdp/
vi docker-compose.override.yml # if you want to override any default value
docker-compose up -d
```
