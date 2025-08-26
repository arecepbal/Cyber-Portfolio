# W1 – Day 2: OSI & Command Outputs

**Goal:** OSI katmanlarını pratikle bağlamak; ipconfig/arp/ping/tracert çıktılarının neyi doğruladığını göstermek

**Setup:** PT , 2960 switch, 3×PC. IP: .10/.11/.12 (/24), gateway yok

## Evidence
- ipconfig: ![ipconfig](../diagrams/w1-ipconfig.png)
- ping:     ![ping](../diagrams/w1-ping.png)
- arp -a:   ![arp](../diagrams/w1-arp-table.png)
- (opt) tracert: ![tr](../diagrams/w1-tracert.png)
- Simulation: ![sim](../diagrams/w1-sim-steps.png)
- Packet Tracer: [w1-lab1-basic-lan.pkt](../labs/w1-lab1-basic-lan.pkt)

## Findings
- İlk ping maximum ARP cache yüzünden; sonraki pingler daha hızlı
- ARP tablosunda .11 ve .12 için MAC eşleşmeleri oluştu, arp -a ile command promptda görüldü
- arp -d sonrası arp -a da MAC yok, No ARP Entries Found çıktısı.
- tracert commandda arada router olmadığı için tek hop ile PC0 PC2 ye ulaştı
- PDI ınformatoın kısmındaki OSI layersda ARP kısmında destination kısmı broad cast ve PC1 in MAC adresi gözükmüyorken,
ICMP kısmında destination kısmında PC1'in IP nin adresi ve  MAC adresi görünüyordu

