# 0G Chain Validator Setup with Foundry

<div align="center">

[![0G Chain](https://img.shields.io/static/v1?label=0G&message=Chain&color=blue&style=for-the-badge)](https://0g.ai)
[![Foundry](https://img.shields.io/static/v1?label=Foundry&message=v1.2.3&color=green&style=for-the-badge)](https://github.com/foundry-rs/foundry)
[![Ubuntu](https://img.shields.io/static/v1?label=Ubuntu&message=18.04%2B&color=orange&style=for-the-badge)](https://ubuntu.com)
[![License](https://img.shields.io/static/v1?label=License&message=MIT&color=yellow&style=for-the-badge)](LICENSE)

**🚀 Complete guide to setup and manage 0G Chain validators using Foundry**

[📋 Prerequisites](#-prerequisites) • [🚀 Installation](#-installation) • [⚙️ Setup](#️-setup) • [🛠️ Management](#️-management) • [⚠️ Troubleshooting](#️-troubleshooting)

---

</div>

## 📋 Prerequisites

### Hardware Requirements
| Component | Testnet | Mainnet |
|-----------|---------|---------|
| **RAM** | 64 GB | 64 GB |
| **CPU** | 8 cores | 8 cores |
| **Storage** | 4 TB NVME SSD | 1 TB NVME SSD |
| **Network** | 100 Mbps | 100 Mbps |

### Software Requirements
- ✅ **Ubuntu 18.04+** or Debian-based Linux
- ✅ **0G Node** synced and running
- ✅ **32+ OG Tokens** for staking
- ✅ **Root/sudo access**
- ✅ **Stable internet connection**

### Network Information (Testnet)
```bash
Network Name: 0G-Galileo-Testnet
Chain ID: 16601
RPC URL: https://evmrpc-testnet.0g.ai
Explorer: https://chainscan-galileo.0g.ai
Faucet: https://faucet.0g.ai
```

---

## 🚀 Installation

### 1. System Check
```bash
# Check system compatibility
echo "=== System Information ==="
uname -a && arch && lsb_release -a

# Install required packages
sudo apt update && sudo apt install -y curl jq

# Check if 0G node is synced
curl -s http://localhost:26657/status | jq .result.sync_info.catching_up
# Should return: false (synced)
```

### 2. Install Foundry
```bash
# Check if Foundry is already installed
which cast && which forge && echo "✅ Foundry already installed" || echo "❌ Foundry installation required"

# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup

# Verify installation
echo "=== Foundry Version Information ==="
cast --version
forge --version
anvil --version
```

### 3. Test 0G Chain Connection
```bash
# Test connection to 0G testnet
echo "=== 0G Chain Connection Test ==="

# Check chain ID (should return 16601)
cast chain-id --rpc-url https://evmrpc-testnet.0g.ai

# Get current block number
cast block-number --rpc-url https://evmrpc-testnet.0g.ai

# Test staking contract read
cast call 0xea224dBB52F57752044c0C86aD50930091F561B9 \
    "validatorCount()(uint32)" \
    --rpc-url https://evmrpc-testnet.0g.ai
```

---

## ⚙️ Setup

### 1. Environment Configuration
```bash
# Set your node data path (adjust to your actual path)
export HOME_DIR=/root/.0gchaind/0g-home/0gchaind-home
export GENESIS_PATH=$HOME_DIR/config/genesis.json
export CHAIN_SPEC=devnet

# 0G Testnet configuration
export RPC_URL=https://evmrpc-testnet.0g.ai
export STAKING_CONTRACT=0xea224dBB52F57752044c0C86aD50930091F561B9
export INITIAL_STAKE=32000000000  # 32 OG in gwei

# Make persistent (add to ~/.bashrc)
echo "export RPC_URL=https://evmrpc-testnet.0g.ai" >> ~/.bashrc
echo "export STAKING_CONTRACT=0xea224dBB52F57752044c0C86aD50930091F561B9" >> ~/.bashrc
echo "export INITIAL_STAKE=32000000000" >> ~/.bashrc
```

### 2. Verify Required Files
```bash
# Check if required files exist
ls -la $HOME_DIR/config/genesis.json || echo "❌ genesis.json not found!"
ls -la $HOME_DIR/config/priv_validator_key.json || echo "❌ priv_validator_key.json not found!"

# Check if 0gchaind binary is accessible
which 0gchaind || echo "❌ 0gchaind not in PATH!"
```

### 3. Generate Validator Keys
```bash
# Generate validator public key
echo "=== Generating Validator Keys ==="
0gchaind deposit validator-keys \
  --home $HOME_DIR \
  --chaincfg.chain-spec=$CHAIN_SPEC

# Output example:
# Eth/Beacon Pubkey (Compressed 48-byte Hex):
# 0xaa0f99735a6436d6b7ed763c2eaa8452d753c5152a4fb1e4dc0bd7e33bcfc8cd4fac0e2d6cbab941f423c17728fecc56
```

**⚠️ IMPORTANT:** Copy the 48-byte pubkey from the output:
```bash
# Replace with your actual pubkey
export PUBKEY=0xaa0f99735a6436d6b7ed763c2eaa8452d753c5152a4fb1e4dc0bd7e33bcfc8cd4fac0e2d6cbab941f423c17728fecc56
```

### 4. Compute Validator Contract Address
```bash
# Calculate validator contract address
echo "=== Computing Validator Contract Address ==="
VALIDATOR_CONTRACT=$(cast call $STAKING_CONTRACT \
    "computeValidatorAddress(bytes)(address)" \
    $PUBKEY \
    --rpc-url $RPC_URL)

echo "Validator Contract Address: $VALIDATOR_CONTRACT"
export VALIDATOR_CONTRACT=$VALIDATOR_CONTRACT
```

### 5. Generate Signature
```bash
# Create validator signature
echo "=== Generating Validator Signature ==="
0gchaind deposit create-validator \
  $VALIDATOR_CONTRACT \
  $INITIAL_STAKE \
  $GENESIS_PATH \
  --home $HOME_DIR \
  --chaincfg.chain-spec=$CHAIN_SPEC

# Output example:
# ✅ Deposit message created successfully!
# pubkey: 0xaa0f99735a6436d6b7ed763c2eaa8452d753c5152a4fb1e4dc0bd7e33bcfc8cd4fac0e2d6cbab941f423c17728fecc56
# signature: 0x8d9f2e7a6b5c4d3e8f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e...
```

**⚠️ IMPORTANT:** Copy the signature from the output:
```bash
# Replace with your actual signature
export SIGNATURE=0x8d9f2e7a6b5c4d3e8f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e...
```

### 6. Prepare Wallet
```bash
# Set your private key (get from MetaMask)
# ⚠️ SECURITY: Never share your private key!
export PRIVATE_KEY=0x1234567890abcdef...  # Replace with your actual private key

# Check wallet balance (should have 32+ OG)
export YOUR_WALLET_ADDRESS=0x...  # Replace with your wallet address
cast balance $YOUR_WALLET_ADDRESS --rpc-url $RPC_URL --ether
```

### 7. Register Validator
```bash
# Register validator with staking contract
echo "=== Registering Validator ==="
cast send $STAKING_CONTRACT \
  "createAndInitializeValidatorIfNecessary((string,string,string,string,string),uint32,uint96,bytes,bytes)" \
  "('Your Validator Name','keybase-id','https://yourwebsite.com','security@youremail.com','Your validator description')" \
  50000 \
  1 \
  $PUBKEY \
  $SIGNATURE \
  --value $INITIAL_STAKE \
  --private-key $PRIVATE_KEY \
  --rpc-url $RPC_URL

# You will receive a transaction hash
# Example: 0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
```

#### Parameter Explanation:
- `'Your Validator Name'`: Display name for your validator
- `'keybase-id'`: Keybase identity (optional)
- `'https://yourwebsite.com'`: Your website URL (optional)
- `'security@youremail.com'`: Contact email
- `'Your validator description'`: Brief description
- `50000`: 5% commission rate (50000/1000000)
- `1`: 1 gwei withdrawal fee

---

## 🔍 Verification

### 1. Check Transaction Status
```bash
# Check transaction on explorer
echo "Check your transaction at: https://chainscan-galileo.0g.ai"

# Verify transaction status (replace with your actual tx hash)
cast tx 0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef --rpc-url $RPC_URL
```

### 2. Verify Validator Registration
```bash
# Check if validator is registered
echo "=== Validator Registration Check ==="

# Get validator contract address
cast call $STAKING_CONTRACT \
    "getValidator(bytes)(address)" \
    $PUBKEY \
    --rpc-url $RPC_URL

# Check validator balance (should show 32000000000)
cast call $VALIDATOR_CONTRACT \
    "tokens()(uint256)" \
    --rpc-url $RPC_URL

# Check commission rate (should show 50000)
cast call $VALIDATOR_CONTRACT \
    "commissionRate()(uint32)" \
    --rpc-url $RPC_URL

# Check delegator shares
cast call $VALIDATOR_CONTRACT \
    "delegatorShares()(uint256)" \
    --rpc-url $RPC_URL
```

### 3. Monitor Node Logs
```bash
# Monitor validator activation in node logs
tail -f $HOME_DIR/../log/0gchaind.log | grep -i validator

# Check consensus participation
curl -s http://localhost:26657/validators | jq '.result.validators[] | select(.pub_key.value != null)'
```

---

## 🛠️ Management

### Withdraw Commission
```bash
# Withdraw earned commission (only validator operator can do this)
cast send $VALIDATOR_CONTRACT \
    "withdrawCommission(address)" \
    $YOUR_WALLET_ADDRESS \
    --private-key $PRIVATE_KEY \
    --rpc-url $RPC_URL
```

### Check Delegation Information
```bash
# View total delegator shares
cast call $VALIDATOR_CONTRACT \
    "delegatorShares()(uint256)" \
    --rpc-url $RPC_URL

# Check specific delegator's shares
cast call $VALIDATOR_CONTRACT \
    "getDelegation(address)(address,uint256)" \
    $DELEGATOR_ADDRESS \
    --rpc-url $RPC_URL

# Calculate estimated token value for shares
# Formula: (shares * totalTokens) / totalShares
```

### Delegate Additional Tokens
```bash
# Add more stake to your validator
cast send $VALIDATOR_CONTRACT \
    "delegate(address)" \
    $YOUR_WALLET_ADDRESS \
    --value 1000000000 \
    --private-key $PRIVATE_KEY \
    --rpc-url $RPC_URL
```

### Undelegate (Advanced)
```bash
# Undelegate shares (requires withdrawal fee)
cast send $VALIDATOR_CONTRACT \
    "undelegate(address,uint256)" \
    $WITHDRAWAL_ADDRESS \
    $SHARES_AMOUNT \
    --value 1000000000 \
    --private-key $PRIVATE_KEY \
    --rpc-url $RPC_URL
```

### Network Statistics
```bash
# Total validator count
cast call $STAKING_CONTRACT \
    "validatorCount()(uint32)" \
    --rpc-url $RPC_URL

# Maximum validator count
cast call $STAKING_CONTRACT \
    "maxValidatorCount()(uint32)" \
    --rpc-url $RPC_URL

# Current gas price
cast gas-price --rpc-url $RPC_URL
```

---

## ⚠️ Troubleshooting

### Common Errors

#### 1. "insufficient funds"
```bash
# Check wallet balance
cast balance $YOUR_WALLET_ADDRESS --rpc-url $RPC_URL --ether
# Need at least 32.1 OG (extra for gas)

# Get test tokens from faucet
echo "Get tokens from: https://faucet.0g.ai"
```

#### 2. "DelegationBelowMinimum"
```bash
# Check stake amount
echo $INITIAL_STAKE
# Must be exactly 32000000000 (32 OG in gwei)
```

#### 3. "signature mismatch"
```bash
# Regenerate signature
0gchaind deposit create-validator \
  $VALIDATOR_CONTRACT \
  $INITIAL_STAKE \
  $GENESIS_PATH \
  --home $HOME_DIR \
  --chaincfg.chain-spec=$CHAIN_SPEC
```

#### 4. "command not found: 0gchaind"
```bash
# Check if 0gchaind is in PATH
which 0gchaind

# Add to PATH if needed
export PATH=$PATH:/path/to/0gchaind/bin
echo 'export PATH=$PATH:/path/to/0gchaind/bin' >> ~/.bashrc
```

#### 5. "invalid opcode"
```bash
# Check if using correct EVM version
# Ensure contracts are compiled with --evm-version cancun
```

### Debug Commands
```bash
# Check node sync status
curl -s http://localhost:26657/status | jq .result.sync_info

# Check network connection
curl -s https://evmrpc-testnet.0g.ai >/dev/null && echo "✅ RPC accessible" || echo "❌ Connection issue"

# View latest block
cast block latest --rpc-url $RPC_URL

# Check transaction pool
cast tx-pool --rpc-url $RPC_URL
```

### Health Check Script
```bash
#!/bin/bash
echo "=== 0G Validator Health Check ==="

echo "1. Node sync status:"
curl -s http://localhost:26657/status | jq -r .result.sync_info.catching_up

echo "2. Validator contract address:"
cast call $STAKING_CONTRACT "getValidator(bytes)(address)" $PUBKEY --rpc-url $RPC_URL

echo "3. Validator balance:"
cast call $VALIDATOR_CONTRACT "tokens()(uint256)" --rpc-url $RPC_URL

echo "4. Commission rate:"
cast call $VALIDATOR_CONTRACT "commissionRate()(uint32)" --rpc-url $RPC_URL

echo "5. Total network validators:"
cast call $STAKING_CONTRACT "validatorCount()(uint32)" --rpc-url $RPC_URL

echo "6. Wallet balance:"
cast balance $YOUR_WALLET_ADDRESS --rpc-url $RPC_URL --ether

echo "=== Health check complete ==="
```

---

## ✅ Success Criteria

Your validator setup is successful if:

- ✅ **Transaction confirmed** - Shows SUCCESS status on explorer
- ✅ **getValidator()** returns your validator contract address
- ✅ **tokens()** returns 32000000000 (32 OG)
- ✅ **Node logs** show validator activation messages
- ✅ **Consensus participation** - Validator appears in validator set

---

## 📚 Resources

### Official Links
- **Documentation**: https://docs.0g.ai
- **GitHub**: https://github.com/0glabs
- **Block Explorer**: https://chainscan-galileo.0g.ai
- **Faucet**: https://faucet.0g.ai

### Contract Addresses
- **Staking Contract**: `0xea224dBB52F57752044c0C86aD50930091F561B9`

### RPC Endpoints
- **Primary**: https://evmrpc-testnet.0g.ai
- **QuickNode**: Custom endpoint available
- **ThirdWeb**: Custom endpoint available

### Support
- **Discord**: [0G Labs Community](https://discord.gg/0glabs)
- **Telegram**: Official 0G Channel
- **GitHub Issues**: For technical problems

---

## 📄 License

This guide is released under the MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**🎉 Congratulations! You are now running a 0G Chain validator! 🎉**

*If you found this guide helpful, please ⭐ star this repository!*

</div>
