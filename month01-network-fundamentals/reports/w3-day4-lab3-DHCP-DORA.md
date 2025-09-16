
---

# W3 – DHCP Relay (DORA) + DNS Çözümleme (PT)

## Goal

* **DHCP DORA** akışını (Discover → Offer → Request → Ack) **relay** üzerinden **paket seviyesinde** kanıtlamak.
* **Doğru havuz seçimi** (giaddr) ve **Option 1/3/6** (Mask/Gateway/DNS) alanlarının **Offer/Ack** içinde doğru geldiğini göstermek.
* **DNS A kaydı** ile isim→IP çözümünü doğrulamak.

---

## Setup (Topology)

**Topoloji (2 LAN + Router + Server):**

* **LAN10 (Client)**

  * PC-A — `Fa0` → **SW2**
  * R1 `g0/0` → **SW2**
* **LAN20 (Server)**

  * Server-PT (DHCP+DNS\[+HTTP ops.]) — `Fa0` → **SW1**
  * R1 `g0/1` → **SW1**

**IP Planı**

* **R1 `g0/0` (LAN10):** `192.168.10.1/24`  ← *client tarafı, helper burada*
* **R1 `g0/1` (LAN20):** `192.168.20.1/24`
* **Server-PT `Fa0`:** `192.168.20.100/24`, **GW:** `192.168.20.1`
* **PC-A:** DHCP (IP’yi LAN10 havuzundan alacak)

**DHCP Havuzları (Server-PT → Services → DHCP → On)**

* **Pool: LAN10 (gerekli)**

  * **Default Gateway (Opt.3):** `192.168.10.1`
  * **DNS Server (Opt.6):** `192.168.20.100` *(veya tercih ettiğin DNS)*
  * **Start IP Address:** `192.168.10.100`
  * **Subnet Mask (Opt.1):** `255.255.255.0`
  * **Maximum Users:** `50`
* **Pool: LAN20 (opsiyonel)**

  * **Default Gateway:** `192.168.20.1`
  * **DNS:** `192.168.20.100`
  * **Start IP:** `192.168.20.100`, **Mask:** `255.255.255.0`

**DNS (Server-PT → Services → DNS → On)**

* **A kaydı:** `web.local → 192.168.20.100`
* (Ops.) **HTTP (On)**: `index.html = "OK"`

**Router R1**

```txt
enable
conf t
interface g0/0
 description LAN10 (client side)
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.20.100
 no shut
!
interface g0/1
 description LAN20 (server side)
 ip address 192.168.20.1 255.255.255.0
 no shut
end
```

**Switch Notu:** PC ve R1 g0/0 aynı switch/VLAN’da (default VLAN 1) olmalı.

---

## Steps (Runbook)

### 1) DORA’yı yakala (Simulation)

1. **Simulation** moduna geç → **Edit Filters**: **DHCP, UDP, IPv4** açık.

2. **Event List → Clear.**

3. PC-A **IP Configuration**: **Static → DHCP** yap (**tetik**).

4. Event List’teki **Discover/Offer/Request/Ack** paketlerini sırayla aç → **PDU Details**’ta **BOOTP/DHCP** bölümünü incele:

   * **Discover (Client→Bcast, UDP 68→67)**

     * **giaddr = 192.168.10.1**  *(relay doğru arayüzde)*
     * **chaddr = PC MAC**, **ciaddr = 0.0.0.0**, **Message Type = Discover**
   * **Offer (Server→Client, UDP 67→68)**

     * **yiaddr = 192.168.10.x**  *(doğru subnet teklifi)*
     * **Router/Gateway = 192.168.10.1** (Opt.3) 
     * **DNS Server = 192.168.20.100** (Opt.6) 
     * **Subnet Mask = 255.255.255.0** (Opt.1)
     * **Server Identifier = 192.168.20.100** (Opt.54)
   * **Request (Client→Bcast)**

     * **Requested IP = 192.168.10.x** (Opt.50)
     * **Server ID = 192.168.20.100** (Opt.54)
   * **Ack (Server→Client)**

     * **yiaddr = 192.168.10.x**
     * **Opt.1/3/6** tekrar görünür (PT sürümüne bağlı olarak numara/metin)

