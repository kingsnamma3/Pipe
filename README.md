# Pipe Node Run

## Pre-Requirements

* **Need invite code for Node run and devent node ID for invite code(if ran devnet node than you should have it, if not, run that and get the node id and use it in the form for invite code)**

* If you already have invite code, go to point 2 and start from there directly.

* Invite code ke liye google form fill krdo, wait kro fir invite code gmail pr aane ka:- node id devnet node run krke google form me fill krdena and save rakhlena kahin:- [Form](https://tinyurl.com/4435uukt)

# 1. Devnet Node for Node id

```
curl -L -o pop "https://dl.pipecdn.app/v0.2.8/pop"
```
```
chmod +x pop
```
* Make Directory
```
mkdir download_cache
```
* Sign up
```
 ./pop --signup-by-referral-route e4b9ef5a01c0d8b7
```
* Start Devnet node
```
sudo ./pop --ram 8 --max-disk 500 --cache-dir /data --pubKey <KEY> --enable-80-443
```
* Change `<KEY>` with you phantom/Solana wallet address and adjust ram and disk space according to your specification. but need only node id, no need to run this node any further.

* Output have node id: save it somewhere safe and use it for testnet node


# 2. Testnet Node
* Install/Update Required Dependecies
```
sudo apt-get update && sudo apt-get upgrade -y
```
```
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev screen ufw -y
```

* Creating new screen
```
screen -S pipe
```

* Allow Incoming connections on Ports 
```
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow ssh
sudo ufw enable
sudo ufw reload
```

* Applying Configurations
```
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'
sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```

* Increase file limits for performance
```
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```

* Create Directories
```
sudo mkdir -p /opt/popcache
sudo mkdir -p /opt/popcache/logs
```


# Download the binary file for node run

* Open - [Download_Binary](https://download.pipe.network)
* Enter Invite Code that you have recieved on mail
* Select the `Architecture` As per your vps or Device (x64 mostly) and Download it in your `Personal Computer`!

* if using vps ->  move the binary file from your Local device to Vps

* Move & Unrap the pop file with following these commands!

```
sudo mv ~/pop-v0.3.2-linux-x64.tar.gz /opt/popcache/
```
* Whichever version you download, change version in the fhe file path according to that in 1 place in above code

```
cd /opt/popcache/
```

```
sudo tar -xzf pop-v0.3.2-linux-x64.tar.gz
sudo chmod +x ./pop
chmod 777 pop-v0.3.2-linux-x64.tar.gz
sudo ln -sf /opt/popcache/pop /usr/local/bin/pop
```
* Whichever version you download, change version in the fhe file path according to that in the 2 places in above code

# Set Configuration File

```
sudo nano config.json
```

```
{
  "pop_name": "your-pop-name",
  "pop_location": "Your Location, Country",
  "invite_code": "Enter your Invite Code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 0
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-node-name",
    "name": "Your Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your_discord_username",
    "telegram": "your_telegram_handle",
    "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS_FOR_REWARDS"
  }
}
```
* Dont Forget to make all these changes in File! All are Important Configrations!

* `pop_name`: Any username
* `pop_location:` Your's or VPS Region's Location
* `Enter your Invite Code`: Put invite code
* `memory_cache_size_mb:`  How much RAM memory you want to allocate to node! (In MB)- 50% of total RAM!
* `disk_cache_size_gb:` How much disk space do you want to allocate! (In GB) around 100 GB
* `your-node-name`: your username
* `your Name`: same username
* `your.email@example.com`: your email that you got the code on
* replace discord and telegram username as well and solana wallet address
  
*  Paste the following code in `config.json` File and press `CTRL+X` -> `Y` -> `Enter`
 
* Create a Systemd Service File
```
sudo bash -c 'cat > /etc/systemd/system/popcache.service << EOL
[Unit]
Description=POP Cache Node
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/popcache
ExecStart=/opt/popcache/pop
Restart=always
RestartSec=5
LimitNOFILE=65535
StandardOutput=append:/opt/popcache/logs/stdout.log
StandardError=append:/opt/popcache/logs/stderr.log
Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]
WantedBy=multi-user.target
EOL'
```

* Reload
```
sudo systemctl daemon-reload
```

* Enable
```
sudo systemctl enable popcache
```

* Start
```
sudo systemctl start popcache
```

* Get out of the screen
  Press `CTRL + A + D ` 

* Check node Status
```
sudo systemctl status popcache
```

```
tail -f /opt/popcache/logs/stdout.log
```

# save your pop_id somewhere safe
```
curl -sk https://localhost/metrics | jq . && curl -sk https://localhost/state | jq . && curl -sk https://localhost/health | jq .
```

# For restarting the node
* Enter screen
```
screen -r pipe
```

* Stop the service 
```
sudo systemctl stop popcache
```

* Restart
```
sudo systemctl restart popcache
```

# Upgrading node to new release/version for future 
* Enter screen
```
screen -r pipe
```


* Stop popcache service

```
sudo systemctl stop popcache
```

* Delete old binary 

```
sudo rm -f /usr/local/bin/pop

```
```
sudo rm -f /opt/popcache/pop
```

```
sudo rm -f /opt/popcache/pop-v0.3.1-linux-x64.tar.gz
```
* Change version no. in the path above with your version that you were using
* If showed error, delete binary file manually: In local pc: Pc>Linux>Ubuntu>Home>Your_wls_username>delete your old pop binary file
* paste binary file manually : In local pc: Pc>Linux>Ubuntu>Home>Your_wls_username>Paste_your_`pop-v0.3.2-linux-x64.tar.gz`_file
 
* Delete old logs data

```
sudo rm /opt/popcache/logs/stderr.log
```

```
sudo rm /opt/popcache/logs/stdout.log
```

* Move & Unrap the new pop file with following these commands!

```
sudo mv ~/pop-v0.3.2-linux-x64.tar.gz /opt/popcache/
```
* Whichever version you download, change version in the fhe file path according to that in 1 place in above code

```
cd /opt/popcache/
```

```
sudo tar -xzf pop-v0.3.2-linux-x64.tar.gz
sudo chmod +x ./pop
chmod 777 pop-v0.3.2-linux-x64.tar.gz
sudo ln -sf /opt/popcache/pop /usr/local/bin/pop
```
* Whichever version you download, change version in the fhe file path according to that in the 2 places in above code

* After that Reload & Start the service:

```
sudo systemctl daemon-reload
```

```
sudo systemctl start popcache
```

* this is all, can check the logs from previous commands for checking logs
 
* if you get something like mesh service unavailable, it will work automatically, ignore that.
