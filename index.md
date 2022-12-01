# Creating a Wireguard VPN Server using Digital Ocean and Docker

Here is a link to my repo: <https://github.com/rhk7913/VPNproject>

This page will outline how to create a Wireguard VPN Server on Digital Ocean using Docker.

# Create a droplet on Digital Ocean (DO)

A Digital Ocean droplet is a virtual machine (VM) that is ran on top of virtualized hardware.

Go to the following link to make a DO account: <https://m.do.co/c/4d7f4ff9cfe4>

**Note:** You will need to enter a credit card to verify your account, however you won't be charged. 

Enter the Control Panel and locate Droplets under the Manage label. 

Hit Create and follow the next steps:
1. Select your region. I chose New York.
2. Select a datacenter. I chose Datacenter 1.
3. Select an image. I chose Ubuntu 20.04 LTS for my OS and selected the Docker 19.03.12 Marketplace Application so that Docker would already be installed when I launched the droplet. 
4. Select a Droplet Type. I chose to use the default option, the basic shared CPU.
5. Select a CPU option. I chose the $6 a month option (1 GB / 1 CPU, 25 GB SSD disk, and 1000 GB transfer) with the Regular with SSD option.
6. Select an authentication method. I chose to the use password method since I intended to delete my droplet right after I completed the project. 
7. Hit Create Droplet and your droplet is now able to be used!

## Note  
I used the following resource for the remaining steps: <https://thematrix.dev/setup-wireguard-vpn-server-with-docker/>

# Setup Wireguard

Wireguard is a open source program that allows you to create encrypted virtual private networks (VPNs). 

After your droplet has been created and provisioned, you will have the ability to either ssh into the droplet from a local host terminal or through a console on your droplet itself. I chose to launch the console the droplet itself.

Hit the Console button and a window should appear. You are now inside the droplet's console!

The following commands will create two different directories used to setup Wireguard:
```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
```

A YAML file, which is used for configuration processes including applications that deal with storing and transmitting data, is needed here to configure the Docker container hosting the Wireguard VPN server. 

Install your preferred text editor. I chose to use nano here.

The following command will create a Docker Compose YAML file for the Wireguard VPN server:
```
nano ~/wireguard/docker-compose.yml
```

Insert the following information into your file:
```
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

However, don't save and exit the file yet! There are a number of fields that need to be modified:
1. You will need to select your specific TZ from <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>. I chose America/Chicago for my TZ. 
2. You will need to change the SERVERURL field to your droplet's Public IPV4 address. To find this address, locate the Networking menu on your droplet. My particular Public IPV4 address was 24.199.88.147.
3. You may choose to alter the PEERS field, which is the number of user-config files generated when the Docker Container is created, but I chose to leave the default option. 

**Optional change:** You can change the default SERVERPORT option to port 80 so that the port of the VPN may not be blocked on a public Wi-FI network, however leaving SERVERPORT as 51820 is fine.

Hit CTRL + X, Y, and Enter to save and exit the YAML file.

Use the following command to ensure that the above modifications were applied successfully:
```
cat ~/wireguard/docker-compose.yml
```

# Starting Wireguard

To start Wireguard, you will need to change into the ~/wireguard/ directory.

The following command will change to the ~/wireguard/ directory from your current directory:
```
cd ~/wireguard/
```

The following command will start Docker Compose:
```
docker-compose up -d
```

Once you see "Creating wireguard... done", your Wireguard VPN server is setup!

# Using the Wireguard VPN on your phone

The following command will display the execution log and QR codes of the newly created Wireguard VPN:
```
docker-compose logs -f wireguard
```

Download the Wireguard app on your phone.

Enter the app and hit the + button. Select the Create from QR Code option. 

**Note:** If you used the default PEERS option, there should be 3 QR codes that appear. Select any of the three QR codes to use. I chose to use the phone1 QR code.

You now have your Wireguard VPN working on your phone!

This was my phone's IP address before and after using the Wireguard VPN:

**Before**

![Screenshot (96)](https://user-images.githubusercontent.com/114512130/202825537-bdbc4c2b-cea4-4119-9076-49733e57f345.png)

**After**

![Screenshot (95)](https://user-images.githubusercontent.com/114512130/202825407-c68c27ce-43f8-4508-af5a-bcf419411f96.png)

# Using the Wireguard VPN on your computer

Download the Wireguard application on your computer from <https://www.wireguard.com/install/>.

To connect your computer to the Wireguard VPN you will need to change into the ~/wireguard/config directory.

The following command will change to the ~/wireguard/config directory from your current directory:
```
cd ~/wireguard/config
```

The following command will list all the files and directories in the ~/wireguard/config directory:
```
ls
```

Select a PEER option to use in the process of connecting the Wireguard VPN to your computer. I chose to use peer_pc1.

The following command will change into the directory of your selected PEER option:
```
cd peername
```

For instance, I used the following command to change into the peer_pc1 directory:
```
cd peer_pc1
```

The following command will list all the files and directories in the PEER option directory:
```
ls
```

The following command will display the contents of the configuration file:
```
cat peername.conf
```

For instance, I used the following command to display the contents of the peer_pc1 configuration file:
```
cat peer_pc1.conf
```

This is what my configuration file contained:
```
[Interface]
Address = 10.0.0.2
PrivateKey = sA+hbdkKCGfxpzCIAIY87oz0RSXTeaTqPUepnJUicHM=
ListenPort = 51820
DNS = 10.0.0.1

[Peer]
PublicKey = REE1ZvgkIZATqQwmgtyRG6KiF/7vFo9ugHrgfOkhyT8=
PresharedKey = AG1Pz4Hy4avC9qHRbfXBgRuJjkWh8EakL8zNVlGL+pY=
Endpoint = 24.199.88.147:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

**Note:** I did not hide the public and private key information here since I destroyed my droplet right after I completed the project. I would also recommend that you destroy your droplet right after you finish setting up the Wireguard VPN so that you don't accidentally get charged for using it after the 60 day trial ends. 

Open up the Wireguard application. Hit CTRL + N to create a new empty tunnel. 

Copy and paste the information from your configuration file. 

**Note:** I found it easiest to place the information in a .txt file before pasting it into the field since it was difficult to copy and paste the information directly from the DO console. 

**This is what my tunnel information looked like**

![Screenshot (42)](https://user-images.githubusercontent.com/114512130/202824290-ca02541c-ea19-4206-aa1c-bb2469c30a0b.png)

Hit Save.

You now have your Wireguard VPN working on your computer!

This was my computers's IP address before and after using the Wireguard VPN:

**Before**

![Screenshot (40)](https://user-images.githubusercontent.com/114512130/202824460-b9765972-0c8f-4d6f-a05f-eb4f559f8c40.png)
![Screenshot (34)](https://user-images.githubusercontent.com/114512130/202824586-b33c6e5e-67c6-49d8-9f60-b7c4f162400e.png)

**After**

![Screenshot (41)](https://user-images.githubusercontent.com/114512130/202824543-68186fda-49f3-45c1-9d5f-4beea684272e.png)
![Screenshot (35)](https://user-images.githubusercontent.com/114512130/202824515-8a18e757-39be-4f2a-8d2b-a5926c7a5917.png)
