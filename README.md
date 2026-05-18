# LINKTYPE_DECT_NR_TAP 

**DECT‑2020 NR frames with custom metadata TAP header**

`LINKTYPE_DECT_NR_TAP` (with `DLT_DECT_NR_TAP = 304`) is a link‑layer header type for **DECT‑2020 New Radio (NR)** frames that are captured with a **custom TAP‑style metadata header** prepended. The payload after the metadata header is a standard `DLT_DECT_NR` MAC‑layer frame, so dissection can be done using the same parsers as for raw `DLT_DECT_NR`.

## Overall frame layout

Each captured packet has this structure:

- 8‑byte custom header  
- N 8‑byte metadata entries
- Raw `DLT_DECT_NR` modem‑layer data (PCC bytes followed by PDC bytes)

All multi‑byte integer fields are little‑endian unless noted otherwise.

## The header layout

### 1. Custom header (8 bytes)

All fields are **little‑endian**.

| Offset | Field                | Size, bytes | Value / Notes |
|--------|----------------------|-------------|---------------|
| 0      | Version              | 1           | 1             |
| 1      | Reserved1            | 1           | 0             |
| 2–3    | Metadata entry count | 2           | Number of following fixed metadata entries (each entry is exactly 8 bytes) |
| 4–5    | Modem length         | 2           | Length in bytes of the modem data (PCC + PDC) |
| 6–7    | Reserved2            | 2           | 0             |

### 2. Metadata entry format

Each metadata entry is a fixed 8‑byte structure.
All fields are **little‑endian**.

| Field    | Size, bytes | Notes |
|----------|-------------|-------|
| Type     | 2           | Identifies entry semantics |
| Reserved | 2           | 0 (for future use) |
| Value    | 4           | Padded; if actual payload is < 4 bytes, trailing bytes are 0 |

#### Known metadata entry Types

| Type | Metadata name                             | Value size, bytes | Meaning |
|------|-------------------------------------------|----------------|---------|
| 1    | Data Type entry                           | 2              | Indicates the length of PCC bytes and presence of PDC data: <br> 1 -  PCC length is 40 bits; <br> 2 -  PCC length is 80 bits; <br> 3 -  PCC length is 40 bits, PDC bytes is absent due to PDC CRC error; <br> 4 -  PCC length is 80 bits, PDC bytes is absent due to PDC CRC error; <br> 5 - PCC CRC error, neither PCC nor PDC bytes are included; |
| 2    | Header status entry                       | 2              | Indicates PCC status retrieved from modem: <br> 0 -  PHY header is valid; PDC can be received. <br> 1 - PHY header is invalid; PDC can't be received. <br> 2 -  PHY header is valid, but RX operation ends before PDC reception; |
| 3    | PCC RSSI2 entry                           | 2 signed       | PCC RSSI (Received Signal Strength Indicator). Values are in dBm with 0.5 dBm resolution (Q14.1). <br> For example, -87 is -43.5 dbm,  -84 is -42,0 dbm. |
| 4    | PCC SNR entry                             | 2 signed       | PCC Signal‑to‑Noise Ratio. Values are dB values with 0.25 dB resolution (Q13.2) |
| 5    | PDC RSSI2 entry                           | 2 signed       | PDC RSSI (Received Signal Strength Indicator). Values are in dBm with 0.5 dBm resolution (Q14.1). <br> For example, -87 is -43.5 dbm,  -84 is -42,0 dbm. |
| 6    | PDC SNR entry                             | 2 signed       | PDC Signal‑to‑Noise Ratio. Values are dB values with 0.25 dB resolution (Q13.2) |
| 7    | Transaction ID entry                      | 2              | Session / transaction ID. Used to map PCC data with corresponding PDC data. |
| 8    | Channel entry                             | 2              | Radio channel ID |
| 9    | Trace log entry number                    | 4              | Nearest trace log entry string index |

### 3. After the Metadata 
After the metadata entries the actual `DLT_DECT_NR` MAC‑layer data (PCC bytes followed by PDC bytes) is present.

## Usage notes

- This link‑type is intended for **DECT‑2020 NR** captures where the RF‑layer or tester adds a per‑packet metadata header with signal‑quality and channel information.
- Dissectors should skip the 8‑byte header and metadata entries, then pass the remaining bytes to a standard `DLT_DECT_NR` dissector for interpretation.

For details on the DECT‑2020 NR base protocol, see ETSI TS 103‑636‑4 (MAC‑layer) and ETSI TS 103‑636‑5 (DLC and convergence layers).
