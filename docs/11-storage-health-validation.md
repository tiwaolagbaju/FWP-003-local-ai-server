# Phase 8 — Dual NVMe Health Validation

## Goal

Validate both installed 2TB NVMe drives after the PCIe fault-isolation sequence, while keeping device-specific identifiers out of the public project documentation.

## Current Storage Layout

- **WD Blue SN5000 2TB** — operating-system drive
- **SanDisk Optimus 5100 2TB** — secondary storage drive

Linux enumerates both drives successfully.

## SanDisk Optimus 5100 Health

SMART / NVMe health results:

- Critical warning: **0**
- Temperature: **24 C**
- Available spare: **100%**
- Percentage used: **0%**
- Media errors: **0**
- Error-log entries: **0**
- Warning-temperature time: **0**
- Critical-temperature time: **0**
- Power cycles: **6**
- Power-on hours: **0**
- Unsafe shutdowns: **5**

Interpretation: the secondary drive is effectively new and shows no media or thermal-health concerns. The small unsafe-shutdown count is consistent with recent hardware bring-up / troubleshooting activity and is not accompanied by media errors.

## WD Blue SN5000 Health

SMART / NVMe health results:

- Critical warning: **0**
- Composite temperature: **35 C**
- Available spare: **100%**
- Percentage used: **0%**
- Data read: approximately **3.0 TB**
- Data written: approximately **8.45 TB**
- Power cycles: **866**
- Power-on hours: **7,380**
- Unsafe shutdowns: **75**
- Media errors: **0**
- Error-log entries: **0**
- Warning-temperature time: **0**
- Critical-temperature time: **0**
- Highest reported internal sensor at capture: **55 C**

Interpretation: the OS drive has substantial prior operating time but currently reports no media errors, no NVMe critical warning, full available spare, and no recorded warning/critical-temperature duration.

The higher internal sensor value is being noted for future monitoring, but the composite temperature is normal and no thermal-management events were recorded.

## Result

| Check | Result |
|---|---|
| Both NVMe devices enumerate | **PASS** |
| SanDisk health | **PASS** |
| WD Blue health | **PASS** |
| Media errors | **0 on both drives** |
| Critical warning | **0 on both drives** |
| Thermal warning history | **0 on both drives** |
| Second SSD implicated in prior PCIe 928 fault | **No — ruled out by controlled reintegration** |

## Security / Documentation Note

Drive serial numbers and other unique hardware identifiers were intentionally omitted from public documentation.

## Engineering Takeaway

The secondary SSD was removed and reintroduced as a controlled variable during PCIe troubleshooting. It now boots and enumerates normally, and SMART data shows both NVMe drives are healthy. This allows the dual-SSD configuration to remain part of the known-good baseline before the second V620 is reintroduced.