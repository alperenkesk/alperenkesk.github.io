---
title: "Nuclei ile Zafiyet Tespiti"
date: 2025-06-15
author: "Alperen Keskin"
description: "Nuclei aracı ile güvenlik zafiyetlerini otomatik olarak tespit etmeyi öğrenin."
tags: ["nuclei", "siber güvenlik", "pentest", "zafiyet", "otomasyon"]
---

# Nuclei ile Zafiyet Tespiti

Nuclei, ProjectDiscovery ekibi tarafından geliştirilen, güvenlik açığı taraması, keşif ve penetrasyon testlerini otomatikleştiren güçlü bir açık kaynak aracıdır. Go diliyle yazılan bu araç, TCP, DNS, HTTP gibi farklı protokoller üzerinden hızlı ve esnek zafiyet taramaları yapabilir. Özelleştirilebilir yapısı sayesinde hem geliştiriciler hem de güvenlik araştırmacıları için popülerdir.

Nuclei'nin en büyük avantajlarından biri, sürekli güncellenen zafiyet şablonlarıyla yaygın güvenlik açıklarını hızlıca tespit edebilmesidir. Ayrıca, diğer güvenlik araçlarıyla entegrasyon sağlayarak otomatik güvenlik testlerini iş akışlarına dahil etmek isteyenler için ideal bir çözüm sunar.

## Nuclei Kurulumu

Nuclei kurulumunu çeşitli yöntemlerle gerçekleştirebilirsiniz. En yaygın üç kurulum yöntemi:

### 1. Go ile Kurulum

Eğer Go dili yüklü ise, Nuclei'yi aşağıdaki komut ile kolayca kurabilirsiniz:

```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

### 2. Docker ile Kurulum

Docker kullanıyorsanız, Nuclei'yi Docker üzerinden şu komutla çalıştırabilirsiniz:

```bash
docker pull projectdiscovery/nuclei:latest
```

### 3. GitHub ile Kurulum

Nuclei'yi kaynak koddan derleyerek kurmak isterseniz, GitHub deposundan aşağıdaki adımlar ile kurabilirsiniz:

```bash
git clone https://github.com/projectdiscovery/nuclei.git
cd nuclei/cmd/nuclei
go build
mv nuclei /usr/local/bin/
nuclei -version
```

Bu yöntemler ile Nuclei'yi sisteminize kolayca kurup kullanıma hazır hale getirebilirsiniz.

## Nuclei Templates

Nuclei kurulumunu tamamladıktan sonra, kullanabilmek için Nuclei Templates'i indirmeniz gerekiyor. Bu şablonlar, Nuclei'nin zafiyet taramaları sırasında kullandığı hazır kurallar ve testlerden oluşur. Nuclei'nin gücü, topluluk tarafından sürekli güncellenen ve katkıda bulunulan bu `nuclei-templates` sayesinde daha da artar.

Nuclei Templates'i indirmek için şu komutu çalıştırabilirsiniz:

```bash
nuclei -update-templates
```

## Şablonlar (Templates)

Nuclei, farklı zafiyet türlerini taramak için YAML tabanlı şablonlar kullanır. Bu şablonlar, güvenlik sorunlarını tespit etmek için gerekli kontrolleri ve kriterleri tanımlar. YAML kullanımı, şablonların okunmasını ve yazılmasını kolaylaştırır; bu sayede, kullanıcılar geniş programlama bilgisine ihtiyaç duymadan kendi özel şablonlarını oluşturabilirler.

### Örnek Şablon (Template)

```yaml
id: zafiyet-ismi

info:
  name: Zafiyet İsmi
  author: sizin-isminiz
  severity: high
  description: Bu şablon, web uygulamalarındaki xxx zafiyeti tespit eder.

requests:
  - method: GET
    path:
      - "{{BaseURL}}/vulnerable-path"
    matchers:
      - type: word
        words:
          - "zafiyet"
        condition: and
        part: body
