# Easy Ergo Node + Oracle Setup

Prerequisites: make sure git, nano?, docker, and docker compose plugin are installed.

Clone this repo:

```console
git clone https://github.com/reqlez/tn-ergo-easy-oracle.git && cd tn-ergo-easy-oracle
```

Clone oracle-core repo:

```console
git clone -b develop https://github.com/ergoplatform/oracle-core.git
```

Create Docker network and set folder permissions with container UIDs:

```console
docker network create tn-ergo-node
```

```console
sudo chown -R 9052:9052 node_data && sudo chown -R 9010:9010 oracle_data
```

Build + start node container temporarily to generate API Key Hash:

```console
docker compose build --no-cache
docker compose up --no-start
docker compose start node
```

Generate an 'apiKeyHash' for node, ex:

```console
curl -X POST "http://localhost:9052/utils/hash/blake2b" -H "Content-Type: application/json" -d "\"YOUR_API_KEY\""
```

Stop node container:

```console
docker compose stop node
```

Uncomment / set settings like apiKeyHash:

- `sudo nano node_data/ergo.conf`

Start node container and initialize wallet:

```console
docker compose start node
```

```console
curl -X POST "http://localhost:9052/wallet/init" -H "api_key: YOUR_API_KEY" -H "Content-Type: application/json" -d "{\"pass\":\"YOUR_WALLET_PASS\"}"
```

Get wallet address so you can set it under 'oracle_address' in oracle config yaml

```console
curl -X GET "http://localhost:9052/wallet/addresses" -H "api_key: YOUR_API_KEY"
```

Visit https://tn-faucet.ergohost.io and get some test ERG for your wallet address.

Uncomment / set settings like CCARCH ( aarch64 or x86_64 ), ORACLE_NODE_API_KEY, oracle_address, etc:

- `nano docker-compose.yml`
- `sudo nano oracle_data/oracle_config.yaml`

Wait for node to sync, you can monitor progress under: http://ip.of.your.node:9052/panel

```console
docker compose down
```
```console
docker compose up -d
```

Please note that you will need the oracle tokens sent to that address as well ( oracle_address ).
Keep the wallet unlocked, for the oracle to be operational.

For troubleshooting, check combined node + oracle logs via:

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
