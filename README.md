# Setting Up Raspberry Pi


### Sources
https://www.raspberrypi.org/documentation/installation/installing-images/README.md

https://www.raspberrypi.org/documentation/remote-access/ssh/README.md

https://www.raspberrypi.org/documentation/configuration/wireless/headless.md

### Preparing SD Card

Use Raspberry Pi Imager to flash RaspberryPi OS onto SD card. The Lite version does not include the desktop GUI, so use that if you are setting the raspberry pi up headless.

To enable ssh, create empty file called `.ssh` at root level on your SD card before the first boot.

To automatically connect to a wifi network, add a wpa_supplicant.conf file at root level before first boot with the following:

  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1
  country=<Insert 2 letter ISO 3166-1 country code here>

  network={
   ssid="<Name of your wireless LAN>"
   psk="<Password for your wireless LAN>"
  }

### First boot

Ssh into the raspberry pi using the username "pi" and password "raspberry".

# Securing Raspberry Pi

### Sources

https://www.raspberrypi.org/documentation/raspbian/updating.md

https://www.raspberrypi.org/documentation/configuration/security.md

https://www.youtube.com/watch?v=ukHcTCdOKrc

### Change password for pi user
Run `sudo raspi-config`, go to “System Options”, and then “Password”

### Set up ssh

Either create empty file named “ssh” at root level on your SD card before the first boot, or run `sudo raspi-config`, then go to “Interface Options” and then “SSH”.

### Create a new administrative superuser account

Run `sudo adduser <account_name>`, then `sudo gpasswd -a <account_name> adm`, and then `sudo gpasswd -a <account_name> sudo`.

Finally, check the new user is capable of logging in to the raspberrypi using a new terminal and is able to use sudo by running `sudo whoami`.

### Lock the pi account

We could delete the pi accound instead of locking it, but some software still relies on the pi account to work.

Log onto the administrative superuser account set up in the previous step, then run `sudo passwd --lock pi`.

### Updating and upgrading rasp pi OS

Run `sudo apt update`, then `sudo apt full-upgrade -y`, and finally `sudo apt clean` to clean up the downloaded package files.

### Kill unnecessary system services

List running services and disable services you don't need - e.g. wifi or bluetooth.

To see all active services, run `sudo systemctl --type=service --state=active`.

To disable the wifi service now, run `sudo systemctl disable --now wpa_supplicant.service`.

To disable the bluetooth service now, run`sudo systemctl disable --now bluetooth.service`

If you wanted to enable a service again by running `sudo systemctl enable --now bluetooth.service`.

### Restrict ssh accounts

Run `sudoedit /etc/ssh/sshd_config`.

Under the line “# Authentication”, add `AllowUsers <account_name1> <account_name2>`.

After the change, you will need to restart the sshd service using `sudo systemctl restart ssh` or rebooting.

### Firewall

Use ufw (uncomplicated firewall). Need to be careful not to lock yourself out. See links to rasp pi doc and YouTube video above.

### Brute-force detection

Use fail2ban which watchs system logs for repeated login attempts and add a firewall rule to prevent further access for a specified time. See links to rasp pi doc and YouTube video above.

### Automatic package update and upgrade

Not for production because of potential compatibility problems that may arise.

Use unattended-upgrades with raspberry pi specific config. Another option is to set up a cron job to run the update/upgrade commands. 

See link to YouTube video above for more details.

# Installing Git

### Sources 

https://projects.raspberrypi.org/en/projects/getting-started-with-git/3

### Install git

Run `sudo apt install git`, then check the installation by running `git --version`.

### Configuring git

Set up your username and email by running:
`git config --global user.name "Harry Potter"`, then `git config --global user.email "h.potter@hogwarts.prof"`.

Check the config by running `git config --list`.

The configuration is stored in the `~/.gitconfig` file. Edit this directly, or via `git config` to make further changes if required.

