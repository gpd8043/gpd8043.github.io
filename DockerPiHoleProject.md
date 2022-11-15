First, I downloaded Docker for Ubuntu on the VM with these commands
```
sudo apt update
 
sudo apt install ca-certificates curl gnupg lsb-release
 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
sudo apt update
 
sudo apt install docker-ce docker-ce-cli containerd.io
 
sudo usermod -aG docker sysadmin
 
# Test with:
sudo docker run hello-world
```

Then I installed Docker Compose
`sudo apt-get update`
`sudo apt-get install docker-compose-plugin`

I chose to utilize Pi Hole via Docker.

First I started Docker `sudo systemctl start docker`

Then downloaded the pi hole implementation from docker 
`sudo docker pull pihole/pihole`

I created the docker-compose file with the following command and utilized nano for the next section to customize the .yml file. 

`nano docker-compose.yml`

I copied a docker-compose.yml file from the Pi-Hole documentation. 

More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/

```
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: 'America/Chicago'
    volumes:
       - './etc-pihole/:/etc/pihole/'
       - './etc-dnsmasq.d/:/etc/dnsmasq.d/'
    dns:
      - 127.0.0.1
      - 1.1.1.1
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

Then run the compose file with the following command

`sudo docker -f docker-compose.yml up -d`

I had issues running this command, as the port 53 was already being used by something, I found out what was using it with command
`sudo lsof -i -P -n | grep LISTEN`

Then stopped the service with this command 
`systemctl disable systemd-resolved.service`

Then finally reran the compose command.

To finish up the setup, you need to set a password for the pi hole web app

Move to the container with this command 
`sudo docker exec -it pihole bash`

Then changed the password to `sysadmin`, with this command 
`pihole -a -p`

After `exit`-ing the container, you can navigate to the web dashboard by typing the IP address associated with the pi-hole. 

![[Pasted image 20221115172312.png]]
![[Pasted image 20221115172506.png]]