5. PC-A **Command Prompt** → `ipconfig` çıktısını kaydet.

### 2) DNS’i kanıtla

1. PC-A → `ping web.local` → başlıkta **`[192.168.20.100]`** görünmeli.
2. **Simulation** filtrelerine **DNS** ekle → **Query/Response** paketlerini aç:

   * **Query:** `QNAME=web.local`, **QTYPE=A(1)\`**
   * **Response:** **Answer: A web.local = 192.168.20.100**

### 3) (Ops.) Negatif test

* R1 `g0/0`’dan `no ip helper-address 192.168.20.100` yap → PC **IP alamaz** (Discover’lar yanıtsız kalır).
* Tekrar ekleyip normal duruma dön.

---

## Evidence (ekran görüntüsü/artefact)

> `diagrams/` ve `artefacts/` klasörlerine koy.

* `diagrams/dhcp_discover_giaddr.png` — **Discover** PDU’da **giaddr=192.168.10.1** işaretli
* `diagrams/dhcp_offer_options.png` — **Offer**’da **yiaddr/Router/DNS/Mask** işaretli
* `diagrams/dhcp_ack.png` — Ack özeti
* `artefacts/ipconfig_pc.txt` — PC `ipconfig` çıktısı (IP/GW/DNS)
* `diagrams/dns_query.png` — DNS Query (A `web.local`)
* `diagrams/dns_response.png` — DNS Answer (IP = `192.168.20.100`)

---

## Findings (yorum)

* **Relay şartı kanıtlandı:** **Discover** yayını **router’ı geçmedi**; R1 **giaddr=192.168.10.1** ile relay edince server **LAN10 havuzunu** seçip doğru **GW (192.168.10.1)** ve **DNS (192.168.20.100)** dağıttı.
* **Doğru opsiyonlar:** Offer/Ack’te **Opt.1/3/6** (mask/gw/dns) doğrulandı; **UDP 68↔67** portları doğru.
* **DNS A kaydı** başarılı: `web.local` → `192.168.20.100`, uygulama seviyesinde `ping/http` ile doğrulandı.

---

## Troubleshooting Notes

* **ARP cevap yoksa:** PC ve R1 g0/0 aynı VLAN’da mı? **PC mask** doğru mu? SW portları **up** mı?
* **Yanlış GW/DNS geldiyse:** **Helper yanlış arayüzde** olabilir (g0/1’e yazma). **LAN10 pool’u** eksik/yanlış olabilir.
* **PT’de opsiyon numaraları görünmüyorsa:** Aynı alanlar **metin** olarak çıkar (“Router: …”, “DNS: …”). Bu normal.

---

## DoD (Definition of Done)

* [ ] **DORA** 4 adımı **Simulation**’da paket paket gösterildi (SS eklendi)
* [ ] **Discover** PDU’da **giaddr=192.168.10.1** kanıtlandı
* [ ] **Offer/Ack**’te **yiaddr=192.168.10.x**, **Gateway=192.168.10.1**, **DNS=192.168.20.100**, **Mask=255.255.255.0** görüldü
* [ ] PC `ipconfig`: **IP 192.168.10.x / GW 192.168.10.1 / DNS 192.168.20.100**
* [ ] **DNS** Query/Response ve **Answer: A web.local = 192.168.20.100** ekran görüntüsü
* [ ] (Ops.) **Helper kapalıyken başarısız**, açıkken başarılı karşılaştırması

---
# DHCP — DORA Akışı ve DHCP Options (Kopyala-Yapıştır .md)

**Amaç:** DHCP’nin **DORA** (Discover → Offer → Request → Ack) akışında *paketlerde mutlaka/çoğu zaman görmen gereken alanlar* ile **DHCP Options** (kod → anlam) özetini tek sayfada toplamak.  
**Not:** “MUST/SHOULD/OPTIONAL” ifadeleri RFC 2131/2132 mantığına göredir. Packet Tracer bazen opsiyon adını yazar (kodu göstermeyebilir); Wireshark hem isim hem kodu gösterir.

---

## DORA — Paket Alanları (Özet)

### 1) **DHCPDISCOVER** (İstemci → **broadcast**, UDP **68→67**)
**BOOTP/DHCP başlık alanları (zorunlu):**
- `op=1` (request), `htype=1` (Ethernet), `hlen=6`
- `xid` (transaction id), `secs`, `flags` (**0x8000** = broadcast biti)
- `ciaddr=0.0.0.0`, `yiaddr=0`, `siaddr=0`
- `chaddr` = istemci **MAC**
- `giaddr` = **relay** IP (VARSA; istemciye yakın router ara yüzü)

**Opsiyonlar:**
- **53** = **1** (Message Type: Discover) — **MUST**
- **55** (Parameter Request List) — **SHOULD** (genelde 1/3/6/51/…)
- **61** (Client Identifier) — **SHOULD**
- **12** (Host Name) — **OPTIONAL**
- **50** (Requested IP) — **OPTIONAL** (INIT-REBOOT’ta)
- **60** (Vendor Class) — **OPTIONAL**

---

### 2) **DHCPOFFER** (Sunucu → istemci, UDP **67→68**)
**Başlık:**
- `op=2` (reply), `yiaddr` = **teklif edilen IP**
- Relay varsa `giaddr` korunur.

**Opsiyonlar:**
- **53** = **2** (Offer) — **MUST**
- **54** (Server Identifier) = sunucu IP — **MUST**
- **1** (Subnet Mask), **3** (Router), **6** (DNS), **51** (Lease) — **SHOULD**
- **58/59** (T1/T2) — **RECOMMENDED**
- **15/119** (Domain/Domain Search), **121/249** (Classless Routes) — **OPTIONAL**

---

### 3) **DHCPREQUEST** (İstemci → **broadcast**, UDP **68→67**)
**Varyantlar:** SELECTING (Offer seçimi), INIT-REBOOT, RENEWING/REBINDING

**SELECTING (en yaygın):**
- **53** = **3** (Request) — **MUST**
- **50** (Requested IP) = teklif edilen IP — **MUST**
- **54** (Server ID) = seçilen sunucu — **MUST**
- `ciaddr=0.0.0.0` (IP henüz yok)

**RENEWING/REBINDING:**
- `ciaddr` = **mevcut IP** (doldurulur)
- **50/54** olmayabilir (sunucu seçimi yapılmıyor; kira yenileniyor)

---

### 4) **DHCPACK** (Sunucu → istemci, UDP **67→68**)
**Başlık:**
- `op=2` (reply), `yiaddr` = **atanan IP**

**Opsiyonlar:**
- **53** = **5** (Ack) — **MUST**
- **54** (Server ID) — **MUST**
- **1/3/6/51** (Mask/GW/DNS/Lease) — **SHOULD**
- **58/59**, **15/119**, **121/249** — **OPTIONAL**

> **Not:** **DHCPNAK** (53=6) gelirse istemci başa döner (yeniden Discover).

---

## DHCP Options — Hızlı Sözlük (DEC & HEX)

> Kodlar belgelerde genelde **DEC** yazılır; pakette 1-byte **HEX** görülür (ör. 53 dec = **0x35** hex).

| Kod (dec) | Kod (hex) | Adı | Kısa açıklama |
|---:|:---:|---|---|
| **1** | 0x01 | Subnet Mask | Alt ağ maskesi (örn. 255.255.255.0) |
| **3** | 0x03 | Router (Default GW) | Varsayılan ağ geçidi(leri) |
| **6** | 0x06 | DNS Servers | DNS sunucuları |
| **12** | 0x0C | Host Name | İstemci makine adı |
| **15** | 0x0F | Domain Name | Yerel etki alanı adı |
| **28** | 0x1C | Broadcast Address | Ağın yayın adresi |
| **42** | 0x2A | NTP Servers | Zaman sunucuları |
| **50** | 0x32 | Requested IP Address | İstenilen IP (REQUEST’te) |
| **51** | 0x33 | Lease Time | Kira süresi (saniye) |
| **52** | 0x34 | Option Overload | sname/file alanlarını opsiyonlar için kullan |
| **53** | 0x35 | DHCP Message Type | 1=Discover, 2=Offer, 3=Request, 5=Ack, 6=Nak, 7=Release, 8=Inform |
| **54** | 0x36 | Server Identifier | Sunucu kimliği (genelde IP) |
| **55** | 0x37 | Parameter Request List | İstemcinin istediği opsiyon listesi |
| **56** | 0x38 | Message | İnsan tarafından okunur metin/hata |
| **57** | 0x39 | Maximum DHCP Msg Size | Maksimum mesaj boyutu |
| **58** | 0x3A | T1 (Renewal Time) | Yenileme başlangıcı |
| **59** | 0x3B | T2 (Rebinding Time) | Yeniden bağlanma süresi |
| **60** | 0x3C | Vendor Class ID | Örn. “MSFT 5.0” |
| **61** | 0x3D | Client Identifier | İstemci kimliği (çoğunlukla MAC tabanlı) |
| **66** | 0x42 | TFTP Server Name | PXE/boot |
| **67** | 0x43 | Bootfile Name | Boot imaj dosyası |
| **82** | 0x52 | Relay Agent Info | Option 82 (Circuit-Id, Remote-Id) |
| **119** | 0x77 | Domain Search | DNS arama dizisi |
| **121** | 0x79 | Classless Static Routes | RFC3442, sınıfsız rota listesi |
| **249** | 0xF9 | MS Classless Routes | MS varyant (121 ile benzer) |
| **0** | 0x00 | Pad | Dolgu (değer yok) |
| **255** | 0xFF | End | Opsiyonların sonu |

---

## “Decimal mi, Hex mi?” — Hızlı Notlar
- Doküman/konfig: **DEC** (53, 54…).  
- Paket içi byte alanı: **HEX** (53 → **0x35**, 54 → **0x36**, 50 → **0x32**…).  
- **Wireshark** iki biçimi de gösterir: *“Option: (53) DHCP Message Type”*.  
- **Packet Tracer** bazen yalnızca adını gösterir (kod görünmeyebilir).

**Mini dönüşüm kopyası:**  
`1→0x01, 3→0x03, 6→0x06, 50→0x32, 51→0x33, 52→0x34, 53→0x35, 54→0x36, 55→0x37, 56→0x38, 57→0x39, 58→0x3A, 59→0x3B, 82→0x52, 119→0x77, 121→0x79, 249→0xF9, 255→0xFF`

---

## PDU Kontrol “Kopya Kâğıdı” (Lab’a koymalık)

- **Discover:** `op=1`, `ciaddr=0`, `chaddr=MAC`, `giaddr=<relay IP (varsa)>`, `53=1`, `55=…`  
- **Offer:** `op=2`, `yiaddr=<teklif IP>`, `53=2`, `54=<server IP>`, `1/3/6/51/58/59 (varsa)`  
- **Request (Selecting):** `op=1`, `ciaddr=0`, `53=3`, `50=<offered IP>`, `54=<server ID>`  
- **Ack:** `op=2`, `yiaddr=<atanan IP>`, `53=5`, `54=<server ID>`, `1/3/6/51/58/59 (varsa)`  
- **Relay varsa:** `giaddr` her cevapta korunur; sunucu **giaddr**’a göre doğru havuzu seçer.

---

## Wireshark/Filtre İpuçları
- **Tüm DHCP/BOOTP:** `bootp`  
- **Yalnızca Discover:** `bootp.option.dhcp == 1`  
- **Yalnızca Offer:** `bootp.option.dhcp == 2`  
- **Yalnızca Request:** `bootp.option.dhcp == 3`  
- **Yalnızca Ack:** `bootp.option.dhcp == 5`

> **Hatırla:** DORA yayın (broadcast) adımlarında L2’de **ff:ff:ff:ff:ff:ff**, L3’te **255.255.255.255** görürsün; relay (Option 82/`giaddr`) kullanıyorsan yayınlar segment/relay mantığıyla taşınır.
