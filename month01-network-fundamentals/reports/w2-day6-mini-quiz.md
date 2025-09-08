
---

# W2 — ARP & Ethernet Mini-Quiz (10 Soru)

**Skor:** 8 / 10  
**DoD**
- [x] 10 S/C işaretlendi
- [x] Yanlışlar için kısa açıklama yazıldı

---

## Soru & Cevaplar

1) **ARP’nin temel amacı nedir?**  
   **Benim:** B — IP → MAC • **Doğru:** B — IP → MAC ✅

2) **Off-subnet ping’de ilk ARP Who-has paketinin TPA alanı hangi IP olmalı?**  
   **Benim:** C — Default gateway IP’si • **Doğru:** C ✅

3) **`arp -d *` sonrası aynı subnet ping’de doğru olay sırası?**  
   **Benim:** B — ARP Who-has → Is-at → ICMP • **Doğru:** B ✅

4) **Klasik Ethernet (VLAN yok) FCS dahil min–maks boyut?**  
   **Benim:** B — 64–1518 B • **Doğru:** B ✅

5) **IPv4 için EtherType değeri?**  
   **Benim:** A — 0x0800 • **Doğru:** A ✅

6) **(Çoklu doğru)** Hub vs Switch & CSMA/CD ile ilgili doğru ifadeler?  
   **Benim:** A ve B • **Doğru:** A, B ve **C** ❌  
   ↳ Not: **Full-duplex’te CSMA/CD gerekmez** (C de doğru). “Switch ARP tutar” ifadesi yanlıştır.

7) **Duplex mismatch’te (100/full ↔ 100/half) full tarafta hangi sayaç(lar) artar?**  
   **Benim:** C — CRC / input errors • **Doğru:** C ✅

8) **L2 switch hangi bilgileri tutar/tutmaz?**  
   **Benim:** A • **Doğru:** **A ve D** ❌  
   ↳ Not: L2 switch **MAC/CAM** ve **STP** bilgisini tutar; **ARP** ve **routing** tablosunu tutmaz.

9) **802.1Q trunk etiketi nereye eklenir? Max çerçeve boyu ne olur?**  
   **Benim:** B — Src MAC’ten sonra/EtherType’tan önce 4 B; **1522 B** • **Doğru:** B ✅

10) **ARP ile ilgili doğru ifade?**  
    **Benim:** C — L2 broadcast, routable değil • **Doğru:** C ✅

---

## Yanlışlarım & Notlarım
- **S6:** C şıkkını atlamışım. **Full-duplex’te CSMA/CD yok** → C de doğru.  
- **S8:** STP bilgisini unuttum. L2 switch **MAC/CAM** + **STP** tutar; **ARP** ve **routing** tablosu tutmaz.

---

## Kısa Komut Hatırlatma
- MAC/CAM tablo: `show mac address-table dynamic`  
- VLAN listesi: `show vlan brief`  
- STP durumu: `show spanning-tree`

