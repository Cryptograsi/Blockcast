# Blockcast Node

Sistem ihtiyaçları çok düşük, Hetzner en düşük sunucuya bile kurulabilir

Öncelikle eğer hala Blockcast üyeliğiniz yoksa [buradan](https://app.blockcast.network?referral-code=tK6Qzf) blockcast kayıt yapın ve sosyal medya görevlerini tamamlayın

## Gerekli Paketlerin Kurulumu
```
sudo apt-get update && sudo apt-get upgrade -y && sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
```

## Docker Kurulumu
```
curl -fsSL https://get.docker.com | sh
```

## Repoyu Klonlayın
```
git clone https://github.com/Blockcast/beacon-docker-compose.git
```

## Nodu Çalıştırın
```
cd beacon-docker-compose
docker compose pull
```

## Blockcast Servisi Çalıştırın
```
docker compose up -d
```

## Nod Bilgilerini Alın
```
docker compose exec blockcastd blockcastd init
```
![Ekran görüntüsü 2025-06-09 204744](https://github.com/user-attachments/assets/8c50d0e2-1b29-42ca-9598-3605dc48d052)

Size bu şekilde bir çıktı verecek. Bunları kopyalayıp bir yere kaydedin, kaybolmasın. Kırmızı ile gösterdiğim bağlantıyı kopyalayıp tarayıcınıza yapıştırın ve nodunuzu manuel olarak kaydedin. Size lokasyon soracak onu da sonraki kodla alacağız.

## Konum Alın
```
sudo apt install jq
curl -s https://ipinfo.io | jq '.city, .region, .country, .loc'
```

Hepsi bu kadar, gerisi site üzerinden nodun kayıt edilmesi.

Ben bazı hatalar aldım, hatayı nasıl çözdüğümü kısaca yazayım, siz de duruma göre aynı yolu izleyebilirsiniz.


1) beacon-docker-compose dizini içindeyken (eğer o dizinden çıktıysanız cd beacon-docker-compose kodu ile dizine gidin) "sudo apt update && sudo apt install docker-compose -y" kodu ile docker compose kurdum
2) Kurulumu doğrulamak için "docker-compose --version" koduyla versiyon kontrol ettim
3) "nano docker-compose.yml" kodu ile docker compose yml dosyasının içine girip yeniden düzenledim
4) yml dosyası içine bunu yaptım (eğer siz de yapacaksanız eskisini silip bunu komple koyalayıp yapıştırın):
```
   x-service: &service
  image: blockcast/cdn_gateway_go:${IMAGE_VERSION:-stable}
  restart: always
  network_mode: "service:blockcastd"
  volumes:
    - ${HOME}/.blockcast/certs:/var/opt/magma/certs
    - ${HOME}/.blockcast/snowflake:/etc/snowflake
    - /var/run/docker.sock:/var/run/docker.sock
  labels:
    - "com.centurylinklabs.watchtower.enable=true"

services:
  control_proxy:
    <<: *service
    container_name: control_proxy
    command: /usr/bin/control_proxy

  blockcastd:
    <<: *service
    container_name: blockcastd
    command: /usr/bin/blockcastd -logtostderr=true -v=0
    network_mode: bridge

  beacond:
    <<: *service
    container_name: beacond
    command: /usr/bin/beacond -logtostderr=true -v=0
    volumes:
      - ${HOME}/.blockcast/configs:/etc/magma/configs
      - ${HOME}/.blockcast/certs:/var/opt/magma/certs
      - ${HOME}/.blockcast/snowflake:/etc/snowflake
      - /var/run/docker.sock:/var/run/docker.sock

  watchtower:
    image: containrrr/watchtower
    environment:
      WATCHTOWER_LABEL_ENABLE: "true"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
5) Bu iki kodla yeniden çalıştırdım:
```
docker-compose down
docker-compose up -d
```
6) Kontrol için "docker logs -f beacond" koduyla loglara baktım
![Ekran görüntüsü 2025-06-09 210254](https://github.com/user-attachments/assets/3fa24398-2b44-42af-b4d4-4f450896959f)
