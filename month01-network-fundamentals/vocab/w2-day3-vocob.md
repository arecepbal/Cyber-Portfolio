# L1/L2/L3 Vocabulary — Quick Reference (US IPA)

| Term (EN) | TR | POS | IPA (US) | Kısa Açıklama (TR) | Example (EN) |
|---|---|---|---|---|---|
| `repeater` | tekrarlayıcı | noun | /ɹɪˈpiːtɚ/ | Zayıflayan sinyali yenileyip aynı hızda tekrar eden fiziksel katman cihazı. | A repeater regenerates and retimes the signal. |
| `hub` | hub (çoklayıcı) | noun | /hʌb/ | Tüm portlara bitleri çoğaltan, paylaşımlı çakışma alanı oluşturan cihaz. | A hub repeats incoming bits to all other ports. |
| `CSMA/CD` | CSMA/CD (taşıyıcı algıla-çoklu erişim/çakışma saptama) | noun | /ˌsiː ˌɛs ˌɛm ˌeɪ ˌsiː ˈdiː/ | half-duplex, paylaşımlı ortamda çakışma yönetimi yöntemi (eski Ethernet). | Old Ethernet hubs used CSMA/CD to manage access. |
| `collision` | çakışma | noun | /kəˈlɪʒən/ | İki istasyonun aynı anda aktarımı sonucu çerçevelerin bozulması. | Collisions occur on half-duplex shared media. |
| `late collisions` | gecikmiş çakışmalar | noun (plural) | /leɪt kəˈlɪʒənz/ | Çerçevenin 512-bit(64 byte) sınırından sonra görülen çakışmalar; genelde kablo/duplex sorunu. | Late collisions often indicate a duplex or cabling problem. |
| `duplex` | çift yönlülük (duplex) | noun/adj. | /ˈduːˌplɛks/ | Aynı anda (full) veya sırayla (half) gönderme/alma modu. | Set the port to full duplex to avoid collisions. |
| `half-duplex` | yarı çift yönlü | adj./noun | /ˌhæf ˈduːˌplɛks/ | Gönderme ve alma **aynı anda değil**, sırayla yapılır; çakışma riski vardır. | In half-duplex, devices cannot transmit and receive simultaneously. |
| `full-duplex` | tam çift yönlü | adj./noun | /ˌfʊl ˈduːˌplɛks/ | Gönderme ve alma aynı anda yapılır; çakışmalar ortadan kalkar. | Full-duplex eliminates collisions on a switch port. |
| `mismatch (duplex mismatch)` | (duplex) uyuşmazlığı | noun | /ˌmɪsˈmætʃ/ | Uçların duplex ayarları farklı; geç kayıplar, geç çakışma, hatalar yaratır. | A duplex mismatch can cause late collisions and input errors. |
| `speed mismatch` | hız uyuşmazlığı | noun | /spiːd ˌmɪsˈmætʃ/ | Uçların hız ayarları farklı (örn. 10 vs 100 Mb/s); link kurulmayabilir. | A speed mismatch between endpoints can keep the link down. |
| `autonegotiation` | otomatik pazarlık | noun | /ˌɔtoʊnɪˌgoʊʃiˈeɪʃən/ | Uçların hız/duplex yeteneklerini otomatik uzlaştırması. | Enable autonegotiation on both ends of the link. |
| `FLP (Fast Link Pulse)` | hızlı bağlantı darbesi | noun | /ˌɛf ˌɛl ˈpiː/ | Autonegotiation sırasında yetenek ilanı için gönderilen darbe dizileri. | FLP bursts advertise capabilities during autonegotiation. |
| `parallel detection` | paralel algılama | noun | /ˈpærəˌlɛl dɪˈtɛkʃən/ | FLP yoksa yalnızca hızı (10/100) saptayıp duplex’i **half** varsayma. | Without FLPs, parallel detection often falls back to half-duplex. |
| `AUTO` | otomatik (AUTO) | adj./mode | /ˈɔtoʊ/ | Portun hız/duplex’i autonegotiation ile belirlemesi. | Set speed and duplex to AUTO on both sides. |
| `FORCE` | zorla ayarla (manuel) | adj./mode | /fɔrs/ | Hız/duplex’in elle sabitlenmesi; karşı uç AUTO ise uyuşmazlık riski. | One side was forced to 100/Full, causing a mismatch. |
| `10/100` | 10/100 Mb/sn | noun | /ˈtɛn wʌn ˈhʌndrəd/ | 10 veya 100 Mb/s destekleyen Ethernet hızları. | The old access switch supports only 10/100. |
| `1000BASE-T` | 1000BASE-T (Gigabit bakır) | noun | /wʌn ˈθaʊzənd beɪs ˈtiː/ | Cat5e+ kablo üzerinde 1 Gb/s Ethernet standardı. | The uplinks run over 1000BASE-T on Cat5e or better. |
| `interface` | arayüz (port) | noun | /ˈɪntɚˌfeɪs/ | Cihazın ağ bağlantı noktası (örn. Gi0/1). | Shut/no-shut the interface to reset the link. |
| `link up` | bağlantı kuruldu | noun phrase | /lɪŋk ʌp/ | Fiziksel bağlantı aktif. | The interface changed state to link up. |
| `link down` | bağlantı koptu | noun phrase | /lɪŋk daʊn/ | Fiziksel bağlantı pasif/arıza. | After the cable was pulled, the link went down. |
| `CRC` | döngüsel artıklık denetimi | noun | /ˌsiː ˌɑr ˌsiː/ | Frame sonundaki hata denetimi alanı(FCS); hatalıysa düşürülür. | Rising CRC errors suggest cabling or interference issues. |
| `input errors` | giriş hataları | noun (plural) | /ˈɪnˌpʊt ˈɛrərz/ | Arayüzün(portun) aldığı toplam hatalı çerçeveler (CRC, frame, overrun vb.). | The interface shows increasing input errors under load. |
| `jitter` | gecikme dalgalanması (jitter) | noun | /ˈdʒɪt̬ɚ/ | Gecikmenin zaman içindeki oynaklığı; ses/görüntü kalitesini bozar. | High jitter can break real-time voice quality. |
| `goodput` | yararlı aktarım hızı | noun | /ˈɡʊdˌpʊt/ | Protokol başlıkları ve yeniden iletimler çıkarıldıktan sonra net uygulama hızı. | After overhead, the goodput was about 85 Mbps. |
| `RTT (round-trip time)` | gidiş-dönüş süresi | noun | /ˌɑr tiː ˈtiː/ | Paketin hedefe gidip yanıtla geri dönme süresi. | The RTT to the server averaged 42 ms. |
| `variable RTT` | değişken RTT | noun | /ˈvɛriəbəl ˌɑr tiː ˈtiː/ | RTT’nin zamana göre dalgalanması; kuyruklanma/yoğunluk belirtisi. | Variable RTT caused choppy video playback. |
