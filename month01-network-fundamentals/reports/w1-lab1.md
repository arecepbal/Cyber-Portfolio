# W1 - Lab 1: Basic LAN (1 switch + 2 PC)

**Amaç:** Aynı LAN’da iki PC’yi konuşturmak; ping ve ARP gözlemi.
**Ortam:** Packet Tracer 8.x, Switch-2960, PC0, PC1.

## IP Planı
- PC0: 192.168.1.10/24
- PC1: 192.168.1.11/24

## Adımlar
1) Cihazlar bağlandı (Copper Straight-Through).
2) IP/mask atandı (gateway boş).
3) PC0 → PC1 `ping`.
4) `arp -a` ile ARP önbelleği.

## Kanıtlar
- Ping: ![ping]
- ARP: ![arp](../diagrams/w1-lab1-arp_-a.png)

## Bulgular
- Ping başarılı; ARP’de 192.168.1.11 ↔ MAC eşleşmesi var.

## Sonuç
- Temel LAN iletişimi ve ARP gözlemi tamam.
