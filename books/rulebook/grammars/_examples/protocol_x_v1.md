# Protocol: [Name] (v1.0)
**Status:** Draft / Verified

## Packet Structure
| Offset | Field Name | Type | Description |
| :--- | :--- | :--- | :--- |
| 0 | Magic_Byte | uint8 | Constant: 0x5A |
| 1 | Sequence | uint8 | Increments per packet |
| 2-3 | Payload_Len| uint16 | Big Endian |

## Validation Logic (Pytest Rules)
- magic_byte must always be 0x5A.
- payload_len must match actual remaining bytes in frame.