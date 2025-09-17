# W4 – Gün 1: WAN Türleri (MPLS, Leased Line) & Site-to-Site VPN (kavramsal)

> **Amaç (Goal)**  
> MPLS ve Leased Line gibi WAN seçeneklerinin ne sunduğunu, internet üzerinden **site-to-site VPN** mantığını ve VPN’de **tünelleme/kapsülleme, anahtar değişimi, kimlik doğrulama, bütünlük** kavramlarını kavramsal düzeyde anlamak.

---

## 1) Sözlük (6)

- **sanal özel ağ** — *VPN*, **noun**, /ˌviː.piːˈen/  
- **tünel** — *tunnel*, **noun**, /ˈtʌn.əl/  
- **kapsül(le)me** — *encapsulation*, **noun**, /ɪnˌkæp.sjʊˈleɪ.ʃən/  
- **anahtar değişimi** — *key exchange*, **noun**, /kiː ɪksˈtʃeɪndʒ/  
- **kimlik doğrulama** — *authentication*, **noun**, /ɔːˌθen.tɪˈkeɪ.ʃən/  
- **bütünlük** — *integrity*, **noun**, /ɪnˈteɡ.rə.ti/

---

## 2) WAN nedir? (kısa)

**WAN** (Wide Area Network), coğrafi olarak uzak ofis/şubeleri taşıyıcının (telekom) omurgası veya internet üzerinden birbirine bağlayan yapı. Temel seçenekler:

### 2.1 Leased Line (Kiralanmış Hat)
- **Nedir?** Taşıyıcıdan belirli iki nokta arasında **özel, paylaşımsız** devre (örn. EoMPLS, Metro Ethernet, SDH).  
- **Artıları:** Düşük gecikme/jitter, **SLA** güçlü, deterministik.  
- **Eksileri:** **Pahalı**, esneklik düşük; her yeni nokta için yeni devre/aylık.

### 2.2 MPLS L3VPN (Operatör Omurgasında Sanal Özel Ağ)
- **Nedir?** Operatörün MPLS omurgasında müşteri trafiği **etiketlenir**; müşteri siteleri **L3VPN** üzerinden “aynı özel ağda” gibi görünür.  
- **Artıları:** Çok noktayı ölçekli bağlama, **QoS**/SLA, taşıyıcı yönetimi.  
- **Eksileri:** İnternetten genelde ayrıdır (ek İnternet breakout gerekebilir), taşıyıcıya **vendor-lock** ve maliyet.

### 2.3 İnternet + Site-to-Site VPN
- **Nedir?** Her lokasyon internet erişimi alır; kenar cihazları arası **şifreli tünel** kurulur (çoğunlukla **IPsec**).  
- **Artıları:** **Ucuz**, hızlı devreye alma, esnek (çok sağlayıcı).  
- **Eksileri:** İnternet **en iyi çaba** (best-effort); **SLA/QoS** garanti değil (SD-WAN ile kısmen iyileşir).

> Not: Güncel mimarilerde **SD-WAN**, yukarıdaki taşıma katmanlarını (MPLS/internet/5G) **politikalarla** birleştirir; SASE/ZTNA kavramları kimlik-merkezli erişim uygular (ileri konu).

---

## 3) Site-to-Site VPN (IPsec) — mantık akışı

### 3.1 Tünel ve Kapsülleme
- **Tünel (tunnel):** İçteki özel paketler (ör. 10.0.0.0/8) **yeni bir dış IP başlığı** ile **kapsüllenerek** (encapsulation) internet üzerinden taşınır.  
- **Kapsülleme (encapsulation):** Asıl L3 paket + **ESP** başlığı/fragmanı + yeni dış IP başlığı.

### 3.2 IPsec Bileşenleri (özet)
- **IKEv2** (Internet Key Exchange v2): Anahtar değişimi / karşılıklı kimlik doğrulama.  
  - Aşamalar:  
    1) **IKE_SA** kur (kriptografik parametreler, Diffie-Hellman, kimlik doğrulama)  
    2) **CHILD_SA** kur (veri trafiği için SA: şifre/bütünlük anahtarları)  
- **ESP** (Encapsulating Security Payload):  
  - **Gizlilik (encryption)**: AES-GCM/CTR vb.  
  - **Bütünlük (integrity)** & **anti-replay**: paketler oynanmasın/tekrar kullanılmasın.  
