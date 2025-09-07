# W2 — Throughput vs Latency & Jitter (HTTP mini-dosya)

**BU LAB PT'de BAZI HESAPLAR YAPILAMADIĞI İÇİN YAPILMAMIŞTIR**

## GOAL
Aynı LAN’da küçük bir HTTP dosyası indirerek **throughput** ölçmek; **ping** ile **ortalama latency** ve **jitter** kıyaslamak. 10 Mb/s ve 100 Mb/s hız ayarlarıyla farkı göstermek.

## SETUP
- PC ↔ SW2960 ↔ Server-PT (aynı VLAN/Subnet)
- IP: PC `192.168.10.10/24`, Server `192.168.10.2/24`
- HTTP açık; dosya: `small.txt` (**boyut: … bayt**)

## STEPS
1) Switch port hızları **10 Mb/s** yapıldı (fa0/1, fa0/2 → speed 10).
2) PC’den `http://192.168.10.2/small.txt` indirildi; **süre = … s**.
3) **Throughput_10Mb** = (**bayt** × 8) / **süre** / 1_000_000 = **… Mb/s**.
4) Hız **100 Mb/s**’e alındı; aynı dosya indirildi; **süre = … s** → **Throughput_100Mb = … Mb/s**.
5) `ping 192.168.10.2 -n 20` → **Avg RTT = … ms**, **Jitter ≈ … ms**.

## Ölçüm Tablosu
| Senaryo | Dosya Boyutu (B) | Süre (s) | Throughput (Mb/s) | Avg RTT (ms) | Jitter ~ (ms) |
|---|---:|---:|---:|---:|---:|
| 10 Mb/s | … | … | … | … | … |
| 100 Mb/s | … | … | … | … | … |

## Yorum (Bandwidth ≠ Throughput ≠ Goodput)
- Bandwidth link kapasitesidir (10/100 Mb/s). Throughput ise pratikte ölçülen bit hızıdır; protokol başlıkları ve beklemeler nedeniyle **bandwidth’den düşük** çıktı (**… vs … Mb/s**).
- Bu testte tekrar iletim yoktu; bu yüzden **goodput ≈ throughput** kabul edildi. Büyük dosya / kayıplı hatlarda **goodput** daha da düşer.


## DoD (Definition of Done)
- [ ] Ölçüm tablosu dolduruldu (10 Mb/s & 100 Mb/s)
- [ ] “Bandwidth≠Throughput≠Goodput” yorumu yazıldı


## ChatGPT ile yapılmış LAB

# 1×PC + 1×2960 Switch + 1×Server ile **Throughput** / **Goodput** Hesabı (örnekli)

Bu doküman **tek PC – tek switch – tek server** (aynı LAN) senaryosunda,
Packet Tracer’daki `Server0` üstünde **HTTP** ile çok küçük bir dosyayı
(**"ABCDEFGHIJKLMNOPRSTUVYZ" = 23 B = 184 bit**) indirirken
**throughput** ve **goodput**’u nasıl **elle hesaplayacağını** gösterir.

---

## 0) Tanımlar (kısaca)
- **Throughput (hat üstü hız):** Hattan geçen **tüm** bitler / süre  
  (L2/L3/L4 başlıkları, preamble/SFD, IFG, HTTP başlıkları, el sıkışmalar **dahil**)
- **Goodput (uygulama faydası):** **Sadece uygulama gövdesi** (payload) bitleri / süre  
  (HTTP gövdesi; başlıklar, tekrarlar **hariç**)

Payload’ımız:  
"ABCDEFGHIJKLMNOPRSTUVYZ" = 23 byte = 184 bit

---

## 1) Katmanlara göre **hat üstü** byte’lar (formüller)

Aşağıdaki değerler **VLAN etiketsiz, IPv4/TCP** içindir.

- **Ön-ekler / son-ekler (her çerçevede):**
  - Preamble+SFD = **8 B**
  - IFG (Inter Frame Gap) = **12 B**
- **Başlıklar:**
  - Ethernet header = **14 B**  (Dst MAC + Src MAC + EtherType)
  - IPv4 header = **20 B**      (opsiyonsuz)
  - TCP header = **20 B**       (opsiyonsuz)
  - FCS = **4 B**
- **ARP çerçevesi payload’ı:** **28 B**

> **Boş TCP segment (SYN/ACK/ACK vb.) on-wire**  
> `8 + 14 + 20 + 20 + 4 + 12 = 78 B`

> **Tek ARP çerçevesi on-wire**  
> `8 + 14 + 28 + 4 + 12 = 66 B`

> **HTTP yanıt çerçevesi on-wire (tek segment varsayımı)**  
> `8 + 14 + 20 + 20 + H + 23 + 4 + 12 = (H + 101) B`  
> Burada **H = HTTP response headers** bayt sayısı.

> **HTTP GET isteği çerçevesi on-wire (tek segment varsayımı)**  
> `8 + 14 + 20 + 20 + G + 0 + 4 + 12 = (G + 78) B`  
> Burada **G = HTTP GET headers** bayt sayısı.

---

