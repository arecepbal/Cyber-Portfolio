## Ay 1 – Ağ Temelleri (ICMP/ARP/DNS)
**Identity/IAM (4):**
- SSO ve MFA temelleri
- AuthN vs AuthZ farkı
- Session/refresh token mantığı
- Conditional Access nedir?

**Cloud Defense (4):**
- Azure Diagnostic Settings ile **Activity/Sign-in/Audit** loglarını **Log Analytics + Storage**’a akıt; saklama ≥ 90 gün.
- S3 için **Block Public Access** hesap/bucket düzeyinde **ON**.
- AWS **Security Group**: 0.0.0.0/0 için **RDP/SMB/DB portları** yasak (SCP/Config Rule).
- EC2 için **IMDSv2 zorunlu**; IMDSv1 isteklerini **blokla**.

**Detection Rules (8):**
- ICMP fan-out eşiği (60 sn'de 20+ hedef)
- DNS NXDOMAIN spike (60 sn'de 15+)
- SYN:ACK oranı > 10 (olası SYN taraması)
- ARP rate spike (10 sn'de 30+ request)
- ICMP TTL anomalisi (beklenenden sapma)
- DNS TXT/NULL sıra dışı kullanımı
- ICMP içinde büyük payload (exfil denemesi)
- Aynı kaynak IP'den hızlı port dizisi (1-1024)

**Hunting (4):**
- MITM öncesi ARP spike var mı?
- ICMP sweep sonrası erişilen hostlarda anomali var mı?
- DNS normal → NXDOMAIN fırlaması geçişi var mı?
- Aynı kaynak IP'nin TTL dağılımı beklenenden farklı mı?

**DFIR (1):**
- pcap tabanlı mini timeline: ICMP/ARP/DNS normal→anomali

---

## Ay 2 – Routing/VLAN + Otomasyon
**Identity/IAM (4):**
- Legacy auth nedir ve neden tehlikeli?
- IAM'de role vs group
- Service account/principal kavramı
- Audit vs Sign-in log farkı

**Cloud Defense (4):**
- AWS **CloudTrail** tüm bölgelerde **kapatılamaz** şekilde etkin; **log file validation** açık, S3 **MFA Delete** aktif.
- Azure Storage **AllowBlobPublicAccess = false** (hesap düzeyi).
- Azure **NSG**: **RDP/SMB** inbound **deny**; erişim sadece **Bastion/VPN** ile.
- Azure VM’lerde **public IP** varsayılanı **deny**; erişim Bastion/VPN ile.

**Detection Rules (8):**
- Vertical scan paterni (tek hedefte 8+ port/protokol)
- Rogue DHCP offer tespiti
- Aynı kaynak 2 VLAN'a kısa sürede konuştu
- NAT arkasından sıra dışı outbound fan-out
- TCP RST fırtınası eşiği
- SMB enumeration (null session denemesi)
- mDNS/LLMNR zehirleme belirtisi
- ARP cache inconsistency (aynı IP → farklı MAC)

**Hunting (4):**
- Rogue DHCP lease anomalisi var mı?
- Aynı kaynak kısa sürede 2 VLAN'a konuşmuş mu?
- SYN scan ile başarısız login korelasyonu var mı?
- mDNS/LLMNR kötüye kullanım işareti var mı?

**DFIR (1):**
- NAT/VLAN topolojide tarama olayı timeline'ı

---

## Ay 3 – Windows Güv. + Sysmon
**Identity/IAM (4):**
- Windows oturum ve kimlik artefact'ları
- Local vs domain account farkı
- LAPS/FIDO fikirleri
- Kerberos özet bakış

**Cloud Defense (4):**
- **AWS Config** tüm kaynak türlerinde kayıt; uyumsuzluklar için **SNS/Alarm** üret.
- S3/Blob **default encryption = KMS**; şifrelenmemiş yüklemeleri **deny**.
- ALB/AppGW arkasındaki uygulamalarda **HTTPS zorunlu**; **HSTS** başlıkları kontrol.
- Diskler: **EBS/Azure Managed Disks**’te **default encryption** ve CMK kullan.
- 
**Detection Rules (8):**
- Sysmon: LSASS handle açma (T1003)
- Sysmon: Encoded PowerShell komutu (T1059.001)
- WMI process spawn zinciri (T1047)
- schtasks ile kalıcılık (T1053.005)
- RAR/7z ile ani arşivleme + dışa gönderim ipucu
- Office → cmd → powershell parent-child
- Reg add autorun kalıcılık (T1547)
- rundll32 ile şüpheli DLL çağrısı

**Hunting (4):**
- Office→cmd→powershell zinciri görüldü mü?
- Encoded PowerShell kullanan hesaplarda riskli oturum var mı?
- LSASS erişimi ile aynı saatlerde oturum değişikliği var mı?
- WMI event tüketicisi olan hostlarda outbound anomali var mı?

**DFIR (1):**
- Windows triage: Sysmon + Security.evtx 6 saatlik timeline

---

## Ay 4 – Linux Güv. + Filebeat
**Identity/IAM (4):**
- Linux PAM temelleri
- SSH key yönetimi iyi pratikler
- Sudo politikaları
- Auditd kimlik alanları

**Cloud Defense (4):**
- Azure **Defender for Cloud** etkin; **auto-provisioning** (agent/plan) açık ve güvenlik önerileri takipte.
- Kritik bucketlarda **versioning + object lock** (uygunsa) etkin.
- AWS **Network ACL**: yüksek riskli portlara stateful/ingress kısıtı.
- Patch yönetimi: **SSM/Azure Update** baseline; kritik yamalar **otomatik**.
  
**Detection Rules (8):**
- sudo abuse: ardışık hatalı deneme + shell açılışı
- Reverse shell paterni (bash -i >& /dev/tcp/...)
- cron/anacron yeni job + beklenmeyen kullanıcı
- ssh key authorized_keys'e sessiz ekleme
- Nmap/Netcat binary yürütüm anomali
- SUID binary ile hak yükseltme denemesi
- Auditd: /etc/shadow erişim denemesi
- Kötü amaçlı curl|bash paternleri

**Hunting (4):**
- sudo'yu sıra dışı kullanan kullanıcılar kim?
- Reverse shell paterni sonrası outbound DNS artışı var mı?
- cron job ekleyen kullanıcı son hafta neler yaptı?
- SUID binary erişimi sonrası privilege değişimi oldu mu?

**DFIR (1):**
- Linux triage: auth.log + bash history micro-timeline

---

## Ay 5 – Identity-First (Entra/Okta)
**Identity/IAM (4):**
- Entra sign-in log alanları
- Conditional Access koşulları
- MFA push fatigue riskleri
- Risky user/risky sign-in

  **Cloud Defense (4):**
- VPC/NSG **Flow Logs** → merkezi havuz; yüksek hacim için **dedicated workspace**.
- Azure **Key Vault**: **soft delete + purge protection** + **firewall** + **private endpoint**.
- VPC **Endpoint**’leri ile S3/STS gibi servislere **özel erişim**; public egress kapat.
- User-data/Custom script’te **secret** bulunmamalı; tarama ile engelle.

**Detection Rules (8):**
- Impossible travel (Entra/Okta)
- Legacy auth token kullanımı engel denemesi
- Failed-MFA burst (kısa sürede 5+)
- Consent phishing / risky OAuth grant
- Atypical location sign-in (kurumsal saat dışı)
- Multiple tenant resource access denemesi
- Disabled account ile auth denemesi
- Refresh token reuse/abuse paterni

**Hunting (4):**
- Impossible travel tetiklenen hesaplar hangi uygulamalara gidiyor?
- Failed-MFA burst sonrası başarılı login oldu mu?
- Legacy auth kullanan IP'ler kurumsal mı?
- Risky OAuth consent'ten sonra data erişimi arttı mı?

**DFIR (1):**
- Kimlik olayı: impossible travel → containment notları

---

## Ay 6 – Cloud Defense (Azure/AWS)
**Identity/IAM (4):**
- AWS IAM policy JSON
- Azure RBAC rolleri
- Access Analyzer/Permissions
- Key/secret rotasyonu

  **Cloud Defense (4):**
- S3 **Server Access Logging** kritik bucket’larda açık; ayrı log bucket’ına yaz.
- S3 **bucket policy**: yalnız **TLS (aws:SecureTransport)** ve belirli **VPC Endpoint**’lerden erişim.
- Azure **Private Endpoint**: Storage/Key Vault için zorunlu; public erişim kapalı.
- Auto-scaling launch template’lerde **security hardening** ön-ayarları.

**Detection Rules (8):**
- AWS CloudTrail: public S3 açılımı
- Azure Blob public erişim değişikliği
- IAM policy wildcard ('*') tespiti
- IMDS (169.254.169.254) erişim paterni
- GuardDuty: Credential exfil/Anomalous API
- Çok kısa sürede çok bölge API çağrıları
- AccessKey rotasyonu gecikmesi (age > N gün)
- Security group inbound genişleme (0.0.0.0/0)

**Hunting (4):**
- Public S3/Blob açıldıktan sonra erişim modeli nasıl değişti?
- IAM policy wildcard kullanan rolleri kim assume etti?
- IMDS erişimi geçen hafta nasıldı, outlier var mı?
- Bölge dışı API çağrısı yapan anahtar/hizmet hangisi?

**DFIR (1):**
- Cloud: public bucket açılımı → erişim inceleme timeline

---

## Ay 7 – IDS/IPS + Korelasyon
**Identity/IAM (4):**
- SAML assertion yapısı
- OIDC id_token claim'leri
- SSO federation güvenliği
- App registration izinleri

**Cloud Defense (4):**
- Azure AD **Sign-in** ve **Audit** logları **Sentinel**’e akıyor; **düşük gecikme** hedefi.
- Azure Storage **Secure transfer required**; **SAS** ömürleri kısa (≤ 24s).
- Egress **deny-by-default** yaklaşımı; yalnız onaylı hedeflere çıkış.
- AWS **SSM Session Manager** ile **SSH/RDP kapalı** yönetim (audit’li).
  
**Detection Rules (8):**
- Suricata: C2 beacon interval sabitliği
- Zeek: DNS tünel şüphesi (uzun/liste dışı FQDN)
- Network + Endpoint: beacon + process korelasyonu
- HTTP UA anomali (tool UA)
- DoH uç noktalarına anormal sıklık
- TLS JA3 hash sapmaları
- SMB brute force + başarısız auth korelasyonu
- Egress port anomali (beklenmeyen 8088/8443)

**Hunting (4):**
- Beacon interval sabit hostlar hangi süreçleri çalıştırıyor?
- DNS tünel şüphesi olan sorgular hangi kullanıcıdan?
- JA3 outlier'ları kimlik olaylarıyla örtüşüyor mu?
- DoH uç noktalarına giden trafiği hangi process başlatıyor?

**DFIR (1):**
- Beacon şüphesi: network+endpoint korelasyon timeline

---

## Ay 8 – Network Attacks → Playbook
**Identity/IAM (4):**
- Guest/B2B hesap politikaları
- Break-glass hesaplar
- Just-In-Time erişim
- Consent yönetimi

  **Cloud Defense (4):**
- AWS **GuardDuty** tüm bölgelerde etkin; **delegated admin** ile merkezi yönetim.
- S3 **Public ACL**’leri tamamen reddet; yalnız policy tabanlı izin.
- DoH/DoT outbound **allowlist**; bilinmeyen resolver’lara blok.
- Azure **Disk Encryption Set** + **Key Vault** entegrasyonu.

**Detection Rules (8):**
- ARP spoof/MITM imza seti
- DNS poisoning belirtisi
- Gateway MAC değişim uyarısı
- SSLstrip benzeri pattern
- Rogue AP tespiti (MAC/OUI farkları)
- Proxy ARP yoğunluğu
- IPv6 RA spoof (SLAAC anomali)
- DHCP starvation paterni

**Hunting (4):**
- Gateway MAC değişiminden hemen önce hangi olaylar oldu?
- Rogue AP şüphesi olan BSSID çevresinde hangi hesaplar oturum açtı?
- SSLstrip paterninde hangi domainler hedef?
- DHCP starvation ile eş zamanlı login anomalisi var mı?

**DFIR (1):**
- MITM olayı: ARP/Gateway değişimi → etki analizi

---

## Ay 9 – Web/API + OIDC/SAML (PURPLE)
**Identity/IAM (4):**
- OIDC akışları (Auth code PKCE)
- SAML replay/relaystate riskleri
- JWT imza doğrulama
- WAF + identity entegrasyonu

**Cloud Defense (4):**
- Azure **Activity Log Alerts**: policy disable/role change için anlık uyarı.
- Azure **Defender for Storage** etkin; **malware scanning** (varsa) kullan.
- Proxy/egress üzerinden **TLS inspection** politikaları (hukuki uygunlukla).
- Golden image (AMI/SIG) hattı; drift’i policy ile engelle.
  
**Detection Rules (8):**
- OIDC nonce/replay şüphesi
- Session fixation belirti paternleri
- Refresh token reuse anomali
- WAF: param brute force + 401/403 korelasyonu
- GraphQL introspection taraması
- API rate limit ihlalleri
- JWT alg none / zayıf imza denemesi
- SAML audience mismatch işareti

**Hunting (4):**
- JWT reuse görülen hesaplarda IP/cihaz değişimi var mı?
- WAF 401/403 yoğun IP'lerde login denemeleri var mı?
- SAML audience hatasında uygulama kim?
- GraphQL introspection istekleri hangi endpointlerde?

**DFIR (1):**
- Web kimlik olayı: session fixation şüphesi timeline

---

## Ay 10 – SIEM Use-Cases v1
**Identity/IAM (4):**
- Conditional Access 'device compliance'
- Privileged Identity Mgmt (PIM)
- Session management gelişmiş
- Token lifetime ayarları

  **Cloud Defense (4):**
- AWS **CloudTrail tamper** alarmı: trail kapatma/değişiklikte **CloudWatch Alarm**.
- S3 **lifecycle** ile eski sürümleri arşivle/sil; veri sınıflamasına göre kural.
- VPN/DirectConnect/ExpressRoute rotalarında **prefix filter** ve **BGP guard**.
- EBS/Managed Disk **snapshot**’ları **private** ve **KMS** şifreli.

**Detection Rules (8):**
- Use-case#1: Initial access → Execution zinciri
- Use-case#1: Exfiltration alarmı
- Use-case#2: Valid accounts lateral move
- Use-case#2: Priv-esc denemesi korelasyonu
- Use-case#3: Beacon → C2 → staging
- Use-case#3: Data staging volume spike
- Global FP izleme kuralı (ratio)
- Alert deduplication kuralı

**Hunting (4):**
- Use-case#1 zincirinde atlanan bir adım var mı?
- Use-case#2 lateral hareket eden hesaplar kim?
- Use-case#3 C2 kanalında data staging kanıtı var mı?
- FP üreten kurallarda ortak desen ne?

**DFIR (1):**
- Use-case#1 tam akış: initial→exfil timeline

---

## Ay 11 – DFIR Mini-Vaka
**Identity/IAM (4):**
- Incident'da hesap izolasyonu
- Password reset politikaları
- SSPR güvenliği
- High-risk user akışı

  **Cloud Defense (4):**
- Azure **Secure Score** ve **recommendations**’ı haftalık gözden geçir; en az %70 hedef.
- Azure **Information Protection/Labels** ile hassas veri etiketlemesi.
- DDoS Protection (AWS Shield Advanced / Azure DDoS) kritik uçlara bağlı.
- EC2/VM **termination protection** kritiklerde açık.

**Detection Rules (8):**
- Case TTP: T1566 Phishing artefact'ları
- Case TTP: T1059 PowerShell komut zinciri
- Case TTP: T1003 credential dump izleri
- Case TTP: T1071.001 (HTTPS C2)
- Containment sonrası persistence araması
- Suspect account disable sonrası deneme
- Exfil channel kapatma doğrulama
- Detection gap sonrası kural ekleme alarmı

**Hunting (4):**
- Phish tıklayan kullanıcıların process zinciri ne?
- Credential dump izleriyle örtüşen kimlik olayları var mı?
- HTTPS C2 şüphesi ile DNS outlier örtüşüyor mu?
- Containment sonrası persistence kaldı mı?

**DFIR (1):**
- Phishing olayı: e-posta→host→exfil zinciri timeline

---

## Ay 12 – Konsolidasyon + Demo
**Identity/IAM (4):**
- IAM denetim checklist
- Access review süreçleri
- Role mining temel
- Break-glass tatbikatı

  **Cloud Defense (4):**
- AWS **Security Hub** CIS/Owasp kontrolleri; **finding**’ler triage akışına bağlı.
- S3 **Bucket Key** kullanarak KMS maliyet ve performans optimizasyonu.
- WAF rate-limit ve bot koruması **baz** kural seti etkin.
- Cloud init/Extensions logları merkezi toplama.

**Detection Rules (8):**
- Top false-positive kaynağına özel filtre
- High-noise kuralı iki aşamalı hale getirme
- Watchlist ile istisna yönetimi
- Threshold adaptasyonu (İş saatleri/hafta sonu)
- Alert enrichment (asset criticality)
- Owner notification flow kuralı
- Dead rule (hiç çalmayan) temizliği
- Rule unit-test (sentetik olay enjeksiyonu)

**Hunting (4):**
- En çok noise üreten kural hangi varlıkta?
- İş saatleri vs dışı alarmlarda fark nedir?
- İstisna listeleri kötüye kullanılıyor mu?
- Hiç çalmayan 'ölü' kural kaldı mı?

**DFIR (1):**
- FP azaltma vaka notu: kural tuning ön/sonrası

---

## Ay 13 – Use-Cases v2 + Coverage
**Identity/IAM (4):**
- OAuth consent review
- Service principal hygiene
- SCIM provizyonlama
- Guest access governance

  **Cloud Defense (4):**
- AWS **Organizations SCP**: **CloudTrail/GuardDuty kapatma** engelini uygula.
- WAF’ta **OWASP top10** kural seti + **geo/IP reputation** filtreleri.
- **Secrets Manager/Key Vault** zorunlu; kod/CI’da plain secret yasak.
- **Tag-based quarantine** (EC2/VM) otomasyonu; izolasyon SG/NSG.


**Detection Rules (8):**
- Use-case#4: OAuth risky grants
- Use-case#5: Shared mailbox kötüye kullanım
- Use-case#6: Service principal misuse
- Geolokasyon outlier kuralı
- Impossible travel v2 (device posture ile)
- S3 exfil via AccessKey pattern
- Azure RunCommand abuse paterni
- Kubernetes API istek anomali tespiti

**Hunting (4):**
- Risky grants alan app'ler hangi dataya erişiyor?
- Shared mailbox hareketleri anormal mi?
- Service principal hareketleri saat dışı mı?
- RunCommand kullanan VM'ler nereden tetikleniyor?

**DFIR (1):**
- OAuth risky grant inceleme timeline

---

## Ay 14 – Raporlama & Etik (Red+Blue)
**Identity/IAM (4):**
- Raporlamada kimlik metrikleri
- Least-privilege denetimi
- Exception/ist. yönetimi
- Access recertification

**Cloud Defense (4):**
- Azure **Policy**: **NSG açık port** deny; **public IP** deny (etiket bazlı istisna).
- API Gateway/AppGW’de **mTLS/Client cert** kritik entegrasyonlarda.
- **Key rotation** politikası: 90/180 gün; otom. uyarı.
- CloudTrail/Sentinel **tamper** algısında playbook tetikleme.

**Detection Rules (8):**
- Rapor: Kural kapsama kontrol alarmı
- Rapor: KPI (MTTD/MTTR) veri toplama kuralı
- Rapor: ATT&CK heatmap güncelleme alarmları
- Rapor: Post-incident follow-up ping
- Ethics: pentest sınır ihlali uyarısı
- Red→Blue bulgu eşleme kuralı
- Evidence retention süresi alarmı
- Data privacy ihlali tespiti (PII query)

**Hunting (4):**
- ATT&CK heatmap'te boş kalan tekniklerin verisi var mı?
- Raporladığın bulgularda tekrar eden kök neden hangisi?
- Privacy ihlali şüphesi olan sorgular kimden?
- Post-incident aksiyonları kapandı mı?

**DFIR (1):**
- Post-incident kapatma kontrolü timeline

---

## Ay 15 – Reverse Engineering
**Identity/IAM (4):**
- Malware vakasında kimlik izleri
- MFA bypass desenleri
- Device trust entegrasyonu
- Token theft göstergeleri

  **Cloud Defense (4):**
- AWS **Config Conformance Packs**: CIS benchmark setleri uygulandı.
- Bot koruması + **rate limiting** üretimde aktif.
- Key Vault **purge protection** ve **RBAC** etkin; access-policy yerine RBAC.
- **KMS key compromise** acil prosedürü ve runbook.

**Detection Rules (8):**
- Malware API çağrısı imzası (Sysmon)
- Packed binary sezgisi (entropy/size)
- Unsigned DLL yükleme paterni
- Suspicious code injection (T1055)
- LOLBin yürütme anomalisi
- New autorun registry + hash mismatch
- WMI event consumer kalıcılığı
- Process hollowing belirtisi

**Hunting (4):**
- Şüpheli binary'nin benzerleri ağda mevcut mu?
- Injection paterni görülen hostlarda kimlik olayları ne?
- Unsigned DLL yükleyen süreçler hangileri?
- Autorun eklenen makinelerde outbound anomali var mı?

**DFIR (1):**
- Şüpheli binary analizi → IOC → ortam izleri timeline

---

## Ay 16 – Malware Analysis
**Identity/IAM (4):**
- Credential theft zinciri
- Password spray tespiti
- Modern Auth zorlaması
- App secret/sertifika güvenliği

  **Cloud Defense (4):**
- Azure **Blueprint/Deployment Stacks** ile zorunlu guardrail setleri.
- WAF **bypass** denemeleri için özel imzalar.
- **HSM/CMK** kritik anahtarlar için zorunlu.
- S3 public açılımı tespitinde **auto-remediate** denemesi.

**Detection Rules (8):**
- Sandbox'tan IOC → domain/IP/JA3 tespiti
- File hash sighting + quarantine talebi
- Malware mutex/pipe adı tespiti
- Suspicious scheduled task by non-admin
- Dropper → second-stage korelasyonu
- Masquerading (svch0st, explore.exe)
- Exfil over DNS paternleri
- RDP brute + clipboard exfil işareti

**Hunting (4):**
- Sandbox IOC'leri ağda/hostlarda tekrar görüldü mü?
- Hash sighting sonrası izolasyon oldu mu?
- Dropper → second-stage hatları nerelerde?
- Masquerading yapan süreçler kim?

**DFIR (1):**
- Sandbox bulgusu → host izleri → containment timeline

---

## Ay 17 – Forensics (tam vaka)
**Identity/IAM (4):**
- Forensic'te account timeline
- SSO artefact toplama
- Privileged access olayları
- Audit vs Sign-in korelasyon

  **Cloud Defense (4):**
- Exception/waiver’lar **süreli** ve gerekçeli; otomatik hatırlatma.
- GraphQL **introspection** kapalı (prod); development’ta kısıtlı.
- Secret varlık erişim logları (Get/List) için alarm.
- **Tag-based quarantine** (EC2/VM) otomasyonu; izolasyon SG/NSG.


**Detection Rules (8):**
- USB mount + büyük dosya kopya paterni
- LNK/shortcut spray tespiti
- Browser credential store erişimi
- Rclone/megacmd kullanım imzası
- Randomized subdomain burst
- Volatility artefact alert (swap/susp proc)
- Persistence remnants scan alert
- Shadow copy silme denemesi (T1490)

**Hunting (4):**
- USB ekli makinelerde data spike var mı?
- LNK spray sonrası hangi kullanıcılar riskli?
- Browser cred store erişimi kimde yoğun?
- Shadow copy silinen hostlarda hangi olaylar var?

**DFIR (1):**
- Disk+RAM imajından credential theft timeline

---

## Ay 18 – SIEM Performans (MTTD/MTTR)
**Identity/IAM (4):**
- Kimlik KPI'ları
- Risk-based access
- Geo/device policy
- OAuth app allowlist

  **Cloud Defense (4):**
- Multi-account **landing zone** (Control Tower / CAF) temel kuruldu.
- Header/redirect hijack korumaları ve güvenli header seti.
- Credential **exposure** skanları (git-secrets/trufflehog).
- CloudTrail/Sentinel **tamper** algısında playbook tetikleme.

**Detection Rules (8):**
- MTTD ölçüm kuralı (first_seen → alert_time)
- MTTR ölçüm kuralı (alert → close)
- Queue backlog spike alarmı
- Tiering/priority drift alarmı
- Mute edilen kural sayısı artışı alarmı
- Case reopen oranı alarmı
- On-call SLO ihlal alarmı
- FP oranı > %5 olan kurallara bayrak

**Hunting (4):**
- MTTD yüksek kalan senaryolar hangileri?
- MTTR uzatan adım neresi?
- Backlog spike saatleri hangi patternle örtüşüyor?
- SLO ihlali yapan hafta için ortak neden ne?

**DFIR (1):**
- SLA ihlali vakası: MTTD/MTTR kök neden analizi

---

## Ay 19 – Alert Clustering (AI)
**Identity/IAM (4):**
- Identity sinyaliyle alert clustering
- Account risk score
- Entitlement graf analizi
- Outlier kullanıcı davranışı

  **Cloud Defense (4):**
- Container image **scanning** (ECR/ACR) ve **CI gate** (High CVE fail).
- **semgrep** SAST gate + **trivy** image gate + **tfsec/Checkov** IaC gate.
- DLP/etiketli veri için **egress deny** (Storage/Key Vault policy).
- Detection gap: yeni kural eklenene kadar **kompansasyon guardrail**.

**Detection Rules (8):**
- Alert similarity eşiği (text + entity overlap)
- Cluster başına temsil olay alarmı
- Repeat offender hesap/host alarmı
- Alert correlation candidate işaretleyici
- Outlier cluster alarmı
- New pattern discovery etiketi
- Benzer vaka linkleme öneri sinyali
- Noise source auto-snooze önerisi

**Hunting (4):**
- En büyük alert kümesindeki ortak öznitelik ne?
- Tekrarlayan fail IP/hesap/host kim?
- Outlier cluster hangi kurallardan geliyor?
- Linklenen vakalarda ortak TTP var mı?

**DFIR (1):**
- Alert cluster inceleme timeline

---

## Ay 20 – ML Baseline (Precision/Recall)
**Identity/IAM (4):**
- Model feature'larında kimlik alanları
- Labeling için kimlik bağlamı
- Identity drift takibi
- High-risk session feature

**Cloud Defense (4):**
- K8s **Pod Security Standards** (baseline/restricted) etkin.
- CI/CD runner’larda **OIDC federation**; statik secret yok.
- Kayıt dışı bölgelere veri kopyasında alarm.
- High-noise kaynaktan gelen veriye **pre-filter** katmanı.

**Detection Rules (8):**
- Precision/Recall raporlama kuralı
- Label eksikliği alarmı
- Drift (veri dağılımı) alarmı
- Feature değer aralığı ihlali
- Threshold auto-tune önerisi
- High confidence false alarms listesi
- Model inference başarısızlığı alarmı
- Model versiyon değişimi uyarısı

**Hunting (4):**
- High-confidence FP neden üretiliyor?
- Label oranı düşen kurallar hangileri?
- Drift en çok hangi özellikte?
- Auto-tune sonrası FP azaldı mı?

**DFIR (1):**
- Model drift vakası timeline

---

## Ay 21 – DL Anomaly (Autoencoder)
**Identity/IAM (4):**
- Anomaly modelinde kimlik kullanımı
- Long-lived session kontrolü
- Impossible travel v2
- Risky grant tespiti

**Cloud Defense (4):**
- K8s **NetworkPolicy default deny**; sadece gerekli egress/ingress.
- Protected branch + **mandatory review**; **CODEOWNERS**.
- S3/Blob’a **server-side encryption** olmadan yükleme **deny**.
- Alert **dedup** ve **suppress** politikaları dökümante.

**Detection Rules (8):**
- Anomaly score > eşik alarmı
- Sequence length outlier uyarısı
- Beacon periodicity + low-variance skor
- Long-lived session outlier
- Rare command sequence işareti
- New binary in sensitive path uyarısı
- Unusual parent-child pair alarmı
- Weekend/after-hours rare activity

**Hunting (4):**
- Anomaly score yüksek olaylar hangi varlıklarda?
- Rare sequence hangi kullanıcıdan?
- Long-lived session kimde?
- Weekend outlier hangi uygulamada?

**DFIR (1):**
- Anomaly spike timeline (hafta sonu)

---

## Ay 22 – AI-driven IR (Triage API)
**Identity/IAM (4):**
- IR otomasyonunda kimlik aksiyonu
- Disable/force reset akışları
- Containment playbook'ları
- Ticketing entegrasyonu
  
**Cloud Defense (4):**
- OPA **Gatekeeper/Kyverno** ile **privileged: false**, hostPath deny.
- SBOM üretimi (Syft) + **grype** ile sürekli izleme.
- PII erişim sorgularında **Justification** alanı zorunlu.
- Kritik kural için **two-person change** onayı.

**Detection Rules (8):**
- IR otomasyon hatası alarmı
- Playbook başarısız adım uyarısı
- Auto-triage düşük güven öneri filtresi
- Tagging consistency kontrol alarmı
- Rate-limit ihlali (API)
- IR API latency spike alarmı
- Case assignment anomaly
- Escalation loop tespiti

**Hunting (4):**
- IR playbook'ta en çok hata veren adım?
- Latency spike hangi saatlerde?
- Escalation loop hangi ekipler arasında?
- Otomasyon devre dışı kalınca MTTD/MTTR ne oldu?

**DFIR (1):**
- IR otomasyon hatası → manuel fallback timeline

---

## Ay 23 – Exam Readiness
**Identity/IAM (4):**
- Sertifika eksikleri: kimlik alanı
- Deneme sınavları kimlik konuları
- Kapsam boşlukları
- Cheatsheet güncelle

  **Cloud Defense (4):**
- K8s secret’lar **KMS/Key Vault** ile şifreli; `Secret` yerine **external secret**.
- Sigstore **cosign** ile imzalı imaj; imzasız imaj deny.
- DLP/etiketli veri için **egress deny** (Storage/Key Vault policy).
- Detection gap: yeni kural eklenene kadar **kompansasyon guardrail**.

**Detection Rules (8):**
- Sınav hazırlık eksik alan alarmı
- Coverage gap 'todo' alarmı
- Dead link/artefact kontrolü
- Lab credential sızıntı kontrolü
- Exam timing practice uyarısı
- Cheatsheet güncelleme hatırlatıcı
- Practice box rotation alarmı
- Weak area repeat practice flag

**Hunting (4):**
- Zayıf alanlarda hangi teknikler eksik?
- Coverage gap kapanmış mı?
- Practice tekrarları etkili oldu mu?
- Cheatsheet pratikte işe yarıyor mu?

**DFIR (1):**
- Sınav/kapama sprinti timeline (zayıf alan)

---

## Ay 24 – Capstone (Adversary Emulation)
**Identity/IAM (4):**
- Emulation için kimlik TTP'leri
- PurpleSharp ile kimlik
- Fix doğrulama
- Post-incident kimlik değişiklikleri

**Cloud Defense (4):**
- Auditing etkin; **kube-apiserver** logları SIEM’e.
- Dependabot/Renovate ile bağımlılık güncelleme guardrail’i.
- Kayıt dışı bölgelere veri kopyasında alarm.
- High-noise kaynaktan gelen veriye **pre-filter** katmanı.

**Detection Rules (8):**
- Adversary TTP run → detect doğrulama alarmı
- E2E zincir geçti/kalem notu alarmı
- Emulation drift alarmı
- Blue fix uygulanma doğrulaması
- Regression: eski kural bozuldu alarmı
- Playbook coverage check
- Atomic test coverage alarmı
- IR communications checklist alarmı

**Hunting (4):**
- Emulation'da kaç TTP kaçtı?
- Fix sonrası aynı TTP yakalanıyor mu?
- Regression var mı (eski kural kırıldı)?
- IR iletişim adımları aksadı mı?

**DFIR (1):**
- Adversary emulation turu timeline

---

## Ay 25 – AD Attacks (RED-lite)
**Identity/IAM (4):**
- Kerberos derinleşme
- AS-REP/kerberoast savunması
- DC koruması (tiering)
- AdminSDHolder/PACL

**Cloud Defense (4):**
- Org-level **SCP**: **root key usage** ve **policy deletion** yasak.
- Azure **Policy**: **NSG açık port** deny; **public IP** deny (etiket bazlı istisna).
- Key Vault **purge protection** ve **RBAC** etkin; access-policy yerine RBAC.
- AWS **Network ACL**: yüksek riskli portlara stateful/ingress kısıtı.

**Detection Rules (8):**
- Kerberoast artefact tespiti (T1558.003)
- AS-REP Roast tespiti (T1558.004)
- Pass-the-Hash/Ticket paterni (T1550)
- DCSync izleri (T1003.006)
- AdminSDHolder değişimi uyarısı
- Mimikatz imza + davranışsal tespit
- LDAP enumeration burst
- GPO değişikliği denetimi

**Hunting (4):**
- Kerberoast tetikleyen SPN'ler kim?
- AS-REP hedef kullanıcılar?
- Pass-the-Hash denemeleri hangi makinelerde?
- DCSync'e yakın LDAP patterni var mı?

**DFIR (1):**
- AD kerberoast olayı timeline

---

## Ay 26 – Priv-Esc & Persistence (RED-lite)
**Identity/IAM (4):**
- Local privesc engelleri
- UAC/policy sertleştirme
- Lateral harekette kimlik sinyalleri
- Token/cred guard

**Cloud Defense (4):**
- Centralized **log archive account**; harici erişim deny.
- Key Vault **purge protection** ve **RBAC** etkin; access-policy yerine RBAC.
- AWS **Network ACL**: yüksek riskli portlara stateful/ingress kısıtı.
- Azure **Key Vault**: **soft delete + purge protection** + **firewall** + **private endpoint**.
  
**Detection Rules (8):**
- winPEAS/linPEAS bulgusu → kural
- SUID/Capability abuse tespiti
- Service binpath hijack (T1574.011)
- UAC bypass paterni
- DLL search order hijack tespiti
- Scheduled task abuse tespiti
- Token impersonation paterni
- Hidden file/alt stream tespiti

**Hunting (4):**
- winPEAS/linPEAS bulguları yaygın mı?
- DLL hijack olası yollar nerede?
- Token impersonation kimde?
- Hidden file/ADS kullanan süreç var mı?

**DFIR (1):**
- Priv-esc + persistence olayı timeline

---

## Ay 27 – Tunneling & Pivoting (RED-lite)
**Identity/IAM (4):**
- Pivoting sırasında kimlik izleri
- Egress policy + kimlik
- Jump host ilkeleri
- Strong auth off-network

  **Cloud Defense (4):**
- Cross-account role **external ID** zorunlu.
- AWS **Network ACL**: yüksek riskli portlara stateful/ingress kısıtı.
- Azure **Key Vault**: **soft delete + purge protection** + **firewall** + **private endpoint**.
- S3 **Server Access Logging** kritik bucket’larda açık; ayrı log bucket’ına yaz.

**Detection Rules (8):**
- chisel/ligolo beacon periodicity
- SSH reverse tunnel paterni
- Proxychains DNS paterni
- socat port forward izleri
- Multiple hops latency paterni
- Egress port policy ihlali
- DNS over HTTPS aşırı kullanım
- Split-tunnel trafik anomalisi

**Hunting (4):**
- Reverse tunnel açan hostlar hangi segmentte?
- Proxychains kullanan process var mı?
- DoH artışıyla aynı saatlerde hangi olaylar var?
- Egress policy ihlal eden servisler hangileri?

**DFIR (1):**
- Pivoting/tunnel olayı timeline

---

## Ay 28 – Cloud & IAM (Derin)
**Identity/IAM (4):**
- Cloud IAM derinleşme
- Policy-as-code (Azure/AWS)
- Key/secret vault hygiene
- Federasyon dış entegrimler

**Cloud Defense (4):**
- Multi-region **GuardDuty/Defender** delegasyonu ve otomatize aktivasyon.
- Azure **Key Vault**: **soft delete + purge protection** + **firewall** + **private endpoint**.
- S3 **Server Access Logging** kritik bucket’larda açık; ayrı log bucket’ına yaz.
- AWS **SSM Session Manager** ile **SSH/RDP kapalı** yönetim (audit’li).
  
**Detection Rules (8):**
- Terraform drift değişimi alarmı
- Open S3/Blob denetimi (periyodik)
- IAM policy diff alarmı
- Azure Policy ihlal alarmı
- Role assumption outlier (STS)
- Service principal key age > N gün
- Key vault secret erişim anomali
- CloudTrail kaybı/bozulması alarmı

**Hunting (4):**
- IAM diff sonrası kimler etkilendi?
- Open bucket/blobs tekrar açılıyor mu?
- Key/secret erişim outlier'ları kimde?
- CloudTrail boşluk/friction dönemleri?

**DFIR (1):**
- Cloud IAM misconfig → exploit → fix timeline

---

## Ay 29 – DevSecOps/K8s
**Identity/IAM (4):**
- CI/CD kimlikleri
- K8s service account'ları
- Workload identity federation
- Secret rotation otomatizasyonu

**Cloud Defense (4):**
- Org-level **SCP**: **root key usage** ve **policy deletion** yasak.
- S3 **Server Access Logging** kritik bucket’larda açık; ayrı log bucket’ına yaz.
- AWS **SSM Session Manager** ile **SSH/RDP kapalı** yönetim (audit’li).
- Azure **Policy**: **NSG açık port** deny; **public IP** deny (etiket bazlı istisna).
  
**Detection Rules (8):**
- Supply chain: unsigned artefact tespiti
- SBOM değişim alarmı
- CI runner secret sızıntısı
- Container image high CVE gate
- IaC policy ihlali (tfsec/Checkov)
- K8s RBAC aşırı yetki tespiti
- K8s NetworkPolicy boşluğu alarmı
- Admission controller bypass paterni

**Hunting (4):**
- SBOM değişimi kritik paketlerde mi?
- CI runner secret sızıntısı işareti var mı?
- K8s RBAC aşırı yetkiler hangi namespacelerde?
- Admission bypass denemeleri nerede?

**DFIR (1):**
- CI/CD secret sızıntısı olayı timeline

---

## Ay 30 – OSCP-stili Prova + Rapor
**Identity/IAM (4):**
- Tatbikat kimlik senaryoları
- Break-glass test
- Access review sonuçları
- Son ay governance toparlama

**Cloud Defense (4):**
- Centralized **log archive account**; harici erişim deny.
- AWS **SSM Session Manager** ile **SSH/RDP kapalı** yönetim (audit’li).
- Azure **Policy**: **NSG açık port** deny; **public IP** deny (etiket bazlı istisna).
- **Secrets Manager/Key Vault** zorunlu; kod/CI’da plain secret yasak.

**Detection Rules (8):**
- Tatbikat T1059 (script) kuralı doğrulama
- Tatbikat T1003 (cred dump) kuralı doğrulama
- Tatbikat T1071 (C2) kuralı doğrulama
- Tatbikat T1041 (exfil) kuralı doğrulama
- Tatbikat T1021 (lateral) kuralı doğrulama
- Tatbikat T1053 (schtasks) doğrulama
- Tatbikat T1566 (phish) doğrulama
- Tatbikat T1547 (autorun) doğrulama

**Hunting (4):**
- Tatbikat sırasında kaçmayan/kaçan TTP'ler?
- Hangi zincirde false negative kaldı?
- Hangi kural en çok değer kattı?
- IR süresi nerede uzadı?

**DFIR (1):**
- Final tatbikat: Red→Blue→IR tam zincir timeline

---

