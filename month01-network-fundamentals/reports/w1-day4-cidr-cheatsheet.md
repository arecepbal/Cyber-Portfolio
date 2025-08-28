# W1 – Day 4: IPv4 & CIDR Cheatsheet (Binary Dahil)

Bu sayfa; **IPv4’ün 4 oktet yapısı**, **CIDR (/xx)** mantığı, **/24–/30** aralığı için hızlı tablo,  
**ağ/broadcast/ilk–son host** hesaplama yöntemi ve **binary görünümler**i içerir.

---

## 1) IPv4 adres formatı (4 oktet)
- IPv4 = **32 bit** = **4 oktet** × **8 bit**: `A.B.C.D` (örn. `192.168.10.5`)
- Her oktet **0–255** arasıdır [binary gösteriminde 8 bitin son sayısı 255'dir(11111111), 256 ile 9 bite geçilir(100000000)
o sebeble 255 sondur]
- Örnek (ikili gösterim):
  - 192 = `11000010`
  - 168 = `10101000`
  - 10  = `00001010`
  - 5   = `00000101`
- IP’nin binary hali: `11000000.10101000.00001010.00000101`

**Üç özel (private) blok vardır:**
-10.0.0.0/8  toplam adres sayısı: 16,777,216

-172.16.0.0/12   toplam adres sayısı:1,048,574

-192.168.0.0/16  toplam adres sayısı:65,534

İnternette yönlendirilmez; ev/ofis LAN’larında serbestçe kullanılabilir. 
Ev modemleri/DHCP’ler bu yüzden genelde 192.168.0.0/24 veya 192.168.1.0/24 verir. Dış dünyaya çıkarken NAT yapılır.

---

## 2) CIDR /xx nedir?
- **/xx** = maskede **xx bit** **ağ** (network), kalan bitler **host** içindir.
(diğer bi tabirle 32 bitlik bu sistemde **1** olan bitler ağ, **0** olan bitler host içindir)
- Hangi **oktette** çalışacağını böyle belirlersin:
  - **/1–/8  → 1. oktet**
  - **/9–/16 → 2. oktet**
  - **/17–/24 → 3. oktet**
  - **/25–/32 → 4. oktet**

> Örn. **/27** → ilk 3 oktet tamamen ağ (24 bit), 4. oktette **3 ağ biti** daha var → **toplam 27 ağ biti**.

---

## 3) Subnet mask – /xx’den nasıl yazılır?
- Maske = ağ bitleri **1**, host bitleri **0**.
- Oktet başına 8 bit; **1’leri soldan sağa** dizersin.
- Son (kısmi) oktetin değeri şu bit ağırlıklarıyla bulunur: **128, 64, 32, 16, 8, 4, 2, 1**  
  (Hangi bitler 1 ise topla.)

**Örnek: /27 → 255.255.255.224**
- 24 bit (3 oktet) = hep 1 → `255.255.255`
- 4. oktette 3 ağ biti (128+64+32) = **224**
- Binary maske: `11111111.11111111.11111111.11100000`

**/24–/32 için son oktet kısa tablo:**
- /24 → **.0**  
- /25 → **.128** (1 bit)  
- /26 → **.192** (128+64)  
- /27 → **.224** (128+64+32)  
- /28 → **.240** (…+16)  
- /29 → **.248** (…+8)  
- /30 → **.252** (…+4)  
- /31 → **.254** (…+2)  
- /32 → **.255** (…+1)

---

## 4) /24–/30 hızlı tablo (blok boyu & host sayısı)

| CIDR | Maske              | Binary son oktet | **Blok** (artış) | Toplam adres | **Kullanılabilir host** |
|------|--------------------|------------------|------------------|--------------|-------------------------|
| /24  | 255.255.255.0      | 00000000         | **1**            | 256          | **254**                 |
| /25  | 255.255.255.128    | 10000000         | **128**          | 128          | **126**                 |
| /26  | 255.255.255.192    | 11000000         | **64**           | 64           | **62**                  |
| /27  | 255.255.255.224    | 11100000         | **32**           | 32           | **30**                  |
| /28  | 255.255.255.240    | 11110000         | **16**           | 16           | **14**                  |
| /29  | 255.255.255.248    | 11111000         | **8**            | 8            | **6**                   |
| /30  | 255.255.255.252    | 11111100         | **4**            | 4            | **2**                   |

> **Blok (artış)**: ilgili oktette alt ağların başladığı adım.  
> Örn. /27’de 4. oktet **0, 32, 64, 96, …** ile başlar.

---

