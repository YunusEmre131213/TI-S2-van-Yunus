# Practicum UART (deel 2) – Logic Analyzer

Tijdens deze les heb ik UART-communicatie opgezet tussen twee PCs via CP2102 USB-to-UART adapters en de communicatie gemeten met mijn Logic Analyzer (24MHz 8CH).

## Hardware opzet

### Benodigdheden
- 2× CP2102 USB 2.0 to UART TTL 5-pin adapter
- Logic Analyzer 24MHz 8CH
- Verbindingskabels (jumpers)

### Aansluitschema

| 1e USB-2-TTL | 2e USB-2-TTL |
| :----:       | :----:       |
| TXD          | RXD          |
| RXD          | TXD          |
| GND          | GND          |

> **Let op:** De pinnen `3V3` en `+5V` worden **niet** gebruikt in deze opdracht.

De Logic Analyzer is aangesloten op:
- **CH0** → TXD van PC1 (= RXD van PC2)
- **CH1** → RXD van PC1 (= TXD van PC2)
- **GND** → gemeenschappelijke GND

## Software instelling

### Seriële Terminal (PuTTY)

Op beide PCs is PuTTY gebruikt als seriële terminal:

| Instelling   | Waarde  |
|:-------------|:--------|
| Baud rate    | 9600    |
| Data bits    | 8       |
| Stop bits    | 1       |
| Parity       | None    |
| Flow control | None    |

Dit is de standaard **8N1** UART-configuratie.

### Logic Analyzer instelling (24MHz 8CH)

In de Logic Analyzer software (bijv. Saleae Logic / PulseView):

| Instelling        | Waarde     |
|:------------------|:-----------|
| Sample rate       | 1 MHz      |
| Capture tijd      | 1 seconde  |
| Trigger           | Falling edge op CH0 |
| Decoder           | UART / Async Serial |
| Baud rate decoder | 9600        |
| Bit order         | LSB first   |
| Stop bits         | 1           |
| Parity            | None        |

## Meting en bevindingen

### Wat je ziet in de Logic Analyzer

Een UART-frame bij het versturen van 1 byte (bijv. de letter **'A'** = ASCII 0x41 = 0b01000001) ziet er zo uit:

```
Idle:   ─────────────────────────────── (HIGH = 1)
Start:       ___                        (LOW = 0, 1 bit)
Data:           D0 D1 D2 D3 D4 D5 D6 D7 (LSB eerst)
Stop:                               ─── (HIGH = 1, 1 bit)
```

Voor 'A' (0x41 = 01000001 binair):
- LSB first: 1, 0, 0, 0, 0, 0, 1, 0
- Start bit: LOW
- Bit 0 (LSB): HIGH (1)
- Bit 1: LOW (0)
- Bit 2: LOW (0)
- Bit 3: LOW (0)
- Bit 4: LOW (0)
- Bit 5: LOW (0)
- Bit 6: HIGH (1)
- Bit 7 (MSB): LOW (0)
- Stop bit: HIGH

Bij 9600 baud duurt elke bit: **1 / 9600 ≈ 104.2 µs**

### Persoonlijke bevindingen

- De Logic Analyzer toont duidelijk het start-bit (dalende flank) en stop-bit (stijgende flank) per byte.
- Door de UART-decoder toe te voegen in de software wordt automatisch de ASCII-waarde weergegeven naast het signaal — dit klopt precies met wat ik in PuTTY typte.
- TXD van PC1 is zichtbaar op CH0: elke keer als ik een teken tik op PC1, zie ik een puls op CH0.
- RXD van PC1 (= TXD van PC2) is zichtbaar op CH1: tekens getikt op PC2 zijn hier te zien.
- GND is **essentieel** — zonder gemeenschappelijke GND werkt de meting niet betrouwbaar.
- De 24MHz sample rate van de Logic Analyzer is meer dan voldoende voor 9600 baud (Nyquist: minimaal 2× 9600 Hz = ~19.2 kHz nodig).

## Screenshot Logic Analyzer

> *(Voeg hier een screenshot in van de Logic Analyzer meting, waarop het UART-frame zichtbaar is met de gedecodeerde waarde)*

Aanbevolen: maak een screenshot waarbij je de decoder-annotatie zichtbaar hebt, zodat de byte-waarde automatisch wordt weergegeven boven het signaal.

## Conclusie

UART is een eenvoudig maar effectief protocol: één start-bit, 8 data-bits (LSB eerst), één stop-bit. Met de Logic Analyzer is het goed te zien hoe elke byte exact wordt overgedragen. De 24MHz 8CH Logic Analyzer geeft meer dan genoeg resolutie voor standaard UART-baudrates zoals 9600 en zelfs 115200 baud.
