
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

## Appendix A — Hızlı Komut/Filtre Kopya Kağıdı

**R1**

```txt
show ip interface brief
show arp | include 192.168.10.
show running-config | s interface g0/0
```

**Simulation Filters:** `DHCP`, `UDP`, `IPv4` (+ DNS için `DNS`)
**UDP portları:** Client→Server **68→67**, Server→Client **67→68**
**DHCP alanları:** `giaddr`, `yiaddr`, Options **1/3/6/50/53/54**
**DNS:** **QTYPE=A**, **Answer=A web.local = 192.168.20.100**

---

## Appendix B — Sözlük (kısa)

* **DORA:** Discover, Offer, Request, Ack
* **giaddr:** Relay IP (server’ın hangi subnet için pool seçtiğini belirler)
* **yiaddr:** İstemciye teklif/atanan IP
* **Option 1/3/6:** Subnet Mask / Default Gateway / DNS Server
* **Server Identifier (54):** DHCP sunucusunun kimliği (genelde IP’si)

---


