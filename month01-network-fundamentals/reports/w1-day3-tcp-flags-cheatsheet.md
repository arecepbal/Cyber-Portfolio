## Appendix — TCP Flags (Cheatsheet)

### Flag bitleri
| Bit | Flag | Hex | Binary      | Açıklama                  |
|-----|------|-----|-------------|---------------------------|
| 7   | CWR  | 0x80| 10000000    | Congestion Window Reduced |
| 6   | ECE  | 0x40| 01000000    | ECN-Echo                  |
| 5   | URG  | 0x20| 00100000    | Urgent                    |
| 4   | **ACK** | **0x10**| **00010000** | Acknowledge             |
| 3   | PSH  | 0x08| 00001000    | Push                      |
| 2   | RST  | 0x04| 00000100    | Reset                     |
| 1   | **SYN** | **0x02**| **00000010** | Synchronize             |
| 0   | FIN  | 0x01| 00000001    | Finish                    |

### 3-Way Handshake (beklenen flaglar)
1) **SYN** → `0x02` (`00000010`) — client → server  
2) **SYN/ACK** → `0x12` (`00010010`) — server → client  
3) **ACK** → `0x10` (`00010000`) — client → server

### Packet Tracer’da nasıl okunur?
- Simulation → **Event List**’te bir **TCP** olayına **tıkla**  
- **Inbound/Outbound PDU Details → TCP** bölümünde **FLAGS** alanını oku  
  - Bazı sürümlerde ikili gösterim satır kırar, örn.  
    `0b000100` + `10` ⇒ **`0b00010010`** (= SYN+ACK)  
- Yön/port kontrolü:  
  - SYN: `src=ephemeral → dst=80`  
  - SYN/ACK: `src=80 → dst=ephemeral`  
  - ACK: `src=ephemeral → dst=80`

