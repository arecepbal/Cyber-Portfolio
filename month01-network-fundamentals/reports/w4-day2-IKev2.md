
# W4 – Gün 2 · IKEv2 + ESP Wireshark İncelemesi (PCAP Raporu)

> ## 1)Goal (Amaç)  
> - IKEv2 el sıkışmasını (**IKE_SA_INIT → IKE_AUTH**) paket seviyesinde doğrulamak.  
> - **ESP** veri akışının varlığını ve **SPI**’ları tespit etmek.  
> - **NAT-T** kullanımını kanıtlamak (UDP/4500, Non-ESP Marker).  
> - **Inner vs Outer** başlık ayrımını kavramsal olarak açıklamak.  

---

## 2)Steps (Adımlar)

1) **Dosyayı aç**  
   - PCAP: `artefacts/pcap/ikev2_sample.pcapng`
   
2) **Filtre uygula** (tam akış görünümü)  
   ```wireshark
   ikev2 or esp or udp.port==4500

3. **INIT/AUTH’ı ayrı ayrı göster**

   * Yalnız **IKE\_SA\_INIT** (Exchange Type **34**):

     ```wireshark
     isakmp.exchange_type == 34)
     ```
   * Yalnız **IKE\_AUTH** (Exchange Type **35**):

     ```wireshark
     isakmp.exchange_type == 35
     ```
   * (Ops.) **INFORMATIONAL** (Exchange Type **37**):

     ```wireshark
     isakmp.exchange_type == 37
     ```

4. **NAT-T’yi kanıtla**

   * Sadece ESP (UDP-encap) için:

     ```wireshark
     udp.port==4500 && esp
     ```
   * Bir ESP paketini aç → `Non-ESP Marker = 00 00 00 00` satırını gör.

  
5. **SPI yönlerini ayır**

   * İlk iki ESP paketten **SPI (Initiator→Responder / Responder→Initiator)** değerlerini kopyala.
   * Her yön için filtre:

     ```wireshark
     esp.spi == 0xc1a9656b   # I→R 
     esp.spi == 0xac0faf03   # R→I
     ```

6. **Sütun (Column) ekle**

   * Paketi aç → orta panelde ilgili alan üzerinde **sağ tık → Sütun olarak uygula**:

     * `IKEv2 → Exchange type`
     * `IKEv2 → Message ID`
     * `ESP → Security Parameters Index (SPI)`
     * `UDP → Kaynak/Hedef bağlantı noktası`
   * Alternatif: **Düzenle → Tercihler → Görünüm → Sütunlar → (+) Özel (Custom)**

     * Alan adları: `ikev2.exchange_type`, `ikev2.msgid`, `esp.spi`, `udp.srcport`, `udp.dstport`

7. **Zaman akışı (Flow Graph)**

   * **İstatistikler → Akış Grafiği…** → **Görüntülenen paketler** → **Tamam**.
   * Şu sırayı gözle: **INIT → AUTH → (sonra) ESP**.

8. **Conversations (Oturumlar)**

   * **İstatistikler → Oturumlar → UDP** sekmesi.
   * En yoğun akışın **:4500 ↔ :4500** olduğuna bak (NAT-T göstergesi).
   * Satıra çift tıkla → ilgili akışa filtre uygulanır.

---

## 3)Evidence (Kanıtlar)




---

## Findings (Bulgular)

* **El sıkışma sırası doğrulandı:** Flow Graph’ta **IKE\_SA\_INIT (34) → IKE\_AUTH (35) → ESP** başladı.
* **NAT-T:** Trafik **UDP/4500** görüldü → **NAT-T aktif**.
* **Kripto teklifleri (INIT):** ENCR/INTEG/PRF/DH alanları **INIT**’te **şifresiz** göründü (ör. `AES-GCM-128 / – / PRF-HMAC-SHA2-256 / DH-14`)
* **Kimlik ve Trafik Seçiciler (AUTH):** **IDi/IDr** ile **TS\_i/TS\_r** alanları **Encrypted** (anahtar yoksa içeriği görünmez — beklenen).
* **ESP akışı:** **SPI (I→R) = 0x{{SPI\_IR}}**, **SPI (R→I) = 0x{{SPI\_RI}}**; anti-replay **sıra numarası artıyor**.
* **INFORMATIONAL (37)** paket(ler)i görüldü; **Flags** ve **Message ID** başlıkta, payload **Encrypted** (muhtemel DPD/DELETE/NOTIFY).

---

## Notes (Notlar)

* **Inner vs Outer:**

  * **Outer:** Public IP’ler + (NAT-T’de) **UDP/4500** + **ESP başlığı** (SPI/Seq **görünür**).
  * **Inner:** İç IP başlığı + üst katman veri **ESP şifreli yük** içinde (**görünmez**).
* **NAT / NAT-D / NAT-T farkı (tek satır):** NAT = çeviri; NAT-D = “NAT var mı?” bildirimi (INIT’te görülebilir); NAT-T = ESP’yi **UDP/4500** içine kapsülle.
* **Neden içeriği göremiyorum?** IKE\_AUTH ve ESP payload’ları **anahtarsız açılmaz** (ephemeral DH → Perfect Forward Secrecy).
* **Deşifre etmek istersen:**

  * **Statik ESP anahtarların** varsa → `Düzenle → Tercihler → Protokoller → ESP → Anahtarlar…`
  * **IKEv2 oturum anahtarı/log** (ör. strongSwan key-log) varsa → `Protokoller → IKEv2` altında ekle; Wireshark **CHILD\_SA** anahtarlarını türetip **ESP’yi açar**.
* **Vendor ID:** Kimlik ifşası değildir; en fazla **parmak izi**. Güvenlikte asıl kritik olan **zayıf mod/PSK** değilse endişe gerekmez.
* **Türkçe arayüz ipucu:** “Sütun” = **Column**, “Akış Grafiği” = **Flow Graph**, “Oturumlar” = **Conversations**.

---

## 7) DoD (Definition of Done)
- [ ] IKEv2: **IKE_SA_INIT** ve **IKE_AUTH** paketleri ekran görüntüsüyle kanıtlandı  
- [ ] ESP akışı (**proto 50** veya **UDP/4500 içinde**) ve **SPI’lar** gösterildi  
- [ ] **NAT-T durumu** (var/yok) belirtildi  
- [ ] **TS_i/TS_r** (Traffic Selectors) okundu ve rapora yazıldı  
- [ ] Gizlilik alanları **maskelendi**

     
---

# IKEv2: Baştan sona ne olur?

## 0) Oyuncular ve kavramlar

* **Initiator (başlatan)** ↔ **Responder (yanıtlayan)**
* **IKE SA**: IKEv2 kontrol kanalı için güvenli oturum (INIT/AUTH mesajları buradan gider).
* **CHILD SA**: Asıl veri şifrelemesini taşıyan oturum(lar) (tipik olarak **ESP**; iki yön için iki **SPI**).
* **NAT-T**: NAT varsa IKE/ESP, **UDP/4500** içine kapsüllenerek gider.
* **PFS**: Perfect Forward Secrecy—her oturum için taze (ephemeral) DH; yakalama dosyasıyla sonradan “kırmayı” engeller.

---

## 1) IKE\_SA\_INIT (Exchange Type = 34) — “kripto temeli atılır”

**Amaç:** Taraflar kripto algoritmalarını uzlaşır, DH ile ortak sır üretir, NAT var mı anlar.

### 1.1 Pakette neler var?

* **SA Teklifleri (Proposals/Transforms)**: Şifre (ENCR), bütünlük (INTEG), PRF, DH grubu (örn. **AES-GCM / PRF-SHA2-256 / DH-14**).
* **KE (Key Exchange)**: Ephemeral **Diffie-Hellman** kamusal değeri.
* **Nonce (Ni/Nr)**: Rastgelelik ve tazelik için.
* **NAT Detection (NAT-D)**: “Arada NAT var mı?” hash tabanlı bildirimler.
* (Opsiyonel) **Cookie**: DoS azaltmak için responder’ın istediği ekstra kanıt.

**Bu mesajlar ŞİFRESİZ gider.** Yani Wireshark’ta SA/KE/Nonce/NAT-D alanlarını net görürsün.

### 1.2 Kripto içerde nasıl işler? (özet formüller)

1. Taraflar DH ile ortak sırrı üretir: `g^ir`
2. **SKEYSEED** hesaplanır:
   `SKEYSEED = prf(Ni | Nr, g^ir)`
3. Buradan IKE SA için anahtarlar türetilir:
   `SK_d | SK_ai | SK_ar | SK_ei | SK_er | SK_pi | SK_pr = prf+(SKEYSEED, Ni | Nr | SPIi | SPIr)`

   * **SK\_e\***: IKE mesajlarını **şifreler**
   * **SK\_a\***: IKE mesajlarını **imzalar/bütünlük**
   * **SK\_p\***: **AUTH** hesaplarında kullanılır
   * **SK\_d**: **CHILD SA** anahtar türetimi için

> **Wireshark:** `ikev2 && (ikev2.exchange_type==34 or isakmp.exchange_type==34)`
> NAT-T kanıtı başlangıçta çıkmayabilir; ama birazdan UDP/4500’a geçişle netleşir.

---

## 2) IKE\_AUTH (Exchange Type = 35) — “kimlik ispatı ve ilk CHILD SA”

**Amaç:** Taraflar birbirini **kimlik doğrular**, **iç trafik** (TS) için yetki verir, **ilk CHILD SA**’yı kurar.

### 2.1 Pakette neler var?

Bu aşamadaki payload’lar **ŞİFRELİ**dir (INIT’te türetilen SK\_\* ile):

* **IDi / IDr**: Kimlik (IP, FQDN, sertifika DN vb.)
* **AUTH**: “El sıkışmayı ve kimliğimi biliyorum” ispatı (PSK / sertifika / EAP’e göre farklı hesaplanır ama **SK\_pi/SK\_pr** kullanır).
* **SA (CHILD)**: Veri kanalı için ESP/AH teklifleri (pratikte ESP).
* **TS\_i / TS\_r**: **Traffic Selectors**—tünelden hangi **iç IP/port/protokol** aralıkları geçecek.
* (Opsiyonel) **CFG/CP**: Remote-access istemciye **tünel IP’si**, DNS vb. atamak için.
* (Opsiyonel) **EAP**: Uzak erişimde ek kimlik doğrulama adımları.

**Sonuç:** İlk **CHILD SA** kurulur → veri düzlemi **ESP** ile başlar.

> **Wireshark:** `ikev2 && (ikev2.exchange_type==35 or isakmp.exchange_type==35)`
> İçerikler **Encrypted** görünür—bu normal. Anahtar (key-log/statik keys) vermezsen ayrıntısı açılmaz.

---

## 3) Veri Düzlemi: ESP (proto 50) — “artık trafik şifreli”

**Amaç:** Uygulama trafiği şifreli gider. İki yön için iki SPI vardır.

### 3.1 Başlık ve görünürler

* **ESP SPI** (Security Parameters Index) — görünür
* **Sequence Number** — görünür (anti-replay)
* **Encrypted Payload** — **görünmez** (iç IP başlığı + TCP/UDP veri burada şifreli)

### 3.2 NAT varsa: NAT-T

* IKE/ESP, **UDP/4500** içine konur → Wireshark “**ESP (UDP-encap)**” yazar.
* Paketin başında **Non-ESP Marker: 0x00000000** görülür.
* Başlangıçta 500/UDP hiç görülmeyebilir; bazı yığınlar doğrudan 4500’le başlar.

> **Wireshark:** `esp` veya `udp.port==4500 && esp`
> Yönleri ayır: `esp.spi == 0xAABBCCDD` (I→R), `esp.spi == 0xEEFF0011` (R→I)

---

## 4) Yaşam Döngüsü (IKEv2 bitti mi? Hayır—devam eder!)

### 4.1 INFORMATIONAL (Exch=37) — “idare işleri”

* **DPD/Liveness**: Karşı taraf canlı mı?
* **DELETE**: Bir **CHILD SA** veya **IKE SA** kapatılır.
* **NOTIFY**: Durum/uyarı iletileri. (Bu mesajlar çoğunlukla **Encrypted**’dır.)

### 4.2 CREATE\_CHILD\_SA (Exch=36)

* **CHILD SA rekey**: Veri anahtarlarını yenile.
* **Yeni CHILD SA**: Yeni TS/algoritmalarla ek tüneller.
* İsteğe bağlı yeni **DH** (PFS). Anahtarlar yine **SK\_d**’den türetilir.

### 4.3 IKE SA rekey (Exch=36)

* Tüm IKE SA tekrar kurulur (yeni SPIs). Eski IKE SA **DELETE** ile kapatılır.

### 4.4 MOBIKE (opsiyonel)

* IP/arayüz değişse de IKE SA ayakta kalır (ör. LTE→Wi-Fi geçişi).

### 4.5 Yeniden iletim / Parçalama

* **Retransmission**: Mesajlar kaybolursa aynı Message ID ile tekrar.
* **IKEv2 Fragmentation**: Büyük IKE mesajları parçalara bölünür (UDP’de MTU sorunlarına çare).

---

## 5) Güvenlik özellikleri – “neden sonradan açamıyorum?”

* **Ephemeral DH + PFS**: Oturum anahtarları uçlarda üretilir; pcap’ta **yoktur**.
* **AUTH**: Kimlik + tüm el sıkışma transkriptini kriptografik bağ ile iliştirir (MITM engeli).
* **Anti-replay**: ESP sequence ve pencere yönetimi.
* **NAT-T**: NAT arkasında çalıştırır, güvenlik modelini bozmaz.
* **Sertifika / PSK / EAP**: Farklı doğrulamalar; modern kurulumlarda **sertifika** veya güçlü PSK tercih edilir.

---

## 6) Wireshark’ta **nerede neyi** görürüm?

| Aşama     | Filtre                          | Görünenler                                                | Görünmeyenler                                 |
| --------- | ------------------------------- | --------------------------------------------------------- | --------------------------------------------- |
| INIT (34) | `ikev2 && exch==34`             | SA teklifleri, **KE**, **Nonce**, (varsa) **NAT-D**       | —                                             |
| AUTH (35) | `ikev2 && exch==35`             | **Encrypted** başlık; sadece Exchange/Flags/MsgID görünür | **IDi/IDr, AUTH, TS** (anahtar yoksa açılmaz) |
| ESP       | `esp` / `udp.port==4500 && esp` | **SPI**, **Seq**, (NAT-T’de Non-ESP Marker)**             | **Inner IP + üst katman veri**                |
| NAT-T     | `udp.port==4500 && esp`         | UDP/4500 kapsül, Non-ESP marker                           | —                                             |
| INFO (37) | `ikev2 && exch==37`             | Başlık, Flags; çoğunlukla **Encrypted** payload           | İçerik (DPD/DELETE)                           |

> **exch==XX** için alan adı sürüme göre `ikev2.exchange_type` veya `isakmp.exchange_type` olabilir.

---

## 7) En sık sorular (çok net cevaplar)

**Inner IP’ler neden görünmüyor?**
→ **ESP payload** içindeler ve **şifreli**. Anahtar yoksa açılmaz.

**PSK’yi bilirsem açar mıyım?**
→ IKEv2’de **ephemeral DH** var. Sadece PSK yetmez; oturum anahtarları için DH gizlileri gerekir.

**NAT mı NAT-D mi NAT-T mi?**

* **NAT**: Çevirenin kendisi.
* **NAT-D**: “NAT var mı?” algısı (INIT’te Notify).
* **NAT-T**: Çalışma yöntemi (ESP’yi **UDP/4500** içine koy).

**Exchange Type sayıları?**
`34=IKE_SA_INIT`, `35=IKE_AUTH`, `36=CREATE_CHILD_SA`, `37=INFORMATIONAL`.

**Flags (Bayraklar) kısa okuma?**
`0x08=Initiator`, `0x20=Response`, `0x00=Responder-Request`, `0x28=Initiator-Response`.

---

## 8) Kopyala-yapıştır mini akış (3 satırda “tüm IKEv2”)

1. **INIT (34):** ENCR/INTEG/PRF/DH pazarlık + **DH**, **Nonce**, (varsa) **NAT-D** → `SKEYSEED` ve **SK\_**\* anahtarları türetilir.
2. **AUTH (35):** **IDi/IDr**, **AUTH**, **TS\_i/TS\_r** (şifreli) → **ilk CHILD SA** kurulur.
3. **ESP:** Veri şifreli akar (NAT varsa **UDP/4500** + **Non-ESP Marker**). Devamında **CREATE\_CHILD\_SA (36)** ile rekey/ek tünel, **INFORMATIONAL (37)** ile DPD/DELETE.

---

## 9) “Gerçekte cihazda ne olur?” (paketin ötesi)

* Cihaz, seçilen algoritmalara göre **HW hızlandırma (AES-NI/crypto ASIC)** kullanır.
* **Anti-replay penceresi** ve **lifetime (zaman/byte)** sayaçları CHILD SA için izlenir; dolunca **rekey** tetiklenir.
* **Policy/ACL (proxy-id)** TS ile uyumlu değilse trafik düşer (drop/no matching SA).
* **Log** tarafında INIT/AUTH/DPD/Delete kayıtları vardır; sorun giderirken onlar hakemdir.

---

### Son söz

IKEv2’nin özü: **INIT** ile kripto temeli, **AUTH** ile kimlik + yetki + ilk veri tüneli, sonrası **ESP** ile şifreli trafik ve yaşam döngüsü yönetimi (**CREATE\_CHILD\_SA/INFORMATIONAL**). Wireshark’ta başlıklar ve SPI’lar görünür; **içerik şifreli** olduğu için normalde görünmez. Anahtar (veya cihazdan “decryption secrets”) olmadan pcap’tan “iç IP ve uygulama” verisi çıkarılamaz—tasarım gereği böyledir.

