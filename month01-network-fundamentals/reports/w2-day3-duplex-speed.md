# W2 — Duplex/Speed & Collision Domain (Hub vs Switch)

1) ## GOAL
- Hub vs Switch ile **collision domain** farkını kanıtlamak.
- **Duplex/speed mismatch** semptomlarını (jitter, goodput düşüşü, CRC/late collisions) göstermek.

## Setup
- Mini-Lab A: Hub + PC1 + PC2 (aynı subnet)
- Mini-Lab B: Switch(2960) + PC1 + PC2 (aynı subnet)
- IP Planı: 192.168.1.10/24 (PC1), 192.168.1.11/24 (PC2)

2) ## Steps
### A) Hub vs Switch
1) Hub senaryosu → iki yönde eşzamanlı ping (Simulation).
2) Event List’te **Collision** kaydı ss.
3) Aynısını Switch’le tekrarla → collision yok.

### B) Duplex/Speed Mismatch
1) **Normal (auto/auto)** ölçüm → `show interfaces fa0/1` ss.
2) PC0 NIC: **half/10**, Switch fa0/1: **full/100**.
3) PC1 e ping; tekrar `show interfaces fa0/1` ss.


4) ## Evidence
- `diagram/screen_w2_hub-collision.png`
- `diagram/screen_w2_switch-no-collision.png`
- `diagram/screen_w2_duplex-normal-show-int.png`
- `diagram/screen_w2_duplex-mismatch-show-int.png`
- (ops.) `diagram/screen_w2_duplex-sim.png`


5) ## Findings
**LABA:**
- HUB half/duplex olduğundan sistemde PC0 dan PC1 e ping gönderince collasions oluştu
- Switch full/duplex olduğundan collasions oluşmadı
- Mismatch: **CRC/late collisions/input errors** artışı, **jitter** ve **goodput** düşüşü (ss + kısa ölçüm notu)

**LABB:**
- PC nin internete bağlandığı portu da switchin o PC ye bağlandığı portuda AUTO-AUTO iken **link in**
- PC ninkini AUTO dan çıkartıp(FORCE ye geçip) FULL duplex yada 10 MB/s (yada iksii birden) yaptığımda **link down**(Normalde fastethernet olduğundan duplexdeğişiminde link down olmamalıyda ama PT problem yaptı)


6) ## Notes
**10/100BASE-T (FastEthernet):**
- Eğerki iki tarafın hızı farklıysa direk link down
- Mismatch’te bir uç **half-duplex**, diğeri **full-duplex** çalıştığı için half tarafı **CSMA/CD** beklerken diğer uç aynı anda gönderim yapar;bu **gecikme dalgalanması (jitter)**, **yeniden iletimler** ve **CRC/late collision** sayaç artışlarına yol açar.(late collisions / collisions (half tarafta),CRC / input errors (full tarafta))
- İki taraf AUTO ise taraflar birbirine **FLP (Fast Link Pulse)**gönderir, birbirlerinin hız ve duplexlerini öğrenirlerfakat taraflardan biri FORCE ye geçerse AUTO taraf direk 100/Half'e düşer.





## DoD (Definition of Done)
- [ ] **Normal vs mismatch** arayüz çıktıları ss (show int fa0/1)
- [ ] Hub’da **collision**, switch’te **no collision** kanıtı
- [ ] Mismatch için **fark açıklaması** (4–6 cümle)