You can also tell git what text editor you'd like to use, for example this sets it to nano: `git config --global core.editor nano`.

# Installing Docker

### Sources

https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/

https://blog.alexellis.io/getting-started-with-docker-on-raspberry-pi/

https://sanderh.dev/setup-Docker-and-Docker-Compose-on-Raspberry-Pi/

https://dev.to/rohansawant/installing-docker-and-docker-compose-on-the-raspberry-pi-in-5-simple-steps-3mgl

https://www.zuidwijk.com/blog/installing-docker-and-docker-compose-on-a-raspberry-pi-4/

### Install Docker

Run `curl -sSL https://get.docker.com | sh`, then run `sudo gpasswd -a pi docker`, replacing pi with whatever account is the main account. Logout using `logout` command and log back in.

Test it by showing version and running the hello-world container: `docker version` and `docker run hello-world`. Afterwards, clean up by removing the container and the hello-world image: `docker rm <container id>` then `docker image rm hello-world`.

If you are not using the pi user, remember to add the account you wish to use to the docker usergroup. Check username is in the right groups using this command `grep '<username>' /etc/group`.

### Install Docker Compose

Run `sudo apt install libffi-dev libssl-dev python3 python3-pip`, then `sudo apt remove python-configparser`, and finally run `sudo pip3 -v install docker-compose`. Reboot the pi.

# Pihole with Docker Compose

### Sources

https://docs.pi-hole.net/

https://hub.docker.com/r/pihole/pihole/ 

https://github.com/pi-hole/docker-pi-hole

https://docs.pi-hole.net/guides/unbound/ 

https://github.com/chriscrowe/docker-pihole-unbound

https://github.com/anudeepND/whitelist

### Start Pihole in container

(Go to the next section if you want to start Pihole with Unbound. This is for starting Pihole in docker.)

Use the docker-compose.yaml in the above links. 

Edit the file to uncomment the environment property `WEBPASSWORD` and point it to the host's environment variable - i.e. `WEBPASSWORD: $PIHOLE_PASSWORD`. If we don't give it a password, a random one will be generated.

The docker-compose.yml sets pihole's password to whatever the environment variable `$PIHOLE_PASSWORD` is. Set it on the pi by running: `export PIHOLE_PASSWORD=’<password>’`. A more permanent alternative is to create a file in the same directory as the docker-compose.yaml called `.env` by running `touch .env`, then go into the file using an editor by running `sudo nano .etc` and add your environment variables - e.g. `PIHOLE_PASSWORD=password`.

Run the `docker-compose.yml` in `/docker-pihole` directory using command `docker-compose up -d`.

### Start Pihole and Unbound in a single container

Clone this git repository by running `git clone https://github.com/willypapa/raspberrypi.git`. It will clone the files into `/raspberry` directory, where you will find the docker-compose.yml file. We will now refer to the `pihole-unbound` service in the docker-compose.yml.

Change directories to be in the `/raspberrypi` directory by running, for example, `cd raspberrypi`. 

Create a `.env` file by running `sudo touch .env` in the same directory as the docker-compose.yml file. Populate it with the following environment variables which are referred to by the docker-componse.yml file:

  PIHOLE_PASSWORD=password
  PIHOLE_TIMEZONE=Europe/London
  PIHOLE_ServerIP=<IP address of the host raspberry pi - e.g.192.168.0.2>

In the same directory as the docker-compose.yml, run `docker-compose up -d`.

### Whitelist common false-positives

This is optional. The github repo we refer to here keeps a list of common false-positive domains for us to whitelist.

The whitelist is installed using a python script. However, the pihole/pihole docker image does not include a python installation. So we have to run the following on the host raspberry pi itself which should have python3 installed.

Run `git clone https://github.com/anudeepND/whitelist.git`, then `sudo python3 whitelist/scripts/whitelist.py --dir <path to /etc-pihole/ volume> --docker`
