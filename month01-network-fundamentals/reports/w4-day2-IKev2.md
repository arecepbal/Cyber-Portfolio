
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
