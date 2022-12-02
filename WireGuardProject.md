I started out by making an account on Digital Ocean, and redeeming my free $200 credit. 

Then using the Digital Ocean website, I created a minimum size droplet, which is a Linux virtual machine.  

Using the web console, start with installing the tools for docker using the online instructions.

`sudo apt install apt-transport-https ca-certificates curl software-properties-common -y`

Then create a docker key with the following command.
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

Finally, add the docker repository with this command. 
```
sudo add-apt-repository \
 "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) \
 stable"
```

Then switch to the correct repo,
`apt-cache policy docker-ce`

Install Docker
```
sudo apt install docker-ce -y
```

Install Docker compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Set permissions for Docker Compose
```
sudo chmod +x /usr/local/bin/docker-compose
```


To set up Wire Guard, there is a set of commands to be put in rapid succession. 

```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```

I copy and pasted this from a Wire Guard set up guide, then adjusted the settings for my specific case. 

```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - US=America/Chicago
      - SERVERURL=159.89.236.87
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```

Then start WireGuard with this command. 

```
cd ~/wireguard/
docker-compose up -d
```

To connect WireGuard, use this command
````
docker-compose logs -f wireguard
````

After downloading the WireGuard app on my phone and used the QR code provided with the peer phone1 code.

After naming the tunnel, turn on the VPN and test with the website. 
![[pic 2.jpg]]
![[pic .jpg]]
To utilize Wire Guard on desktop, first find the config file in the droplet. 
```
cd ~/wireguard/configs/peer_pc1
nano peer_pc1.conf
```

Then download that file to the desktop and throw it in WireGuard as a new tunnel. 

![[pic.png]]![[activePNG.png]]
