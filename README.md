# Aztec-node-guide-iziedking
# Aztec Execution & Consensus Node Setup Upgraded Guide with Geth and Prysm

> **Full Setup from Scratch with Explanations for Beginners**

---

##  Step 1: System Requirements

### Minimum Specs for Sequencer

| Spec       | Minimum (Testnet) | Recommended |
|------------|-------------------|-------------|
| CPU        | 4 cores            | 6+ cores    |
| RAM        | 8 GB               | 16 GB       |
| Disk       | 500 GB SSD         | 1 TB SSD |
| OS         | Ubuntu 20.04/22.04 or WSL2 on Windows |

 **If you have proper electricity and a robust device with the requirement above you can as well run this for free otherwise use a VPS**
 
* Create a new metamask wallet for this, save your seed phrase securely get your private keys and public address for the node 
* If you are already running node before delete all the data see below these guides all remove commands (reach out to me if you have any errors) and start afresh with this 

### Recommended VPS Providers

* [Hetzner](https://www.hetzner.com)
* [Contabo](https://contabo.com)
* [DigitalOcean](https://www.digitalocean.com)
* [Racknerd](https://racknerd.com) 

* Choose a VPS with **at least:**
  * **4+ CPU Cores**
  * **16GB RAM**
  * **1TB SSD/NVMe**
  * **25 Mbps up/down bandwidth**

---
![Screenshot 2025-05-23 083138](https://github.com/user-attachments/assets/d635811d-5aa3-4861-846d-ec2984dcda0b)
![Screenshot 2025-05-23 083305](https://github.com/user-attachments/assets/34ca9ce4-8030-4c65-b5a6-b30da04ce1cb)
![Screenshot 2025-05-23 083341](https://github.com/user-attachments/assets/a450f64b-8cd5-4dde-995b-44b83c3795e4)
![Screenshot 2025-05-23 083405](https://github.com/user-attachments/assets/6510b992-d3c0-402b-bda7-7ac98fd4b302)

Choose a password you won't forget!
After first purchase go back to your accounct and make second purchase via an upgrade to get required specs follow the Image guide 
---
![Screenshot 2025-05-23 084416](https://github.com/user-attachments/assets/e1bb0c20-ef93-4076-afc0-4b8569bfe8c4)
![Screenshot 2025-05-23 084357](https://github.com/user-attachments/assets/743b2c9b-f5fb-48d9-a959-3fa42a668fb0)
![Screenshot 2025-05-23 084450](https://github.com/user-attachments/assets/f46664f6-b6d7-4fc2-8bb2-e843961a6a9f)
![Screenshot 2025-05-23 084541](https://github.com/user-attachments/assets/5778c91d-0751-4a11-ada2-d506c68d65c4)

---
* Once you successfully purchase a VPS:
  1. Download [Termius](https://termius.com/download/windows)
  2. watch this video here to learn how to set it up with termius with your VPS [Video](https://t.me/crypto_circuitN/1381) (Credit to AirdropAnalyst for video)
 
> Note if you are using your System for this ensure to install WSL windows subsystems and Docker Desktop (Need guide in setting this up? reach out to me. VPS strongly recommended)

---

##  Step 2: Install Essential Dependencies

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip screen -y
```

* Updates system packages and installs everything needed to compile, monitor, and run Ethereum clients.

---

##  Step 3: Install Docker & Add User to Docker Group

```bash
source <(wget -O - https://raw.githubusercontent.com/frianowzki/installer/main/docker.sh)
```
```bash
sudo groupadd docker && sudo usermod -aG docker $(whoami) && newgrp docker
```
```bash
bash -i <(curl -s https://install.aztec.network)
```
```bash
echo 'export PATH=$PATH:/root/.aztec/bin' >> ~/.bashrc
```
```bash
source ~/.bashrc
```
```bash
aztec-up alpha-testnet
```

* This sets up Docker so we can run the Aztec sequencer in a container.

---

##  Step 4: Activate Firewall & Open Required Ports

Run the following commands:

```bash
sudo ufw allow 8545/tcp    
sudo ufw allow 30303/tcp   
sudo ufw allow 30303/udp   
sudo ufw allow 3500/tcp    
sudo ufw allow 4000/tcp    
sudo ufw allow 12000/udp  
sudo ufw allow 13000/tcp   
sudo ufw allow 8082/tcp 
sudo ufw allow 8081/tcp   
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp     
sudo ufw enable
```

>  **Tip**: If any port above is already used when you try to run step 10 or blocked by your VPS provider, consider changing the Aztec port (e.g., from `8082` to `8181`) by modifying the `--port` flag in your `aztec start` command see Image.

To find an available port:

```bash
sudo lsof -i -P -n | grep LISTEN
```

This will show currently used ports so you can pick a free one.

![Screenshot 2025-05-23 092218](https://github.com/user-attachments/assets/f68972a2-6d83-4952-a8f2-0013a690fc58)


---

##  Step 5: Setup Geth + Prysm Users & JWT Secret
* Add users
```bash
sudo adduser --home /home/geth --disabled-password --gecos 'Geth Client' geth
sudo adduser --home /home/beacon --disabled-password --gecos 'Prysm Beacon Client' beacon
```
* Add eth group
```bash
sudo groupadd eth
sudo usermod -a -G eth geth
sudo usermod -a -G eth beacon
```
* Create JWT secret
```bash
sudo mkdir -p /var/lib/secrets
```
```bash
sudo chgrp -R eth /var/lib/ /var/lib/secrets
```
```bash
sudo chmod 750 /var/lib/ /var/lib/secrets
```
```bash
sudo openssl rand -hex 32 | tr -d '\n' | sudo tee /var/lib/secrets/jwt.hex > /dev/null
```
```bash
sudo chown root:eth /var/lib/secrets/jwt.hex
```
```bash
sudo chmod 640 /var/lib/secrets/jwt.hex
```

 **JWT secret** is required for communication between execution (Geth) and consensus (Beacon) layers.

---

##  Step 6: Setup Data Directories

```bash
sudo -u geth mkdir /home/geth/geth
sudo -u beacon mkdir /home/beacon/beacon
```

---

##  Step 7: Install Geth (Execution Client)

```bash
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum -y
```

```bash
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.15.11-36b2371c.tar.gz

```bash
tar -xvf geth-linux-amd64-1.15.11-36b2371c.tar.gz
```

```bash
sudo mv geth-linux-amd64-1.15.11-36b2371c/geth /usr/bin/geth
```


---

##  Step 8: Create & Start Geth Systemd Service

```bash
sudo nano /etc/systemd/system/geth.service
```

**Insert the following:**

```
[Unit]
Description=Geth
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
Restart=always
RestartSec=5s
User=geth
WorkingDirectory=/home/geth
ExecStart=/usr/bin/geth \
  --sepolia \
  --http \
  --http.addr "0.0.0.0" \
  --http.port 8545 \
  --http.api "eth,net,engine,admin" \
  --authrpc.addr "127.0.0.1" --authrpc.port 8551 \
  --http.corsdomain "*" \
  --http.vhosts "*" \
  --datadir /home/geth/geth \
  --authrpc.jwtsecret /var/lib/secrets/jwt.hex

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start geth
sudo systemctl enable geth
sudo journalctl -fu geth  # View logs
```

---

##  Step 9: Install & Run Prysm (Consensus Client)

```bash
sudo -u beacon mkdir /home/beacon/bin
sudo -u beacon curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh -o /home/beacon/bin/prysm.sh
sudo -u beacon chmod +x /home/beacon/bin/prysm.sh
```

```bash
sudo nano /etc/systemd/system/beacon.service
```

**Insert the following:**

```
[Unit]

Description=Prysm Beacon
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
Restart=always
RestartSec=5s
User=beacon
ExecStart=/home/beacon/bin/prysm.sh beacon-chain \
  --sepolia \
  --http-modules=beacon,config,node,validator \
  --rpc-host=0.0.0.0 --rpc-port=4000 \
  --grpc-gateway-host=0.0.0.0 --grpc-gateway-port=3500 \
  --datadir /home/beacon/beacon \
  --execution-endpoint=http://127.0.0.1:8551 \
  --jwt-secret=/var/lib/secrets/jwt.hex \
  --checkpoint-sync-url=https://checkpoint-sync.sepolia.ethpandaops.io/ \
  --genesis-beacon-api-url=https://checkpoint-sync.sepolia.ethpandaops.io/ \
  --accept-terms-of-use

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start beacon
sudo systemctl enable beacon
sudo journalctl -fu beacon  # View logs
```

 **Let Geth & Beacon fully sync before proceeding may take up to a day but check at hourly intervals mine completed in 2 hours.**

---

##  Step 10: Sync Check Script

```bash
nano sync.sh
```

```bash
#!/bin/bash
echo "=== GETH SYNC STATUS ==="
curl -s -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' -H "Content-Type: application/json" http://localhost:8545 | jq
echo "\n=== BEACON SYNC STATUS ==="
curl -s http://localhost:3500/eth/v1/node/syncing | jq
```

```bash
chmod +x sync.sh
```
```bash
./sync.sh
```
> **You can leave terminal and always come back to check synv with ./sync.sh**
---

* If they are in sync you will get something like this:
  
  ![image](https://github.com/user-attachments/assets/6184fa76-674c-436e-9ada-9b7148f7c028)

* Then you can proceed to step 11

---

##  Step 11: Run Aztec Sequencer with `screen`

```bash
screen -S aztec-sequencer
```

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls RPC_URL  \
  --l1-consensus-host-urls CONSENSUS_HOST_URL \
  --sequencer.validatorPrivateKey 0xPrivateKey \
  --sequencer.coinbase 0xPublicAddress \
  --p2p.p2pIp IP \
  --port 8081 
```
* Change RPC_URL with:
```bash
http://Your_VPS_IP:8545
```
* Change CONSENSUS_HOST_URL with:
```bash
http://Your_VPS_IP:3500
```
* Change IP to:
```bash
Your_VPS_IP
```

> Press `Ctrl+A` then `D` to detach the screen safely.

###  Check logs anytime:

```bash
screen -r aztec-sequencer
```

---

##  Step 12: Stop & Restart Sequencer

```bash
# Find container
docker ps

# Stop container
docker stop <container_id>

# Restart manually (inside screen)
screen -S aztec-sequencer
aztec start ... # (same command in step 10)
```

---

##  Optional: Remove Everything if need arise
* Stop services
```bash
sudo systemctl stop geth.service
sudo systemctl stop beacon.service
```
* Disable services
```bash
sudo systemctl disable geth.service
sudo systemctl disable beacon.service
```
* Remove services
```bash
sudo rm /etc/systemd/system/geth.service
sudo rm /etc/systemd/system/beacon.service
```
* Delete data
```bash
sudo rm -rf /home/geth /home/beacon
```
* Stop All Running Docker Containers
```bash
docker stop $(docker ps -q)
```
* Remove All Docker Containers, Images, Volumes, and Networks
```bash
docker system prune -a --volumes -f

```

---
**If your Node is running, You should get something like this:**

![image](https://github.com/user-attachments/assets/c445a5ee-2ccd-4806-823a-ab26d9065915)

---
# Final step:

* head to aztec Discord [here](https://discord.gg/aztec)
* Check `#operators` and `#node-support` channels
* Use `/operator start` command in Discord to register.

  Submit:
  1. Address you used for the node
  2. Block_Number
  3. Proof
     
![succint (59 4 x 42 cm) (59 x 38 cm) (50 x 35 cm) (39 x 30 cm)](https://github.com/user-attachments/assets/6b736bce-6e67-4a19-b421-0b4c4fde3b1c)

* To get your Block_number and Proof
  RUn this command:
  Get latest proven block:

  ```bash
  curl -s -X POST -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
  http://localhost:8082
  ```

 > Note: use the port your node is running on from your command in step 10 
 
  Generate sync proof:
  
  ```bash
  curl -s -X POST -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["block_number","block_number"],"id":67}' \
  http://localhost:8082 | jq
  ```

> Note: change "block_number to latest Proven Block_Number see Image

  Below is an Image of a block_number you'd get and how to use it to generate a proof

  ![Screenshot 2025-05-23 102928](https://github.com/user-attachments/assets/ed025b8a-0761-427b-bedd-d5e2b9513c4a)


---

  * For guadian role:
     Maintain node uptime then use your VPS or Local System IP and wallet address you used for the node to check at intervals (maybe daily) if you are whitelisted

    ![Screenshot 2025-05-23 105309](https://github.com/user-attachments/assets/959158ab-ba8c-4220-b230-6d8ce98407ef)

      * head to aztec Discord [here](https://discord.gg/aztec)
      * Check `#operators` and `#node-support` channels
      * Use `/checkip` command in Discord to register.

     ![Screenshot 2025-05-23 104951](https://github.com/user-attachments/assets/301a11fd-3586-4116-808e-96b7d50c6759)

    
---








# Using Docker Compose (Most Recomended to have a clean run)

 * After step 10, your Geth and Prysm block should be in Sync
  * Create a .env to save your sensitive infomation 
  ```bash
  nano .env
  ```
  * paste this and replace
    
     your_aztec_private_key_here with your private key
    
     your_VPS_IP with correct IP
    
  
  ```bash
  VALIDATOR_PRIVATE_KEY=your_aztec_private_key_here
  P2P_IP=your_VPS_IP
  ETHEREUM_HOSTS=http://Your_VPS_IP:8545
  L1_CONSENSUS_HOST_URLS=http://Your_VPS_IP:3500
  BOOTNODES=/dns4/bootnode.aztec.network/tcp/40400/p2p/16Uiu2HAmExampleBootNodeID
  ```
  > Press `Ctrl+O` then Enter to save; then `Ctrl+X` to exit.
  
  * Create a docker-compose.yml file 
  ```bash
  nano docker-compose.yml
  ```

  * Paste this command 
  ```bash
  version: "3.8"

  services:
    aztec-sequencer:
      container_name: aztec-sequencer
      image: aztecprotocol/aztec:alpha-testnet
      restart: unless-stopped
      ports:
        - "8080:8080"
        - "40400:40400/tcp"
        - "40400:40400/udp"
      volumes:
        - /root/.aztec:/data   
      environment:
        DATA_DIRECTORY: /data
        VALIDATOR_PRIVATE_KEY: ${VALIDATOR_PRIVATE_KEY}
        P2P_IP: ${P2P_IP}
        ETHEREUM_HOSTS: ${ETHEREUM_HOSTS}
        L1_CONSENSUS_HOST_URLS: ${L1_CONSENSUS_HOST_URLS}
        BOOTNODES: ${BOOTNODES}
        LOG_LEVEL: info
      entrypoint: >
        sh -c "node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start 
        --network alpha-testnet 
        --node 
        --archiver 
        --sequencer 
        --sequencer.governanceProposerPayload 0x54F7fe24E349993b363A5Fa1bccdAe2589D5E5Ef"
  ```
  > Press `Ctrl+O` then Enter to save; then `Ctrl+X` to exit.
---

### Monitor node with Screen
```bash
screen -S aztec-sequencer
```
* Run this command
* Start the Aztec Sequencer
```bash
docker compose up -d
```
* To view Logs
```bash
docker logs -f aztec-sequencer
```
> Press `Ctrl+A` then `D` to detach the screen safely.

###  Check logs anytime:

```bash
screen -r aztec-sequencer
```
---
## In cases of any Issue or adjustments
* Check Logs
```bash
screen -r aztec-sequencer
```
* Restart, Stop, or Update
  * Stop the node:
  ```bash
  docker compose down
  ```
  * Rebuild with latest image:
  ```bash
  docker compose pull
  docker compose up -d 
  ```
  



---

##  Guide by: **@Iziedking**

> This guide helps you run the node enough to get apprentice and guardian roles
> Reach me at: \[Telegram: @Izie17 X: [Twitter Profile](https://x.com/Iziedking) ]

---


