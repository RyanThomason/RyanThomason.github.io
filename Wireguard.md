# Wireguard Install

## Setting up Droplet

1. First thing to do was create the droplet. Start by making Digital Ocean account with referral link for extra credits.
2. Once the account is created, then you need to create the droplet inside Digital Ocean. You do this by going to the droplet menu and creating a droplet with desired settings
3. Our desired settings here was:
    - Ubuntu 20.04 LTS
    - Regular CPU
    - Password
    - Basic package
    - New York Center
4. Once the droplet was created I just had to launch the console from the Digital Ocean website where the droplet was at.

## Getting Docker Setup on Droplet

1. For this I needed to first start out by running the following command to install necessary tools:
  - ```sudo apt install apt-transport-https ca-certificates curl software-properties-common -y```
2. After that command I then add our docker key with the following command:
  - ```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -```
3. Next I needed to add the Docker repo (used the 32bit/64bit OS):
  - ```
    sudo add-apt-repository \
    "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```
4. Next you actually needed to swap to correct repo with this command:
  - ```apt-cache policy docker-ce```
5. Then you can install Docker with the following command:
  - ```sudo apt install docker-ce -y```
6. Last two steps are to install Docker-Compose. For this you use the following commands and then right after you need to set permissions for docker-compose to make it an executable:
  - ```sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```
  - ```sudo chmod +x /usr/local/bin/docker-compose```

## Setting up Wireguard on the Droplet

1. First thing to do is setup the directories for the Wireguard docker. The following creates two directories needed:
  - ```mkdir -p ~/wireguard/```
  - ```mkdir -p ~/wireguard/config/```
2. Then with the following command you create a docker compose file and get into a nano to paste the text into the file:
  - ```nano ~/wireguard/docker-compose.yml```
### Docker Compose File Contents
  - ```
    version: '3.8'
    services:
      wireguard:
        container_name: wireguard
        image: linuxserver/wireguard
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Hong_Kong
          - SERVERURL=1.2.3.4
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
3. Once that is done you do need to fix a few things in that file such as:
  - changing the TZ from Asia/Hong_Kong to US/Central
  - changing the SERVERURL to the correct server IP address that is in Digital Ocean
4. From here you should just have to run the following commands and it should work
  - ```cd ~/wireguard/```
  - ```docker-compose up -d```

## Connecting to Wireguard on mobile device

1. Run the following command and it will bring up QR codes for your wireguard settings for the different peers that are setup in the .yml file:
  - ```docker-compose logs -f wireguard```
2. Scan the QR Code that comes up and then you will have the settings and then you just need to turn on the VPN and then it should work

## Connecting to Wireguard on PC

1. For connecting with a PC all you need to do is install the Wireguard application from their website.
2. After installing you just need to get the config of the peer_pc1.confg file and get it into the application.
3. From there you should be able to turn it on and use it

## Screenshots for using VPN on Phone
### Without VPN
![PhoneWithoutVPN](https://user-images.githubusercontent.com/73131611/144691462-1c577403-d2dd-4730-a047-de92dcc04b74.jpg)
### Enabled VPN

![PhoneEnabledVPN](https://user-images.githubusercontent.com/73131611/144691712-a03b9b06-3e9c-4010-b84a-c65faf66d333.jpg)



### With VPN
![PhoneWithVPN](https://user-images.githubusercontent.com/73131611/144691402-90dffea7-93df-40a2-a6b0-2d666ed9a149.jpg)


## Screenshots for using VPN on PC

![PCScreenshot](https://user-images.githubusercontent.com/73131611/144691482-23691e9e-0430-4173-ac8e-78a7e320b456.png)


## Screenshot of Docker running Wireguard

![RunningInDocker](https://user-images.githubusercontent.com/73131611/144691474-6b509662-c26f-43c2-a6ab-91c2bc1da441.png)


## Resources
1. https://thematrix.dev/install-docker-and-docker-compose-on-ubuntu-20-04/
2. https://thematrix.dev/setup-wireguard-vpn-server-with-docker/

  
