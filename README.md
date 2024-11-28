# Easy Ergo Node + Oracle Setup

Prerequisites: make sure git, nano?, docker, and docker compose plugin are installed.
For Docker, follow: https://docs.docker.com/engine/install/  +  https://docs.docker.com/engine/install/linux-postinstall/

Clone this repo:

```console
git clone https://github.com/reqlez/ergo-easy-oracle.git && cd ergo-easy-oracle
```

Clone oracle-core repo ( OPTIONAL - only if building instead of using DockerHub image ):

```console
git clone -b develop https://github.com/ergoplatform/oracle-core.git
```

Create Docker network and set folder permissions with container UIDs:

```console
docker network create ergo-node
```

```console
sudo chown -R 9052:9052 node_data && sudo chown -R 9010:9010 oracle_data
```

Copy example .env file and edit:
( Replace CHANGE_ME_KEY with NEW key same as YOUR_API_KEY two steps below )

```console
cp env.example .env
nano .env
```

Build + start node container temporarily to generate API Key Hash:

```console
docker compose build --no-cache
docker compose up --no-start
docker compose start node
```

Generate an 'apiKeyHash' for node, ex:

```console
curl -X POST "http://localhost:9053/utils/hash/blake2b" -H "Content-Type: application/json" -d "\"YOUR_API_KEY\""
```

Stop node container:

```console
docker compose stop node
```

Uncomment / set settings like apiKeyHash:

- `sudo nano node_data/ergo.conf`

Start node and initialize wallet - IMPORTANT: make sure to store the mnemonic phrase output in a safe place:

```console
docker compose start node
```

```console
curl -X POST "http://localhost:9053/wallet/init" -H "api_key: YOUR_API_KEY" -H "Content-Type: application/json" -d "{\"pass\":\"YOUR_WALLET_PASS\"}"
```

Get wallet address so you can set it under 'oracle_address' in oracle config yaml

```console
curl -X GET "http://localhost:9053/wallet/addresses" -H "api_key: YOUR_API_KEY"
```

Add some ERG for your wallet address above.

Set oracle_address to the address from previous step:

- `sudo nano oracle_data/oracle_config.yaml`

Wait for node to sync, you can monitor progress under: http://ip.of.your.node:9053/panel

```console
docker compose down
```
```console
docker compose up -d
```

Please note that you will need the oracle tokens sent to that address as well ( oracle_address ).
Keep the wallet unlocked, for the oracle to be operational.

For troubleshooting, check combined logs via:

```console
docker compose logs -f
```

Or individual logs, via:

```console
docker compose logs -f node
docker compose logs -f core
docker compose logs -f prometheus
docker compose logs -f grafana
```

You may also have to restart the oracle, after unlocking wallet, for example:

```console
docker compose restart core
```
However, the auto-restart process should be able to handle that condition.

If you wish to set a cron task to extract tokens you can use below command with 'crontab -e' ex:

```console
@weekly docker compose -f /path/to/ergo-easy-oracle/docker-compose.yml exec -it core /bin/sh -c 'yes YES | oracle-core --oracle-config-file /data/oracle_config.yaml --pool-config-file /data/pool_config.yaml -d /data extract-reward-tokens ADDRESS_TO_EXTRACT_TO_HERE'
```
-------

# Optional: Setting Up a Systemd Service for Ergo Node Management
To manage the Ergo Node Docker Compose setup with systemctl, which will run the docker service in the background, follow these steps:

Create a systemd service file:

```console
sudo nano /etc/systemd/system/ergo-node.service
```

Add the following content to the file:

```console
[Unit]
Description=Ergo Node Docker Compose Service
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/path/to/ergo-easy-oracle
ExecStart=/usr/bin/docker compose up -d
#ExecStart=/usr/bin/docker compose up -d core node # preferred if not running grafana and prometheus extras
ExecStop=/usr/bin/docker compose down
ExecReload=/usr/bin/docker compose restart
RemainAfterExit=true
TimeoutStopSec=300     # gives enough grace to safely reboot, shutdown, and startup
Restart=on-failure
RestartSec=10
User=your_user_name
Group=your_group_name

[Install]
WantedBy=multi-user.target
```

Where:

- /path/to/ergo-easy-oracle with the path to your cloned repository. (go to directory and use pwd)
- your_user_name and your_group_name with the appropriate user and group.

Save the file and reload systemd (ctrl+X, yes):

```console
sudo systemctl daemon-reload
```

Enable and start the service:

```console
sudo systemctl enable ergo-node.service
sudo systemctl start ergo-node.service
```

Verify the service status:

```console
sudo systemctl status ergo-node.service
```

To stop the service:

```console
sudo systemctl stop ergo-node.service
```

This optional step simplifies the management of the Ergo Node, ensuring it restarts automatically and integrates with the system's boot process.
