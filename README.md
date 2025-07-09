# Hybrid Integrity Node Kurulum Rehberi

## Adım 1: Sistem Güncellemesi
Sistemi güncelleyin:
```bash
sudo apt update && sudo apt upgrade -y
```

## Adım 2: Docker Kurulumu
Sisteminizde Docker zaten kurulu ise bu adımı atlayın ve kontrol edin:

```bash
docker --version
docker-compose --version
```

Eğer yukarıdaki komutlar çalışıyorsa, Adım 4'e geçin.

### Docker Kurulumu (Sadece kurulu değilse)
Docker'ın eski versiyonlarını temizleyin:
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

Gerekli paketleri yükleyin:
```bash
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
```

Docker'ın resmi GPG anahtarını ekleyin:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Docker repository'sini ekleyin:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Docker'ı yükleyin:
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

Docker servisini başlatın ve otomatik başlatmayı etkinleştirin:
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Kullanıcıyı docker grubuna ekleyin (sudo olmadan docker kullanmak için):
```bash
sudo usermod -aG docker $USER
```

Oturumu yeniden başlatın veya şu komutu çalıştırın:
```bash
newgrp docker
```

## Adım 3: Docker Compose Kurulumu

Docker Compose kurulu mu kontrol edin:
```bash
docker-compose --version
```

Eğer kurulu değilse, Docker Compose'u yükleyin:
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Çalıştırma iznini verin:
```bash
sudo chmod +x /usr/local/bin/docker-compose
```

Kurulumu kontrol edin:
```bash
docker-compose --version
```

## Adım 4: Projeyi Klonlama
GitHub repo'sunu klonlayın:
```bash
git clone https://github.com/buildonhybrid/integrity-node.git
cd integrity-node
```

## Adım 5: Çevre Değişkenleri Ayarları
.env.example dosyasını .env olarak kopyalayın:
```bash
cp .env.example .env
```

.env dosyasını düzenleyin:
```bash
nano .env
```

**.env dosyasındaki örnek içerik:**
```env
RPC_URL=<your_arbitrum_rpc_url>
OPERATOR_PRIVATE_KEY=<your_operator_private_key>
PROTOCOL_CONTRACT_ADDRESS=0x070Af5bcE820f61e409417AfF9b4C1B15B6537df
```

**Bu kısımları kendi bilgilerinizle değiştirin:**
- `<your_arbitrum_rpc_url>` - Arbitrum RPC URL'niz
- `<your_operator_private_key>` - Operator wallet private key'niz

NOT: Operator adresine 0.002 falan fee arb eth gönderin. Operator kaydı için gerekli.

### RPC URL Seçenekleri:
- **Alchemy:** `https://arb-mainnet.g.alchemy.com/v2/YOUR_API_KEY`
- **Infura:** `https://arbitrum-mainnet.infura.io/v3/YOUR_PROJECT_ID`
- **QuickNode:** `https://your-endpoint.arbitrum-mainnet.quiknode.pro/YOUR_TOKEN/`

## Adım 6: Node'u Başlatma
Arka planda çalıştırın:
```bash
docker-compose up -d
```

Logları kontrol edin:
```bash
docker-compose logs -f hybrid-node
```

## Adım 7: Lisans Delegasyonu
Node başarıyla çalıştıktan sonra:
1. https://nodes.buildonhybrid.com/ adresine git
2. **NFT lisans cüzdanınızı** bağla (operator adresinden farklı olabilir)
3. Lisanslarınızı operator adresinize delegate et (Her operator wallet için maksimum 50 lisans)

**ÖNEMLİ:** 
- **Operator adresi:** Node'un çalıştığı cüzdan (`.env` dosyasındaki private key)
- **NFT lisans cüzdanı:** Hybrid NFT lisanslarının bulunduğu cüzdan
- Bu iki cüzdan farklı olabilir, lisansları operator adresine delegate etmeniz gerekir

## Faydalı Komutlar

### Node Durumunu Kontrol Etme
Çalışan container'ları gösterin:
```bash
docker ps
```

Node loglarını izleyin:
```bash
docker-compose logs -f hybrid-node
```

Watchtower loglarını izleyin:
```bash
docker-compose logs -f autoupdate
```

### Node'u Yeniden Başlatma
Durdurun:
```bash
cd integrity-node
docker-compose down
```

Güncelleme ile başlatın:
```bash
docker-compose up -d --pull always
```

### Tamamen Temizlik
Container'ları durdurun ve temizleyin:
```bash
cd integrity-node
docker-compose down
sudo docker system prune -a
```

## Sorun Giderme

### Docker İzin Hatası
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Port Çakışması
Kullanılan portları kontrol edin:
```bash
sudo netstat -tulpn | grep :PORT_NUMBER
```

### Log Kontrolü
Detaylı loglar:
```bash
docker-compose logs --tail=100 hybrid-node
```

Hata logları:
```bash
docker-compose logs hybrid-node 2>&1 | grep -i error
```
