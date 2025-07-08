# 0G Chain Validator Kurulum - Foundry ile

<div align="center">

[![0G Chain](https://img.shields.io/static/v1?label=0G&message=Chain&color=blue&style=for-the-badge)](https://0g.ai)
[![Foundry](https://img.shields.io/static/v1?label=Foundry&message=v1.2.3&color=green&style=for-the-badge)](https://github.com/foundry-rs/foundry)
[![Ubuntu](https://img.shields.io/static/v1?label=Ubuntu&message=18.04%2B&color=orange&style=for-the-badge)](https://ubuntu.com)
[![Lisans](https://img.shields.io/static/v1?label=Lisans&message=MIT&color=yellow&style=for-the-badge)](LICENSE)

**🚀 Foundry kullanarak 0G Chain validator kurulum ve yönetim rehberi**

[📋 Gereksinimler](#-gereksinimler) • [🚀 Kurulum](#-kurulum) • [⚙️ Yapılandırma](#️-yapılandırma) • [🛠️ Yönetim](#️-yönetim) • [⚠️ Sorun Giderme](#️-sorun-giderme)

---

</div>

## 📋 Gereksinimler

### Donanım Gereksinimleri
| Bileşen | Testnet | Mainnet |
|---------|---------|---------|
| **RAM** | 64 GB | 64 GB |
| **CPU** | 8 çekirdek | 8 çekirdek |
| **Depolama** | 4 TB NVME SSD | 1 TB NVME SSD |
| **Ağ** | 100 Mbps | 100 Mbps |

### Yazılım Gereksinimleri
- ✅ **Ubuntu 22.04+** veya Debian tabanlı Linux
- ✅ **0G Node** senkronize ve çalışıyor
- ✅ **32+ OG Token** stake için
- ✅ **Root/sudo erişimi**
- ✅ **Kararlı internet bağlantısı**

### Ağ Bilgileri (Testnet)
```bash
Ağ Adı: 0G-Galileo-Testnet
Chain ID: 16601
RPC URL: https://evmrpc-testnet.0g.ai
Explorer: https://chainscan-galileo.0g.ai
Faucet: https://faucet.0g.ai
```

---

## 🚀 Kurulum

### 1. Sistem Kontrolü
```bash
# Sistem uyumluluğunu kontrol et
echo "=== Sistem Bilgileri ==="
uname -a && arch && lsb_release -a

# Gerekli paketleri yükle
sudo apt update && sudo apt install -y curl jq

# 0G node senkronize mi kontrol et
curl -s http://localhost:26657/status | jq .result.sync_info.catching_up
# kENDİ RPC PORTUNU KULLAN DEFAULT:26557 Sonuç: false olmalı (senkronize)
```

### 2. Foundry Kurulumu
```bash
# Foundry kurulu mu kontrol et
which cast && which forge && echo "✅ Foundry zaten kurulu" || echo "❌ Foundry kurulumu gerekli"

# Foundry kur
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup

# Kurulumu doğrula
echo "=== Foundry Versiyon Bilgileri ==="
cast --version
forge --version
anvil --version
```

### 3. 0G Chain Bağlantısını Test Et
```bash
# 0G testnet bağlantısını test et
echo "=== 0G Chain Bağlantı Testi ==="

# Chain ID kontrol et (16601 dönmeli)
cast chain-id --rpc-url https://evmrpc-testnet.0g.ai

# Güncel blok numarasını al
cast block-number --rpc-url https://evmrpc-testnet.0g.ai

# Staking contract okuma testi
cast call 0xea224dBB52F57752044c0C86aD50930091F561B9 \
    "validatorCount()(uint32)" \
    --rpc-url https://evmrpc-testnet.0g.ai
```

---

## ⚙️ Yapılandırma

### 1. Ortam Değişkenleri
```bash
# Node data path'inizi ayarlayın (gerçek path'inize göre düzenleyin)
export HOME_DIR=/root/.0gchaind/0g-home/0gchaind-home
export GENESIS_PATH=$HOME_DIR/config/genesis.json
export CHAIN_SPEC=devnet

# 0G Testnet konfigürasyonu
export RPC_URL=https://evmrpc-testnet.0g.ai
export STAKING_CONTRACT=0xea224dBB52F57752044c0C86aD50930091F561B9

# Stake miktarları (EVM ve Cosmos işlemleri için farklı)
export INITIAL_STAKE_EVM=32000000000000000000    # EVM/cast komutları için 32 OG (wei cinsinden)
export INITIAL_STAKE_COSMOS=32000000000          # 0gchaind komutları için 32 OG (Cosmos formatı)

# Kalıcı hale getir (~/.bashrc'ye ekle)
echo "export RPC_URL=https://evmrpc-testnet.0g.ai" >> ~/.bashrc
echo "export STAKING_CONTRACT=0xea224dBB52F57752044c0C86aD50930091F561B9" >> ~/.bashrc
echo "export INITIAL_STAKE_EVM=32000000000000000000" >> ~/.bashrc
echo "export INITIAL_STAKE_COSMOS=32000000000" >> ~/.bashrc
```

### 2. Gerekli Dosyaları Doğrula
```bash
# Gerekli dosyaların varlığını kontrol et
ls -la $HOME_DIR/config/genesis.json || echo "❌ genesis.json bulunamadı!"
ls -la $HOME_DIR/config/priv_validator_key.json || echo "❌ priv_validator_key.json bulunamadı!"

# 0gchaind binary'sinin erişilebilir olduğunu kontrol et
which 0gchaind || echo "❌ 0gchaind PATH'de değil!"
```

### 3. Validator Key'lerini Oluştur
```bash
# Validator public key oluştur
echo "=== Validator Key'leri Oluşturuluyor ==="
0gchaind deposit validator-keys \
  --home $HOME_DIR \
  --chaincfg.chain-spec=$CHAIN_SPEC

# Çıktı örneği:
# Eth/Beacon Pubkey (Compressed 48-byte Hex):
# 0xaa0f99735a6436d6b7ed763c2eaa8452d753c5152a4fb1e4dc0bd7e33bcfc8cd4fac0e2d6cbab941f423c17728fecc56
```

**⚠️ ÖNEMLİ:** Çıktıdan 48-byte pubkey'i kopyalayın:
```bash
# Gerçek pubkey'inizi buraya yazın
export PUBKEY=0xaa0f99735a6436d6b7ed763c2eaa8452d753c5152a4fb1e4dc0bd7e33bcfc8cd4fac0e2d6cbab941f423c17728fecc56
```

### 4. Validator Contract Adresini Hesapla
```bash
# Validator contract adresini hesapla
echo "=== Validator Contract Adresi Hesaplanıyor ==="
VALIDATOR_CONTRACT=$(cast call $STAKING_CONTRACT \
    "computeValidatorAddress(bytes)(address)" \
    $PUBKEY \
    --rpc-url $RPC_URL)

echo "Validator Contract Adresi: $VALIDATOR_CONTRACT"
export VALIDATOR_CONTRACT=$VALIDATOR_CONTRACT
```

### 5. İmza Oluştur
```bash
# Validator imzası oluştur (Cosmos formatı kullanarak)
echo "=== Validator İmzası Oluşturuluyor ==="
0gchaind deposit create-validator \
  $VALIDATOR_CONTRACT \
  $INITIAL_STAKE_COSMOS \
  $GENESIS_PATH \
  --home $HOME_DIR \
  --chaincfg.chain-spec=$CHAIN_SPEC

# Çıktı örneği:
# ✅ Deposit message created successfully!
# pubkey: 0xaa0f99735a6436d6b7ed763c2eaa8452d753c5152a4fb1e4dc0bd7e33bcfc8cd4fac0e2d6cbab941f423c17728fecc56
# signature: 0x8d9f2e7a6b5c4d3e8f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e...
```

**⚠️ ÖNEMLİ:** Çıktıdan imzayı kopyalayın:
```bash
# Gerçek imzanızı buraya yazın
export SIGNATURE=0x8d9f2e7a6b5c4d3e8f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e...
```

### 6. Cüzdan Hazırla
```bash
# Private key'inizi ayarlayın (MetaMask'tan alın)
# ⚠️ GÜVENLİK: Private key'inizi hiç kimseyle paylaşmayın!
export PRIVATE_KEY=0x1234567890abcdef...  # Gerçek private key'inizi girin

# Cüzdan bakiyesini kontrol edin (32+ OG olmalı)
export YOUR_WALLET_ADDRESS=0x...  # Cüzdan adresinizi girin
cast balance $YOUR_WALLET_ADDRESS --rpc-url $RPC_URL --ether
```

### 7. Validator'ı Kaydet
```bash
# Validator'ı staking contract'ına kaydet (EVM formatı kullanarak)
echo "=== Validator Kaydediliyor ==="
cast send $STAKING_CONTRACT \
  "createAndInitializeValidatorIfNecessary((string,string,string,string,string),uint32,uint96,bytes,bytes)" \
  "('Validator Adınız','keybase-id','https://websiteniz.com','guvenlik@emailiniz.com','Validator açıklamanız')" \
  50000 \
  1 \
  $PUBKEY \
  $SIGNATURE \
  --value $INITIAL_STAKE_EVM \
  --gas-limit 1000000 \
  --gas-price 50000000000 \
  --private-key $PRIVATE_KEY \
  --rpc-url $RPC_URL

# Transaction hash alacaksınız
# Örnek: 0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
```

#### Parametre Açıklamaları:
- `'Validator Adınız'`: Validator'ınızın görünen adı
- `'keybase-id'`: Keybase kimliği (opsiyonel)
- `'https://websiteniz.com'`: Web sitenizin URL'si (opsiyonel)
- `'guvenlik@emailiniz.com'`: İletişim email adresi
- `'Validator açıklamanız'`: Kısa açıklama
- `50000`: %5 komisyon oranı (50000/1000000)
- `1`: 1 gwei çekim ücreti

---

## 🔍 Doğrulama

### 1. Transaction Durumunu Kontrol Et
```bash
# Transaction'ı explorer'da kontrol et
echo "Transaction'ınızı kontrol edin: https://chainscan-galileo.0g.ai"

# Transaction durumunu doğrula (gerçek tx hash'inizi yazın)
cast tx 0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef --rpc-url $RPC_URL
```

### 2. Validator Kaydını Doğrula
```bash
# Validator'ın kayıtlı olup olmadığını kontrol et
echo "=== Validator Kayıt Kontrolü ==="

# Validator contract adresini al
cast call $STAKING_CONTRACT \
    "getValidator(bytes)(address)" \
    $PUBKEY \
    --rpc-url $RPC_URL

# Validator bakiyesini kontrol et (32000000000000000000 göstermeli)
cast call $VALIDATOR_CONTRACT \
    "tokens()(uint256)" \
    --rpc-url $RPC_URL

# Komisyon oranını kontrol et (50000 göstermeli)
cast call $VALIDATOR_CONTRACT \
    "commissionRate()(uint32)" \
    --rpc-url $RPC_URL

# Delegator paylarını kontrol et
cast call $VALIDATOR_CONTRACT \
    "delegatorShares()(uint256)" \
    --rpc-url $RPC_URL
```

### 3. Node Loglarını İzle
```bash
# Node loglarında validator aktivasyonunu izle
tail -f $HOME_DIR/../log/0gchaind.log | grep -i validator

# Consensus katılımını kontrol et
curl -s http://localhost:26657/validators | jq '.result.validators[] | select(.pub_key.value != null)'
```

---

## 🛠️ Yönetim

### Komisyon Çekme
```bash
# Kazanılan komisyonu çek (sadece validator operatörü yapabilir)
cast send $VALIDATOR_CONTRACT \
    "withdrawCommission(address)" \
    $YOUR_WALLET_ADDRESS \
    --gas-limit 1000000 \
    --gas-price 50000000000 \
    --private-key $PRIVATE_KEY \
    --rpc-url $RPC_URL
```

### Delegation Bilgilerini Kontrol Et
```bash
# Toplam delegator paylarını görüntüle
cast call $VALIDATOR_CONTRACT \
    "delegatorShares()(uint256)" \
    --rpc-url $RPC_URL

# Belirli bir delegator'ın paylarını kontrol et
cast call $VALIDATOR_CONTRACT \
    "getDelegation(address)(address,uint256)" \
    $DELEGATOR_ADDRESS \
    --rpc-url $RPC_URL

# Paylar için tahmini token değeri hesapla
# Formül: (paylar * toplamToken) / toplamPaylar
```

### Ek Token Delegate Et
```bash
# Validator'ınıza daha fazla stake ekle
cast send $VALIDATOR_CONTRACT \
    "delegate(address)" \
    $YOUR_WALLET_ADDRESS \
    --value 1000000000000000000 \
    --gas-limit 1000000 \
    --gas-price 50000000000 \
    --private-key $PRIVATE_KEY \
    --rpc-url $RPC_URL
```

### Undelegate (İleri Seviye)
```bash
# Payları undelegate et (çekim ücreti gerekir)
cast send $VALIDATOR_CONTRACT \
    "undelegate(address,uint256)" \
    $WITHDRAWAL_ADDRESS \
    $SHARES_AMOUNT \
    --value 1000000000000000000 \
    --gas-limit 1000000 \
    --gas-price 50000000000 \
    --private-key $PRIVATE_KEY \
    --rpc-url $RPC_URL
```

### Ağ İstatistikleri
```bash
# Toplam validator sayısı
cast call $STAKING_CONTRACT \
    "validatorCount()(uint32)" \
    --rpc-url $RPC_URL

# Maksimum validator sayısı
cast call $STAKING_CONTRACT \
    "maxValidatorCount()(uint32)" \
    --rpc-url $RPC_URL

# Güncel gas fiyatı
cast gas-price --rpc-url $RPC_URL
```

---

## ⚠️ Sorun Giderme

### Yaygın Hatalar

#### 1. "insufficient funds"
```bash
# Cüzdan bakiyesini kontrol et
cast balance $YOUR_WALLET_ADDRESS --rpc-url $RPC_URL --ether
# En az 32.1 OG gerekli (gas için extra)

# Faucet'tan test token al
echo "Token almak için: https://faucet.0g.ai"
```

#### 2. "DelegationBelowMinimum"
```bash
# EVM işlemleri için stake miktarını kontrol et
echo $INITIAL_STAKE_EVM
# Tam olarak 32000000000000000000 olmalı (32 OG wei cinsinden)

# Cosmos işlemleri için stake miktarını kontrol et
echo $INITIAL_STAKE_COSMOS
# Tam olarak 32000000000 olmalı (32 OG Cosmos formatında)
```

#### 3. "signature mismatch"
```bash
# İmzayı yeniden oluştur (Cosmos formatı kullanarak)
0gchaind deposit create-validator \
  $VALIDATOR_CONTRACT \
  $INITIAL_STAKE_COSMOS \
  $GENESIS_PATH \
  --home $HOME_DIR \
  --chaincfg.chain-spec=$CHAIN_SPEC
```

#### 4. "command not found: 0gchaind"
```bash
# 0gchaind'in PATH'de olup olmadığını kontrol et
which 0gchaind

# Gerekirse PATH'e ekle
export PATH=$PATH:/path/to/0gchaind/bin
echo 'export PATH=$PATH:/path/to/0gchaind/bin' >> ~/.bashrc
```

#### 5. "invalid opcode"
```bash
# Doğru EVM versiyonunu kullandığınızdan emin olun
# Contract'lar --evm-version cancun ile derlenmiş olmalı
```

### Debug Komutları
```bash
# Node senkronizasyon durumunu kontrol et
curl -s http://localhost:26657/status | jq .result.sync_info

# Ağ bağlantısını kontrol et
curl -s https://evmrpc-testnet.0g.ai >/dev/null && echo "✅ RPC erişilebilir" || echo "❌ Bağlantı sorunu"

# Son bloğu görüntüle
cast block latest --rpc-url $RPC_URL

# Transaction pool'unu kontrol et
cast tx-pool --rpc-url $RPC_URL
```

### Sağlık Kontrol Script'i
```bash
#!/bin/bash
echo "=== 0G Validator Sağlık Kontrolü ==="

echo "1. Node senkronizasyon durumu:"
curl -s http://localhost:26657/status | jq -r .result.sync_info.catching_up

echo "2. Validator contract adresi:"
cast call $STAKING_CONTRACT "getValidator(bytes)(address)" $PUBKEY --rpc-url $RPC_URL

echo "3. Validator bakiyesi:"
cast call $VALIDATOR_CONTRACT "tokens()(uint256)" --rpc-url $RPC_URL

echo "4. Komisyon oranı:"
cast call $VALIDATOR_CONTRACT "commissionRate()(uint32)" --rpc-url $RPC_URL

echo "5. Toplam ağ validator sayısı:"
cast call $STAKING_CONTRACT "validatorCount()(uint32)" --rpc-url $RPC_URL

echo "6. Cüzdan bakiyesi:"
cast balance $YOUR_WALLET_ADDRESS --rpc-url $RPC_URL --ether

echo "7. Stake miktarları:"
echo "   EVM formatı: $INITIAL_STAKE_EVM"
echo "   Cosmos formatı: $INITIAL_STAKE_COSMOS"

echo "=== Sağlık kontrolü tamamlandı ==="
```

---

## ✅ Başarı Kriterleri

Validator kurulumunuz başarılı ise:

- ✅ **Transaction onaylandı** - Explorer'da SUCCESS durumu gösterir
- ✅ **getValidator()** validator contract adresinizi döndürür
- ✅ **tokens()** 32000000000000000000 (32 OG wei cinsinden) döndürür
- ✅ **Node logları** validator aktivasyon mesajları gösterir
- ✅ **Consensus katılımı** - Validator validator set'te görünür

---

## 📚 Kaynaklar

### Resmi Linkler
- **Dokümantasyon**: https://docs.0g.ai
- **GitHub**: https://github.com/0glabs
- **Block Explorer**: https://chainscan-galileo.0g.ai
- **Faucet**: https://faucet.0g.ai

### Contract Adresleri
- **Staking Contract**: `0xea224dBB52F57752044c0C86aD50930091F561B9`

### RPC Endpoint'leri
- **Birincil**: https://evmrpc-testnet.0g.ai
- **QuickNode**: Özel endpoint mevcut
- **ThirdWeb**: Özel endpoint mevcut

### Destek
- **Discord**: [0G Labs Topluluğu](https://discord.gg/0glabs)
- **Telegram**: Resmi 0G Kanalı
- **GitHub Issues**: Teknik problemler için

---

## 📄 Lisans

Bu rehber MIT Lisansı altında yayınlanmıştır. Detaylar için [LICENSE](LICENSE) dosyasına bakın.

---

<div align="center">

**🎉 Tebrikler! Artık bir 0G Chain validator'ı çalıştırıyorsunuz! 🎉**

*Bu rehber faydalı olduysa, lütfen ⭐ bu repository'yi yıldızlayın!*

</div>
