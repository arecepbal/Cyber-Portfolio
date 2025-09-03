# L2/L3 Vocabulary — Quick Reference (US IPA)

| Term (EN) | TR | POS | IPA (US) | Kısa Açıklama (TR) | Example (EN) |
|---|---|---|---|---|---|
| `forward` | iletmek | verb | /ˈfɔɹ.wɚd/ | Switch’in çerçeveyi hedef porta göndermesi. | The switch forwards the frame out of Fa0/2. |
| `flood` | (unknown unicast) yaymak | verb/noun | /flʌd/ | Hedef MAC bilinmediğinde çerçevenin birden çok porta gönderilmesi. | The switch floods the frame because the destination MAC is unknown. |
| `MAC address` | MAC adresi | noun | /mæk ˈædˌrɛs/ | Her NIC’e özgü 48-bit donanım adresi. | Each NIC has a unique MAC address burned into the hardware. |
| `FCS` | çerçeve denetim dizisi | noun | /ˌɛf.siːˈɛs/ | Hata denetimi için çerçevenin sonundaki 32-bit CRC alanı. | A corrupted frame fails the FCS check and is dropped. |
| `header` | üstbilgi | noun | /ˈhɛdɚ/ | Çerçeve/paket/segment başındaki kontrol bilgileri. | The Ethernet header includes source and destination MACs. |
| `trailer` | altbilgi | noun | /ˈtreɪlɚ/ | Çerçevenin sonunda yer alan ek kontrol bilgisi (örn. FCS). | The frame’s trailer contains the FCS field. |
| `unicast` | tekil yayım | noun/adj. | /ˈjuːnɪˌkæst/ | Tek bir hedefe yönelik iletim türü. | The reply was sent as a unicast to the requester. |
| `broadcast` | yayın | noun/verb | /ˈbrɔdˌkæst/ | Aynı yerel ağdaki tüm cihazlara iletim. | ARP requests are broadcast to the local segment. |
