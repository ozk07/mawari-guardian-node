# Mawari Guardian Node Kurulum Rehberi

Mawari Network TestNet'te Guardian Node kurulumu için basit Türkçe rehber.

## Gereksinimler
- VPS Sunucu (Ubuntu 20.04/22.04)
- 2 CPU, 4GB RAM, 20GB SSD
- MetaMask cüzdan

## Adım 1: Test Token ve NFT Alma

1. MetaMask cüzdanınızı açın
2. https://hub.testnet.mawari.net adresinden test token alın
3. https://testnet.mawari.net adresinden 3 adet Guardian NFT mint edin

## Adım 2: Sunucuya Bağlanma

```bash
ssh root@sunucu_ip_adresi
```

## Adım 3: Docker Kurulumu

```bash
# Docker kur
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Çıkış yap ve tekrar gir
exit
ssh root@sunucu_ip_adresi
```

## Adım 4: Node Kurulumu

```bash
# Klasör oluştur
mkdir -p ~/mawari

# Node'u başlat (CUZDAN_ADRESI yerine kendi adresinizi yazın)
docker run -d \
  --name mawari-node \
  --restart unless-stopped \
  -v ~/mawari:/app/cache \
  -e OWNERS_ALLOWLIST=0xCUZDAN_ADRESI \
  us-east4-docker.pkg.dev/mawarinetwork-dev/mwr-net-d-car-uses4-public-docker-registry-e62e/mawari-node:latest
```

## Adım 5: Burner Wallet Alma

```bash
# 30 saniye bekle
sleep 30

# Burner wallet adresini al
docker logs mawari-node 2>&1 | grep "Using burner wallet"
```

## Adım 6: Delegation İşlemi

1. Burner wallet'a 0.5-1 test token gönderin
2. https://app.testnet.mawari.net adresine gidin
3. 3 NFT'yi burner wallet adresine delegate edin

## Kontrol Komutları

```bash
# Node durumu
docker ps

# Son loglar
docker logs --tail 50 mawari-node

# Heartbeat kontrolü
docker logs mawari-node 2>&1 | grep heartbeat | tail -5

# Node'u yeniden başlatma
docker restart mawari-node

# Node'u durdurma
docker stop mawari-node

# Node'u silme
docker rm -f mawari-node
```

## Otomatik Kurulum Scripti

`kurulum.sh` dosyası oluşturun:

```bash
#!/bin/bash

# Cüzdan adresini sor
echo "Cüzdan adresinizi girin (0x ile başlayan):"
read WALLET

# Docker kontrolü
if ! command -v docker &> /dev/null; then
    echo "Docker kuruluyor..."
    curl -fsSL https://get.docker.com | sh
    sudo usermod -aG docker $USER
    echo "Lütfen exit yapıp tekrar giriş yapın"
    exit 1
fi

# Node başlat
echo "Node başlatılıyor..."
mkdir -p ~/mawari
docker rm -f mawari-node 2>/dev/null

docker run -d \
  --name mawari-node \
  --restart unless-stopped \
  -v ~/mawari:/app/cache \
  -e OWNERS_ALLOWLIST=$WALLET \
  us-east4-docker.pkg.dev/mawarinetwork-dev/mwr-net-d-car-uses4-public-docker-registry-e62e/mawari-node:latest

echo "30 saniye bekleniyor..."
sleep 30

echo "Burner Wallet:"
docker logs mawari-node 2>&1 | grep "Using burner wallet" | head -1

echo ""
echo "ŞİMDİ YAPMANIZ GEREKENLER:"
echo "1. Burner wallet'a 0.5-1 test token gönderin"
echo "2. https://app.testnet.mawari.net adresinden NFT'leri delegate edin"
```

Çalıştırma:
```bash
chmod +x kurulum.sh
./kurulum.sh
```

## Sorun Giderme

### Node çalışmıyor
```bash
docker logs mawari-node
```

### Port sorunu
```bash
sudo ufw allow 22
sudo ufw allow 443
sudo ufw allow 80
sudo ufw enable
```

### Docker permission hatası
```bash
sudo usermod -aG docker $USER
# Çıkış yapıp tekrar girin
```

## Destek

Discord: [Mawari Discord](https://discord.gg/mawari)
Twitter: [@MawariNetwork](https://twitter.com/MawariNetwork)

---
**Not:** Bu rehber TestNet içindir. MainNet'te değişiklikler olabilir.
