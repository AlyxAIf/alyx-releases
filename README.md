
# ALYX Explorer + Public Validator Onboarding

This repo contains the ALYX explorer UI and public onboarding information for the ALYX testnet.

## Network: alyxtest-2

### Chain metadata (public)
- genesis.json: https://alyxai.org/networks/alyxtest-2/genesis.json
- addrbook.json: https://alyxai.org/networks/alyxtest-2/addrbook.json

### Public endpoints
- RPC: https://rpc.alyxai.org
- LCD/REST: https://api.alyxai.org

### Seed (P2P)
- 894c256e2a4601c10046d94d644bd2dab95f3434@p2p.alyxai.org:26656

## Validator join (copy/paste)

> NOTE: The ALYX node binary is built from a private repo. We publish binaries separately.
> You must install `alyxd` at `/usr/local/bin/alyxd` before running the block below.

```bash
# =========================
# ALYX alyxtest-2 — JOIN
# =========================

# 0) Dependencies (Ubuntu)
sudo apt-get update -y
sudo apt-get install -y curl jq unzip

# 1) Install alyxd
# NOTE: We'll publish an official download link. For now, you must place alyxd on PATH.
# Required final location (recommended): /usr/local/bin/alyxd
# Example (placeholder):
# curl -L "<ALYXD_BINARY_URL>" -o /usr/local/bin/alyxd && chmod +x /usr/local/bin/alyxd

alyxd version

# 2) Init node
MONIKER="<your-moniker>"
alyxd init "$MONIKER" --chain-id alyxtest-2

# 3) Fetch genesis + addrbook (official)
GENESIS_URL="https://alyxai.org/networks/alyxtest-2/genesis.json"
ADDRBOOK_URL="https://alyxai.org/networks/alyxtest-2/addrbook.json"

curl -L "$GENESIS_URL" -o ~/.alyx/config/genesis.json
curl -L "$ADDRBOOK_URL" -o ~/.alyx/config/addrbook.json

# 4) Configure seed (official)
SEED="894c256e2a4601c10046d94d644bd2dab95f3434@p2p.alyxai.org:26656"

sed -i 's|^seeds *=.*|seeds = "'"$SEED"'"|g' ~/.alyx/config/config.toml

# Optional: set minimum gas price (sample)
# sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ualyx"|g' ~/.alyx/config/app.toml

# 5) Create systemd service
sudo tee /etc/systemd/system/alyxd.service >/dev/null <<'UNIT'
[Unit]
Description=ALYX Validator Node
After=network-online.target
Wants=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/alyxd start --home /root/.alyx
Restart=always
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
UNIT

sudo systemctl daemon-reload
sudo systemctl enable alyxd
sudo systemctl restart alyxd

# 6) Check sync
journalctl -u alyxd -n 50 --no-pager
curl -s https://rpc.alyxai.org/status | jq -r '.result.sync_info.catching_up, .result.sync_info.latest_block_height'

# 7) Create validator (after synced)
# alyxd tx staking create-validator ... --from <wallet> --chain-id alyxtest-2 --node https://rpc.alyxai.org
