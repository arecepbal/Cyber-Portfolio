# TCP ve UDP Hızlı Rehber

---

## TCP (yaygın)

- **20/21** — FTP (data/control)  
- **22** — SSH (SFTP dâhil)  
- **23** — Telnet  
- **25** — SMTP  
- **53** — DNS (büyük yanıtlar/zone transferleri)  
- **80** — HTTP  
- **110** — POP3  
- **143** — IMAP  
- **179** — BGP  
- **389** — LDAP  
- **443** — HTTPS  
- **445** — SMB/CIFS  
- **465** — SMTPS (eski)  
- **587** — SMTP Submission  
- **636** — LDAPS  
- **853** — DNS over TLS  
- **873** — rsync  
- **993** — IMAPS  
- **995** — POP3S  
- **1433** — Microsoft SQL Server  
- **1521** — Oracle Listener  
- **2049** — NFS  
- **2379/2380** — etcd  
- **3306** — MySQL/MariaDB  
- **3389** — RDP  
- **5432** — PostgreSQL  
- **5900** — VNC  
- **5985/5986** — WinRM (HTTP/HTTPS)  
- **6379** — Redis  
- **6443** — Kubernetes API  
- **8080** — HTTP-alt  
- **8443** — HTTPS-alt  
- **9200/9300** — Elasticsearch (HTTP/transport)  
- **9418** — Git (smart protocol)

---

## UDP (yaygın)

- **53** — DNS  
- **67/68** — DHCP (server/client)  
- **69** — TFTP  
- **123** — NTP  
- **135** — MS RPC Endpoint Mapper  
- **137/138** — NetBIOS (name/datagram)  
- **161/162** — SNMP / SNMP Trap  
- **500** — IKE (IPsec)  
- **514** — Syslog  
- **520** — RIP  
- **1900** — SSDP (UPnP)  
- **3478** — STUN/TURN (**5349/TCP** üzeri TLS)  
- **4500** — IPsec NAT-T  
- **5060** — SIP (**5061/TCP** üzeri TLS)  
- **5353** — mDNS  
- **1812/1813** — RADIUS (auth/accounting)  
- **5355** — LLMNR  
- **11211** — Memcached  
- **443/UDP** — QUIC / HTTP/3

> Not: Bazı servisler hem TCP hem UDP kullanabilir (ör. **DNS 53**, **SIP 5060/5061**).

---

## TCP vs UDP — Hızlı Karşılaştırma

| Özellik | TCP | UDP |
|---|---|---|
| **Bağlantı** | Var (3-yollu el sıkışma: `SYN → SYN/ACK → ACK`) | Yok (datagram; el sıkışma yok) |
| **Güvenilirlik** | Var (ACK, tekrar iletim, sıra numarası) | Yok (en iyi çaba; kayıp/duplikasyon uygulamaya kalır) |
| **Sıralama** | Garantili (uygulamaya doğru sırada verir) | Garanti yok |
| **Akış kontrolü** | Var (receiver window) | Yok |
| **Tıkanıklık kontrolü** | Var (CUBIC/BBR vb.) | Protokolde yok (uygulama/protokol üstte yapabilir) |
| **Veri modeli** | Akış (stream) – “bayt nehri” | Datagram – her paket bağımsız mesaj |
| **Başlık boyutu** | ≥ 20 bayt (opsiyonlarla artar) | 8 bayt |
| **Parçalanma** | TCP bölüp yeniden birleştirir | IP parçalanırsa kayıp tüm datagramı düşürür |
| **NAT/Firewall geçişi** | Genelde kolay (443/TCP yaygın) | Ağa bağlı; bazı yerler sıkı |
| **Tipik kullanım** | Web (HTTP/HTTPS), SSH, SMTP, IMAP/POP3, veritabanı | DNS, VoIP/RTP, oyun, video yayını, DHCP, QUIC(HTTP/3) |

---

## Nasıl Çalışırlar?

### TCP (Transmission Control Protocol)
- **Bağlantı kurar:** `SYN → SYN/ACK → ACK`  
- **Güvenilir ve sıralı teslim:** Kaybolanı **yeniden iletir**, **ACK** ile onaylar, **doğru sırada** teslim eder.  
- **Akış & kontrol:** Akış kontrolü (alıcıyı boğmama) ve tıkanıklık kontrolü (ağı boğmama) yapar.  
- **Mesaj sınırı yoktur:** “Bayt akışı”dır; uygulama mesaj çerçevelemesini kendisi yapar.  
- **Örnek:** `10.0.0.5:53512 → 93.184.216.34:443` (istemci ephemeral → sunucu sabit port).

### UDP (User Datagram Protocol)
- **Bağlantı yok:** El sıkışma olmadan gönder-al; gecikme düşüktür.  
- **Garanti yok:** Paket **kaybolabilir**, **sırası değişebilir**, **duplike** olabilir.  
- **Mesaj temelli:** Her UDP datagramı **tek mesaj**tır.  
- **Üst protokollerle zenginleşir:** Güvenilirlik/akış/tıkanıklık gerekiyorsa üst katmanda sağlanır.  
- **Örnek:** DNS sorgusu `kaynak:ephemeral → hedef:53/UDP`.

> **QUIC notu:** QUIC, **UDP üzerinde** çalışan; **akışlar, tıkanıklık kontrolü ve TLS 1.3’ü** kullanıcı alanında birleştiren modern bir protokoldür. **HTTP/3** QUIC üzerindedir (**443/UDP** yaygın).

---

## Hangi Durumda Hangisi?

- **Kesin, sıralı ve eksiksiz teslim şartsa** (dosya, web sayfası, veritabanı, SSH): **TCP**  
- **Düşük gecikme ve ara sıra kayıp tolere edilebiliyorsa** (VoIP, canlı yayın, oyun): **UDP**  
- **Düşük gecikme + güvenlik/süreklilik** isteniyorsa ve uçları kontrol ediyorsanız: **QUIC/HTTP/3** gibi UDP tabanlı çözümler

---

> Ek notlar: Portlar L4’tedir (0–65535). İstemci portu genelde **ephemeral** (dinamik) seçilir; sunucu **sabit** portta dinler. Port değiştirmek güvenlik sağlamaz; güvenlik **TLS + kimlik doğrulama + yetkilendirme + doğru firewall/ACL** ile sağlanır.