- **Modlar:**  
  - **Tunnel mode** (site-to-site’de yaygın): İç paket tamamen kapsüllenir, **yeni dış IP** ile taşınır.  
  - **Transport mode** (uçtan uca/host-to-host): Yalnızca yük korunur, orijinal IP başlığı korunur.  
- **Kimlik doğrulama:** **PSK (ön paylaşımlı anahtar)** veya **sertifika** (PKI).  
- **NAT-T:** NAT arkasındayken IPsec akışını **UDP/4500** içine alır (ESP’yi geçirir).  
- **Rotalar:** Karşı tarafın “ilgi alanları” (interesting traffic) ACL/selector ile tanımlanır (örn. 192.168.10.0/24 ↔ 192.168.20.0/24).

### 3.3 Paket Yolculuğu (basitleştirilmiş)
1) **İç paket:** `SRC=10.1.1.10  DST=10.2.2.20`  
2) **Eşik/selector eşleşir:** “Bu trafik IPsec tüneline girecek.”  
3) **Kapsülleme:** ESP + **dış başlık** eklenir → `SRC=203.0.113.2  DST=198.51.100.5`  
4) İnternetten karşı uca gider, **ESP doğrulama/açma** yapılır → iç paket asıl hedefe iletilir.

---

## 4) WAN vs VPN — kısa tablo (DoD için)

| Özellik | **Leased Line** | **MPLS L3VPN** | **İnternet + Site-to-Site VPN (IPsec)** |
|---|---|---|---|
| Taşıma | Özel devre (point-to-point) | Operatör MPLS omurga | Genel internet |
| Güvenlik | Taşımada izolasyon (şifreleme opsiyonel) | Mantıksal izolasyon (VRF); şifreleme opsiyonel | **Uçtan uca şifreleme (ESP)** |
| SLA/QoS | **Yüksek** (deterministik) | **Yüksek** (QoS sınıfları) | **Best-effort** (SLA yok; SD-WAN ile iyileşir) |
| Gecikme/Jitter | **Düşük** | **Düşük/öngörülebilir** | Değişken (internet şartlarına bağlı) |
| Maliyet | **Yüksek** | **Orta–yüksek** | **Düşük** |
| Esneklik/Hız | Düşük | Orta (taşıyıcı bağımlı) | **Yüksek** (hızlı kurulum/çok ISS) |
| Ölçeklenebilirlik | Sınırlı (devre başına) | **İyi** (çok nokta) | **İyi** (ek tüneller/SD-WAN) |
| Kullanım örneği | Veri merkezi ↔ çekirdek | Çoklu şube, QoS kritik | Dağıtık ofisler, maliyet odaklı güvenli bağlantı |

---

## 5) SSS / İnce noktalar
- **“VPN zaten güvenli; MPLS’e gerek var mı?”**  
  Güvenlik = **şifreleme ve kimlik**, taşıma = **gecikme/SLA**. Çekirdek uygulamalarda SLA/QoS kritiktir; MPLS/LL bunu verir. VPN, internet üzerinde **gizlilik/bütünlük** sağlar ama gecikmeyi garanti etmez.  
- **“Tunnel vs transport?”**  
  Site-to-site’da **tunnel**; host-to-host/endpoint’te **transport** uygun.  
- **“PSK mi sertifika mı?”**  
  PSK hızlı; **sertifika** ölçekli/kurumsal ve güvenli yönetim için tercih.

---

## 6) Mini kontrol listesi (kendini yokla)
- MPLS ile Leased Line’ın farkını **1 cümle** ile söyleyebiliyor musun?  
- IPsec’te **IKE** ve **ESP**’nin rolünü ayırt edebiliyor musun?  
- **Tunnel** modun neden site-to-site için varsayılan olduğunu açıklayabiliyor musun?

---

## 7) Sonuç (Findings)
- WAN taşıma seçenekleri **SLA/QoS ve maliyet** ekseninde konumlanır.  
- Site-to-site IPsec VPN, **gizlilik+bütünlük+kimlik** sağlar; internetin değişken gecikmesini **çözmez**.  
- İş ihtiyacına göre **MPLS/LL** (deterministik) + **internet VPN** (maliyet/esneklik) hibritleri yaygındır.

---

## 8) DoD (Definition of Done)
- [ ] **WAN vs VPN** kısa tabloyu doldurdum/anladım.  
- [ ] **IPsec akışı** (IKEv2 + ESP, tunnel mode) kavramsal olarak net.  
- [ ] Sözlükteki 6 terimi doğru telaffuz ve anlamla eşleştirebiliyorum.

