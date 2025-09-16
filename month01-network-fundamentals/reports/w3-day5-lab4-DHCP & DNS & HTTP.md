# W3-Lab2 — DHCP & DNS & HTTP (Single LAN)

> **Amaç (Goal)**  
> - PC’lerin **DHCP ile otomatik IP** almasını (DORA) kanıtlamak.  
> - **DNS** ile `web.local` adını **HTTP sunucuya** çözmek ve erişmek.

---

## 1) Setup (Topology)

**Cihazlar:** 1×2960 Switch, 1×Server-PT (DHCP+DNS+HTTP), 2×PC (PC-A, PC-B)  
**Kablolar:** Copper straight-through (PC↔SW, Server↔SW)  
**VLAN:** Varsayılan (VLAN 1), tek yayın alanı

### IP Plan (tek subnet)
| Bileşen | Arayüz | IP/Mask | Not |
|---|---|---|---|
| Server-PT | Fa0 | `192.168.50.10/24` | GW **boş** (tek LAN’da gerekmez) |
| PC-A | Fa0 | **DHCP** | Pool’dan alacak |
| PC-B | Fa0 | **DHCP** | Pool’dan alacak |

### Sunucu Servisleri (Server-PT → Services)
- **DHCP → On**  
  - **Pool Name:** `LAN10`  
  - **DNS Server:** `192.168.10.100`  
  - **Start IP:** `192.168.10.1`  
  - **Subnet Mask:** `255.255.255.0`  
  - **Default Gateway:** *(boş bırakabilirsiniz)*  
  - **Maximum Users:** `50`  
  - **Add/Save**
- **DNS → On**  
  - **A kaydı:** `web.local` → `192.168.10.100`
- **HTTP → On**  
  - (İsteğe bağlı) `index.html = "OK"`

> **Not:** İki subnet/relay kullanıyorsanız kısayol: R1 client tarafında  
> `ip helper-address <ServerIP>` ve **client LAN pool’unda** `Gateway` alanını doldurun.

---

## 2) Steps (Runbook)

### A) DHCP Lease (PC-A)
1. **PC-A → Desktop → IP Configuration:** **DHCP** seç.  
2. **Doğrula:** **Command Prompt** → `ipconfig /all`  
   - IP: `192.168.10.1+`, Mask: `255.255.255.0`, DNS: `192.168.10.100`

*(İstersen Simulation modunda DHCP/UDP/IPv4 filtreleriyle DORA akışını izle.)*

### B) DNS Çözümleme
1. **PC-A → Command Prompt:** `nslookup web.local`  
   - **Name:** `web.local`, **Address:** `192.168.10.100` beklenir.
2. Alternatif: `ping web.local` → köşeli parantezde IP görünür.

### C) HTTP Erişim
1. **PC-A → Web Browser:** `http://web.local/`  
   - “OK” veya varsayılan sayfa açılmalı.

### D) (Opsiyonel) PC-B ile çapraz doğrulama
- Aynı üç adımı PC-B için tekrarla; **aynı DNS** ve **HTTP erişimi** beklenir.

---

## 3) Evidence (ekleyeceğin dosyalar)

- `diagrams/w3-lab2-dhcp-lease.png`  ← PC-A `ipconfig /all` ekran görüntüsü  
- `diagrams/w3-lab2-nslookup.png`     ← `nslookup web.local` çıktısı  
- `diagrams/w3-lab2-http.png`         ← `http://web.local/` açılmış hali  
- `lab/w3-lab2.pkt`                   ← PT proje dosyası

> **Gizlilik notu:** Ekran görüntülerinde **Host Name (Opt 12)**, **Client Identifier (Opt 61)**, **MAC** gibi kişisel bilgileri maskeleyin.

---

## 4) Findings (Kısa Yorum)
- **DHCP havuzundan** `192.168.10.x` IP atandı; **Option 6 (DNS)** doğru (`192.168.10.10`).  
- `web.local` → `192.168.10.10` doğru çözüldü; **HTTP** katmanında erişim başarılı.  
- Tek LAN topolojisinde **gateway zorunlu değil**; DNS/HTTP iç LAN’da çalışır.

---

## 5) Troubleshooting (Hızlı Rehber)
- **DNS gelmiyor/n görünmüyor:**  
  - DHCP Pool’da **DNS=192.168.50.10** yazılı ve **Save** edildi mi?  
  - **Server-PT → DHCP:** Servisi **Off→On** yap, PC’de **Static→DHCP** ile lease yenile.  
  - `nslookup web.local` çalışıyorsa, GUI alanı görünmese bile DNS dağıtımı fonksiyoneldir.
- **HTTP açılmıyor:**  
  - Server-PT’de **HTTP On** mu? **DNS A kaydı** doğru IP’ye mi?  
  - PC ile Server aynı switch/VLAN’da mı? Port LED’leri **yeşil** mi?
- **IP çakışması:**  
  - Start IP’yi `.100` gibi güvenli aralığa taşı; Server IP ile çakıştırma.

---

## 6) DoD (Definition of Done)
- [ ] **DHCP lease ss:** PC-A `ipconfig /all` (IP/Mask/DNS görünüyor)  
- [ ] **nslookup/isim çözümleme kanıtı:** `nslookup web.local` çıktısı  
- [ ] **HTTP erişim:** `http://web.local` erişim ekran görüntüsü (opsiyonel ama önerilir)

---

## 7) (Ek) Simulation’da DORA’yı görme (opsiyonel)
- **Simulation → Edit Filters:** DHCP/UDP/IPv4 → **On**  
- PC-A’yı **Static→DHCP** yap → **Discover → Offer → Request → Ack** paketlerini sırayla aç:  
  - **Offer/Ack**’ta: **yiaddr=192.168.10.x**, **Option 6 (DNS)=192.168.10.10**  
  - (PT bazı opsiyonları numara yerine metin olarak gösterebilir; bu normal.)

---

## 8) Sözlük (kısa)
- **DORA:** Discover → Offer → Request → Ack (DHCP atama akışı)  
- **yiaddr:** “Your IP address” – istemciye teklif/atanan IP  
- **Option 6:** DNS sunucusu (DHCP ile dağıtılır)  
- **A kaydı:** DNS’te ad → IPv4 adres eşlemesi (örn. `web.local` → `192.168.10.10`)
