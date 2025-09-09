# W3 — IP Planlama (Sınıflar vs CIDR, Ağ/Host Hesapları)

## 1) Teori — Kısa Özet
- **Classful (A/B/C)**: Eski yaklaşım (A= /8, B= /16, C= /24 varsayılan). Güncel ağlar **CIDR** kullanır.
- **CIDR (Classless Inter-Domain Routing)**: Ağ maskesi **/prefix** ile gösterilir (ör. **/26**).
- **Formüller:**
  - **Host sayısı** = `2^(32 - prefix) - 2`  *(özel durum: /31 ve /32’de -2 düşmez)*
  - **Blok boyu** (etkilenen oktette) = `256 - maske_okteti`
  - **Network adresi** = IP’yi blok boyunun katına **aşağı yuvarla**
  - **Broadcast** = `network + blok_boyu - 1`
  - **İlk host** = `network + 1`, **Son host** = `broadcast - 1`
  - **Wildcard** (ACL-lar için) = `255.255.255.255 - Subnet mask`

> **Örnek maske ↔ prefix**:  
> /24 → 255.255.255.0 (256 adres, **254 host**)  
> /25 → 255.255.255.128 (128 adres, **126 host**)  
> /26 → 255.255.255.192 (64 adres, **62 host**)  
> /27 → 255.255.255.224 (32 adres, **30 host**)  
> /28 → 255.255.255.240 (16 adres, **14 host**)

---

## 2) Plan-1 — Küçük VLAN (≈50 host): **192.168.10.0/26**
**Neden /26?** `2^(32-26)-2 = 62` host ≥ 50

| Alan | Değer |
|---|---|
| **Prefix / Maske** | **/26** → **255.255.255.192** |
| **Blok boyu** | `256 - 192 = 64`  → alt ağlar: .0, .64, .128, .192 |
| **Seçilen Network** | **192.168.10.0/26** |
| **Network adresi** | **192.168.10.0** |
| **İlk host** | **192.168.10.1** |
| **Son host** | **192.168.10.62** |
| **Broadcast** | **192.168.10.63** |
| **Host sayısı** | **62** |
| **Wildcard** | **0.0.0.63** |

**Not:** Bir sonraki alt ağ **192.168.10.64/26** olur (64’lük sıçrama).

---

## 3) Plan-2 — Orta boy LAN (≈2000–4000 host): **10.20.0.0/20**
**Neden /20?** `2^(32-20)-2 = 4094` host

| Alan | Değer |
|---|---|
| **Prefix / Maske** | **/20** → **255.255.240.0** |
| **Blok boyu (3. oktet)** | `256 - 240 = 16`  → 10.20.**0**.0, 10.20.**16**.0, 10.20.**32**.0 ... |
| **Seçilen Network** | **10.20.0.0/20** |
| **Network adresi** | **10.20.0.0** |
| **İlk host** | **10.20.0.1** |
| **Son host** | **10.20.15.254** |
| **Broadcast** | **10.20.15.255** |
| **Host sayısı** | **4094** |
| **Wildcard** | **0.0.15.255** |

**Not:** Sonraki alt ağ **10.20.16.0/20** (16’lık blok ilerler).

---

## 4) Doğrulama (Validation)
**Kâğıt-kalem:**  
1) **Blok boyu**nu bul (256 − maske_okteti).  
2) IP’nin ilgili oktetini **bloğun en yakın alt katına** indir → **network**.  
3) **Broadcast = network + blok − 1**, **ilk/son host**u çıkar.


---

## 5) Sonuç ve Notlar
- **CIDR**, ihtiyaç kadar host için **en uygun prefix** seçmemizi sağlar (VLSM mantığı).  
- **Baseline** (taban çizgisi): Ağ devreye alındığında ölçtüğün **referans** değerlerdir (ör. ortalama ping, jitter, throughput). İleride sorun olduğunda kıyas için kullan.

---

## DoD (Definition of Done)
- [x] **İki plan** eklendi (küçük /26, orta /20)  
- [x] Hesaplar **tablo** ile gösterildi (network, host aralığı, broadcast, wildcard)  
- [x] **Doğrulama** yöntemi yazıldı (kâğıt-kalem & PT testi)
