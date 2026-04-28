# LINKTYPE_DECT_NR_TAP 

**DECT‑2020 NR frames with custom metadata TAP header**

`LINKTYPE_DECT_NR_TAP` (with `DLT_DECT_NR_TAP = 304`) is a link‑layer header type for **DECT‑2020 New Radio (NR)** frames that are captured with a **custom TAP‑style metadata header** prepended. The payload after the metadata header is a standard `DLT_DECT_NR` MAC‑layer frame, so dissection can be done using the same parsers as for raw `DLT_DECT_NR`.

## Overall frame layout

Each captured packet has this structure:

- 8‑byte custom header  
- Variable‑length TLV section  
- Raw `DLT_DECT_NR` modem‑layer data (PCC bytes followed by PDC bytes)

The total length in the header includes the 8‑byte header, all TLVs, and the modem data.

## The header layout

### 1. Custom header (8 bytes)

All fields are **little‑endian**.

| Offset | Field        | Size, bytes | Value / Notes |
|--------|--------------|-------------|---------------|
| 0      | Version      | 1           | 1             |
| 1      | Reserved1    | 1           | 0             |
| 2–3    | TLVs length  | 2           | Total length of all TLVs in bytes |
| 4–5    | Length       | 2           | Total length of header + TLVs + modem data |
| 6–7    | Reserved2    | 2           | 0             |

### 2. TLV format

Each TLV is 8 bytes on the wire.
All fields are **little‑endian**.

| Field  | Size, bytes | Notes |
|--------|-------------|-------|
| Type   | 2           | Identifies TLV semantics |
| Length | 2           | 1–4 bytes (actual "Value" field size) |
| Value  | 4           | Padded; if actual payload is < 4 bytes, trailing bytes are 0 |

#### Known TLV types

| Type | TLV name                             | Payload size, bytes | Meaning |
|------|--------------------------------------|----------------|---------|
| 1    | Data Type TLV                        | 2              | Indicates the length of PCC bytes and presence of PDC data: <br> 1 -  PCC length is 40 bits; <br> 2 -  PCC length is 80 bits; <br> 3 -  PCC length is 40 bits, PDC bytes is absent due to PDC CRC error; <br> 4 -  PCC length is 80 bits, PDC bytes is absent due to PDC CRC error; <br> 5 - PCC CRC error, neither PCC nor PDC bytes are included; |
| 2    | Header status TLV                    | 2              | Indicates PCC status retrieved from modem: <br> 0 -  Valid status, PDC bytes are included in packet; <br> 1 -  Invalid status, PDC bytes are absent; <br> 2 -  Valid status, but RX operation ends before PDC reception, PDC bytes are absent; |
| 3    | PCC RSSI2 TLV                       | 2 signed       | PCC RSSI (Received Signal Strength Indicator). Values are in dBm with 0.5 dBm resolution (Q14.1). <br> For example, -87 is -43.5 dbm,  -84 is -42,0 dbm.|
| 4    | PCC SNR TLV                          | 2 signed       | PCC Signal‑to‑Noise Ratio. Values are dB values with 0.25 dB resolution (Q13.2)|
| 5    | PDC RSSI2 TLV                       | 2 signed       | The same as for PCC RSSI 2, only for PDC. |
| 6    | PDC SNR TLV                          | 2 signed       | The same as for PCC SNR, only for PDC. |
| 7    | Transaction ID TLV                   | 2              | Session / transaction ID. Used to map PCC data with corresponding PDC data. |
| 8    | Channel TLV                          | 2              | Radio channel ID |
| 9    | Simul trace string number TLV        | 4              | Simulator‑specific string index |

### 3. After the TLVs 
After the TLVs the actual `DLT_DECT_NR` MAC‑layer data (PCC bytes followed by PDC bytes) is present.

## Usage notes

- This link‑type is intended for **DECT‑2020 NR** captures where the RF‑layer or tester adds a per‑packet metadata header with signal‑quality and channel information.
- Dissectors should skip the 8‑byte header and TLVs, then pass the remaining bytes to a standard `DLT_DECT_NR` parser.

For details on the DECT‑2020 NR base protocol, see ETSI TS 103‑636‑4 (MAC‑layer) and ETSI TS 103‑636‑5 (DLC and convergence layers).