```

Bu şablon, bir web uygulamasının yanıt gövdesinde belirli bir anahtar kelimenin ("zafiyet") varlığını kontrol ederek zafiyeti tespit etmeye çalışır.

## Nuclei ile Bir Hedefi Taramak

Eğer tek bir URL'yi taramak istiyorsanız, şu şekilde çalıştırabilirsiniz:

```bash
nuclei -u http://example.com -t nuclei-templates/
```

Alternatif olarak:

```bash
nuclei -target http://example.com -t nuclei-templates/
```

## Bir Dosyadan Hedefleri Taramak

```bash
nuclei -l targets.txt -t nuclei-templates/
```

Bu komut, `targets.txt` dosyasındaki tüm URL'leri `nuclei-templates/` dizinindeki şablonlarla tarar.

## Belirli Klasördeki Tüm Şablonları Kullanmak

```bash
nuclei -t nuclei-templates/dast/vulnerabilities/
```

## Belirli Şablonları Kullanmak

```bash
nuclei -t nuclei-templates/dast/vulnerabilities/redirect/open-redirect.yaml
```

## Şiddet (Severity) Türüne Göre Filtreleme

```bash
nuclei -u http://example.com -s critical,high,medium -t nuclei-templates/
```

## Nuclei'yi Diğer Araçlarla Entegre Etmek

```bash
subfinder -d example.com -silent | httpx | nuclei -t nuclei-templates/
```

Bu komut zinciri ile alt alanlar keşfedilip HTTP üzerinden kontrol edilerek zafiyet taraması yapılabilir.

## Nuclei Şablonu Oluşturmak

Nuclei şablonları, belirli güvenlik açıklarını tespit etmek için yazılmış YAML formatındaki dosyalardır.

### Örnek Şablon:

```yaml
id: git-config

info:
  name: Git Config Dosyası
  author: Ice3man
  severity: medium
  description: /.git/config dosyasının varlığını kontrol eder.

http:
  - method: GET
    path:
      - "{{BaseURL}}/.git/config"
    matchers:
      - type: word
        words:
          - "[core]"
```

Yukarıdaki örnekte, "core" kelimesinin matchers bölümünde yer almasının nedeni, .git/config dosyasının bir Git yapılandırma dosyası olduğunu doğrulamaktır. Git yapılandırma dosyalarının içinde "[core]" bölümü, Git'in temel ayarlarını içerir. Bu nedenle, tarayıcı .git/config dosyasına eriştiğinde, dosyanın gerçekten Git yapılandırması olup olmadığını anlamak için "core" kelimesi aranır. Eğer bu kelime bulunursa, hedef sistemde Git yapılandırma dosyasının var olduğu doğrulanır.

## Open Redirect (Açık Yönlendirme) Zafiyeti İçin Nuclei Şablonu Örneği

Aşağıda, bir open redirect zafiyeti için oluşturulmuş bir Nuclei şablonunu bulabilirsiniz. Open redirect, bir kullanıcının güvenilir bir site üzerinden zararlı bir siteye yönlendirilmesine neden olan güvenlik açığıdır.

```yaml
id: open-redirect-zafiyeti

info:
  name: Open Redirect Zafiyeti
  author: sizin-adınız
  severity: low
  description: Hedef sistemde açık yönlendirme zafiyeti mevcut. Belirtilen parametre ile zararlı sitelere yönlendirme yapılabilir.
  reference:
    - https://owasp.org/www-community/attacks/Open_redirect

requests:
  - method: GET
    path:
      - "{{BaseURL}}/redirect?url=https://example.com"

    matchers:
      - type: status
        status:
          - 302
      - type: header
        part: header
        words:
          - "Location: https://example.com"
```

**Açıklama:**

- `id`: Şablonun kimliği. Burada open redirect (açık yönlendirme) için bir şablon oluşturuyoruz.
- `info`: Şablonun açıklama bölümü. Zafiyetin ne olduğunu ve nasıl kullanılabileceğini belirtiyor.
- `requests`: Sunucuya yapılacak istek. `path` kısmında, zararlı bir siteye yönlendirme yapılacak URL tanımlanmıştır.
- `matchers`: Yanıtın HTTP durumu (status) 302 (yönlendirme) olmalı ve Location başlığında zararlı URL (https://example.com) bulunmalıdır.

Bu şablon, sitenin open redirect (açık yönlendirme) zafiyetine sahip olup olmadığını kontrol eder. Eğer hedef site, `https://example.com` adresine yönlendirme yaparsa, bu zafiyetin mevcut olduğu doğrulanır.
