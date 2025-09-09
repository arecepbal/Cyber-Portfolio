# L3 Addressing & Ops — Quick Vocabulary (US IPA)

| Term (EN) | TR | POS | IPA (US) | Kısa Açıklama (TR) | Example (EN) |
|---|---|---|---|---|---|
| `CIDR` *(Classless Inter-Domain Routing)* | CIDR *(Sınıfsız Alanlar Arası Yönlendirme)* | noun | /ˈsaɪdər/ | Sınıfsız IP adresleme; ağları **önek uzunluğu** ile yazar (örn. `/24`) ve özetleme yapar. | Use CIDR like `10.1.0.0/16` to summarize multiple subnets. |
| `prefix` *(prefix length)* | ön ek / önek uzunluğu | noun | /ˈpriˌfɪks/ | IP ağının **ağ kısmını** belirleyen bit sayısı (örn. `/24` = 24 ağ biti). | The prefix `/24` means 255.255.255.0 as the mask. |
| `network address` | ağ adresi | noun phrase | /ˈnɛtˌwɝk ˈædˌrɛs/ | Alt ağın ilk adresi; **host bitleri 0**. Host’a atanmaz. | For `192.168.1.42/24`, the network address is `192.168.1.0`. |
| `broadcast address` *(IPv4)* | yayın adresi | noun phrase | /ˈbrɔdˌkæst ˈædˌrɛs/ | Alt ağın son adresi; **host bitleri 1**; aynı alt ağdaki herkese gider. | In `192.168.1.0/24`, the broadcast is `192.168.1.255`. |
| `routing` | yönlendirme | noun | /ˈraʊtɪŋ/ | Paketlerin hedef **önek**e göre L3’te iletilmesi (statik/dinamik). | Dynamic routing protocols exchange prefixes to build the RIB. |
| `baseline` | temel çizgi / referans değer | noun | /ˈbeɪsˌlaɪn/ | Normal performansın referansı (gecikme, throughput, hata oranı vb.). | Establish a latency baseline before major changes. |
| `wildcard` *(ACL wildcard mask)* | joker/ters maske | noun | /ˈwaɪldˌkɑːrd/ | Cisco ACL’de **maskenin tersi**: `0` eşleş, `1` umursama. `/24` için `0.0.0.255`. | `permit ip 192.168.1.0 0.0.0.255 any` matches the whole /24. |
| `validation` | doğrulama | noun | /ˌvæləˈdeɪʃən/ | Yapılandırma/verinin kurallara uygunluğunu kontrol etme. | Run config validation for CIDR blocks and ACLs before deploy. |
| `ACL (Access Control List)` | erişim kontrol listesi | noun | /ˌeɪ siː ˈɛl/ | Trafiği **izin/verme** kurallarıyla filtreleyen liste; Cisco ACL’lerde **wildcard mask** kullanılır. | Apply an ACL to permit the management subnet and deny all else. |
| `OSPF (Open Shortest Path First)` | OSPF (Açık En Kısa Yol Önce) | noun | /ˌoʊ ɛs piː ˈɛf/ | **Link-state** yönlendirme protokolü; **alanlar (areas)** ve **SPF (Dijkstra)** kullanır; sınıfsızdır. | OSPF uses areas and SPF to compute shortest paths. |
| `EIGRP (Enhanced IGRP)` | EIGRP (Geliştirilmiş IGRP) | noun | /ˌiː aɪ ˈdʒiː ɑːr ˈpiː/ | Cisco’nun **advanced distance-vector** protokolü; **DUAL** ile hızlı yakınsama, sınıfsız, özetleme destekli. | EIGRP’s DUAL algorithm gives fast convergence after a failure. |
| `pattern wildcard (glob)` | desen jokeri (glob) | noun | /ˈpætɚn ˈwaɪldˌkɑrd/ | Metin/desen eşlemede `*` (her şey), `?` (tek karakter) gibi işaretler; **ACL wildcard mask ile aynı şey değildir**. | Use the pattern wildcard `10.1.*` if the tool supports glob matching. |
| `contiguous (mask/prefix)` | bitişik (maske/önek) | adj. | /kənˈtɪɡjuəs/ | **1** bitleri aralıksız olan standart ağ maskesi (örn. **255.255.255.0**); özetlemede **bitişik önekler** birleştirilir. | Two contiguous /25s can be summarized into a /24. |
| `non-contiguous (ACL wildcard)` | bitişik olmayan (joker) | adj. | /ˌnɑn kənˈtɪɡjuəs/ | **ACL wildcard**’da 0/1 bitleri karışık olabilir; belirli bit desenlerini seçmek için kullanılır. **Subnet maskesi değildir.** | A non-contiguous ACL wildcard like `0.0.5.255` matches specific bit patterns. |

> **Notlar:**
> - **ACL wildcard mask**: `0` = **eşle**, `1` = **umursama**. Örn. `/24` için `0.0.0.255`.  
> - **Pattern wildcard (glob)** dizgi/desen eşlemedir; IP alt ağlarıyla karışmasın.  
> - **OSPF** sınıfsızdır (VLSM/özetleme yapar); **EIGRP** de sınıfsızdır ve `no auto-summary` ile discontiguous topolojilerde doğru çalışır.