## 2) **Toplam** on-wire boyut (tek istek/yanıt için)

**3-yollu TCP el sıkışması:** `3 × 78 = 234 B`

**GET isteği:** `G + 78 B`  
**HTTP yanıtı:** `H + 101 B`  (gövde = 23 B)

- **ARP gerekiyorsa (cache boş):** `2 × 66 = 132 B` ekle
- **ARP gerekmezse (cache dolu):** ekleme yok

**Toplam (ARP **yok**):**
Total_no_ARP = 234 + (G + 78) + (H + 101) = G + H + 413 (byte)

**Toplam (ARP **var**):**
Total_with_ARP = 132 + [G + H + 413] = G + H + 545 (byte)

> Not: PT’de `G` ve `H` değerlerini **Simulation** modunda, PDU detaylarında
> HTTP kısmındaki metni kabaca sayarak (veya Wireshark’ta Length alanından) tahmini alabilirsin.
> Tipik küçük bir GET: **G ≈ 80 B**, küçük yanıt başlığı: **H ≈ 120 B**.

---

## 3) **ÖRNEK HESAP** (1×PC—2960—Server, 100 Mb/s, ARP **var**)

**Varsayımlar:**
- Link hızı `R = 100 Mb/s`
- `G = 80 B` (GET başlıkları)
- `H = 120 B` (Response başlıkları)
- ARP cache **boş** (ilk bağlantı)

**Toplam on-wire:**


> Not: PT’de `G` ve `H` değerlerini **Simulation** modunda, PDU detaylarında
> HTTP kısmındaki metni kabaca sayarak (veya Wireshark’ta Length alanından) tahmini alabilirsin.
> Tipik küçük bir GET: **G ≈ 80 B**, küçük yanıt başlığı: **H ≈ 120 B**.

---

## 3) **ÖRNEK HESAP** (1×PC—2960—Server, 100 Mb/s, ARP **var**)

**Varsayımlar:**
- Link hızı `R = 100 Mb/s`
- `G = 80 B` (GET başlıkları)
- `H = 120 B` (Response başlıkları)
- ARP cache **boş** (ilk bağlantı)

**Toplam on-wire:**
Total_with_ARP = G + H + 545 = 80 + 120 + 545 = 745 B
Toplam bit = 745 × 8 = 5,960 bit


**Süre (yalnızca seriizasyon, gecikmeleri ihmal):**
t_total = 5,960 / 100,000,000 = 59.6 µs


**Goodput (uygulama verisi / süre):**
Goodput = 184 bit / 59.6e-6 s ≈ 3.09 Mb/s

**Throughput (bu pencere için hat üstü):**
Throughput ≈ Toplam bit / t_total = 5,960 / 59.6e-6 = 100 Mb/s # (link hızı)

**Oran:**
Goodput / Throughput = 184 / 5,960 ≈ 3.1 %


---

## 4) **ÖRNEK HESAP** (100 Mb/s, ARP **yok** — ikinci istekte)

**Toplam on-wire:**
Total_no_ARP = G + H + 413 = 80 + 120 + 413 = 613 B
Bit = 613 × 8 = 4,904 bit
t_total = 4,904 / 100e6 = 49.04 µs
Goodput = 184 / 49.04e-6 ≈ 3.75 Mb/s
Goodput/Throughput = 184 / 4,904 ≈ 3.75 %


> Gördüğün gibi **ARP yokken** oran biraz iyileşir.
> Linki **1 Gb/s** yapsan **oran aynı** kalır, sadece değerler **10×** büyür.

---

## 5) Nereden hangi sayıyı alacağım? (PT’de pratik)
1. **Simulation** moduna geç, filtrede **HTTP/TCP/IPv4/Ethernet** açık kalsın.  
2. `PC`’den `http://server-ip`’e git.  
3. **GET** ve **Response** PDU’larına sırayla tıkla:
   - **Application/HTTP** alanındaki metni göreceksin. Buradaki karakter sayısını
     yaklaşık sayıp **G** ve **H** için bayta çevir (ASCII/UTF-8 → 1 char ≈ 1 B).
4. ARP olup olmadığına bak:
   - **Event List**’te ARP Request/Reply görürsen **132 B** ekle.
5. Üstteki formüllere **G** ve **H**’yi koy → byte → bit → süre (bit / `R`) → oran.

---

## 6) Mini “Hızlı Hesap” kartı
- **Handshake (3×):** `3 × 78 = 234 B`
- **GET:** `G + 78 B`
- **Response:** `H + 101 B`  (payload = 23 B = 184 bit)
- **ARP (ops.):** `+132 B`
- **Toplam:**  
  - `no_ARP = G + H + 413 B`  
  - `with_ARP = G + H + 545 B`
- **Süre:** `t = (Toplam_Byte × 8) / R`
- **Goodput:** `184 / t`  
- **Oran:** `184 / (Toplam_Byte × 8)`

> **Düzeltme notu:** Bazı kaynaklar **preamble/IFG**’yi saymaz (fiziksel katman).  
> Biz burada **on-wire gerçek bitleri** gördüğün için **dahil ettik**.

---