## 5) Ağ adresi / Broadcast / İlk–Son Host (tarif)
1) **Hangi oktette** çalışacağını belirle (bkz. Bölüm 2).  
2) **Blok = 256 − maskenin o oktet değeri**(/27 de 224)   
3) IP’nin o oktetini **bloğa böl, aşağı yuvarla** → **Ağ** başlangıcı.  
4) **Broadcast = Ağ + (Blok − 1)**  
5) **İlk host = Ağ + 1**, **Son host = Broadcast − 1**  
6) **Kullanılabilir host = (Toplam adres − 2)**  
   - Toplam adres = \(2^{(32−/xx)}\)

**Örnek (tek tek): `192.168.10.37/27`**
- /27 ⇒ 4. oktet, maske `.224`, **Blok = 256−224 = 32**
- 37 → 32’nin katları: 0, **32**, 64, … ⇒ **Ağ = .32**
- 37 **32-64** arasında
- **Ağ = .32**
- **Broadcast = 32 + 31 = .63 daha kısa bi yöntemle 64 - 1 = .63**
- **İlk–Son = .33 ve .62**
- **Host sayısı = 32 − 2 = 30**
- Tam adresler:  
  - **Ağ:** `192.168.10.32`  
  - **Broadcast:** `192.168.10.63`  
  - **İlk–Son host:** `192.168.10.33 – 192.168.10.62`

---

## 6) Binary ile “aynı ağ mı?” (AND işlemi)
Bir host hedefin **yerel mi uzak mı** olduğunu **IP AND Maske** ile test eder.

**Örnek:** `192.168.1.10/24` bir host, `192.168.1.50/24` hedef

- IP (src): `11000000.10101000.00000001.00001010`  
- Maske :   `11111111.11111111.11111111.00000000`  
- **AND →** `11000000.10101000.00000001.00000000` = **192.168.1.0** (ağ)

Hedef (dst) adres + aynı maske ile AND edince yine **192.168.1.0** çıkarsa **aynı ağ**dır → **ARP** yapar, **switch** üzerinden gönderir. Farklı çıkarsa **gateway** gerekir.

---

## 7) /27 için tüm blokların dökümü (son oktet)
1) **0–31**   → Ağ: .0 | Host: **.1–.30** | Broadcast: .31  
2) **32–63**  → Ağ: .32 | Host: **.33–.62** | Broadcast: .63  
3) **64–95**  → Ağ: .64 | Host: **.65–.94** | Broadcast: .95  
4) **96–127** → Ağ: .96 | Host: **.97–.126** | Broadcast: .127  
5) **128–159**→ Ağ: .128 | Host: **.129–.158** | Broadcast: .159  
6) **160–191**→ Ağ: .160 | Host: **.161–.190** | Broadcast: .191  
7) **192–223**→ Ağ: .192 | Host: **.193–.222** | Broadcast: .223  
8) **224–255**→ Ağ: .224 | Host: **.225–.254** | Broadcast: .255

> Not: `.0, .32, .64, …` **ağ adresleri**; `.31, .63, …` **broadcast**’tır.  
> **Host verilemezler.** Hostlar aradaki aralıktadır.

---

## 8) Kısacık sözlük
- **Network bits (ağ bitleri):** Alt ağın kimliğini belirler.  
- **Host bits:** O alt ağ içindeki cihazları numaralar.  
- **Subnet mask:** Hangi bitlerin **ağ** olduğunu gösterir (1’ler ağ, 0’lar host).  
- **Broadcast:** Aynı alt ağdaki **tüm hostlara** gönderim adresi (router’lar genelde iletmez).

- ## 9) “IP AND Mask” – Hızlı Rehber
- AND: iki bit de 1 ise 1; aksi 0.
- Ağ adresi = (IP **AND** Maske) bit-bit.
- 255 → kopyala; 0 → sıfırla; 224/240/248… → bloğa yuvarla.
- /27 örneği: 37 AND 224 = 32 → ağ: 192.168.10.32
- Aynı ağ testi: (IPsrc AND Mask) == (IPdst AND Mask) ? yerel : uzak

---

## 9) Hızlı pratik 
1) `172.16.55.42/28` → Blok? **16** → Ağ **.32**, Broadcast **.47**, İlk–Son **.33–.46**  
2) `192.168.1.140/26` → Blok? **64** → Ağ **.128**, Broadcast **.191**, İlk–Son **.129–.190**  
3) `10.1.203.77/21` → 3. oktet, Blok **8** → Ağ **10.1.200.0**, Broadcast **10.1.207.255**, İlk–Son **.200.1–.207.254**

