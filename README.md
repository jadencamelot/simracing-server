# Assetto Corsa Server Config

Dockerised private server(s) for Assetto Corsa.

## AC Racing Server - `assetto-server-manager`

From: [JustaPenguin/assetto-server-manager](https://github.com/JustaPenguin/assetto-server-manager)

Web interface: link TBC

## AC Freeroam Server - `AssettoServer`

From: [compujuckel/AssettoServer](https://github.com/compujuckel/AssettoServer)

No web interface

## Set up from scratch

After connecting to server via SSH (root user):

```bash
# Create non-root user
adduser -m user
mkdir /home/user/.ssh
cp /root/.ssh/authorized_keys /home/user/.ssh/
chown -R user:user /home/user/.ssh
chmod 700 /home/user/.ssh
chmod 600 /home/user/.ssh/authorized_keys
groupadd docker
usermod -aG sudo user
usermod -aG docker user
```

On client:

```bash
# Clone this repo, then copy to server
git clone git@github.com:jadencamelot/simracing-server.git
scp -r simracing-server ac-server:/home/user/simracing-server
```

Connect to server (non-root user):

```bash
# Lock down ssh
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Install docker and docker-compose
sudo apt install docker-compose

# Set up correct permissions for server-data folder
# (must be owned by 1000:1000 for docker image to work)
cd /home/user/simracing-server/assetto-server-manager
mkdir server-data
sudo chown -R 1000:1000 server-data
```

Install Assetto Corsa Server (from Steam)

```bash
./steamcmd.sh  # Launch SteamCMD in docker container

# Inside container:
./steamcmd.sh  # Run steamcmd

Steam> force_install_dir /mnt/assetto
Steam> login <SERVER_USERNAME>
Steam> @sSteamCmdForcePlatformType windows
Steam> app_update 302550
Steam> quit
```

Get TLS cert from Let's Encrypt

```bash
# Open Port 80, to allow certbot to do its verification
sudo ufw allow http

# Run Certbot
sudo snap install --classic certbot
sudo certbot certonly --standalone --register-unsafely-without-email

# Ensure domain name is passed to docker container, so it can find the file path where cert was saved
echo 'SERVER_DOMAIN=my.domain.name' > /home/user/simracing-server/assetto-server-manager/.env
```

Run server

```bash
# Download image and run container
docker-compose up --detach

# Check the logs to see if it worked
docker-compose logs | less -R
```
