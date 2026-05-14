# R&D General Panel — PLC Program Repository

General-purpose R&D test panel built on a 5069-L320ER (CompactLogix 5380). The panel is a reusable platform; each deployment is a distinct project with its own I/O assignments and process logic layered on a common program structure.

**Toolchain:** Studio 5000 v36 · Ignition SCADA · Smartsheet (version control)

---

## Hardware

| Slot | Catalog | Name | Type | Channels |
|------|---------|------|------|----------|
| 0 | 5069-L320ER | Local | CPU | — |
| 1 | 5069-IB16/A | Slot1_DI | 24VDC DI | 16 |
| 2 | 5069-OB16/B | Slot2_DO | 24VDC DO | 16 |
| 3 | 5069-IF8/B | Slot3_AI | Analog In (4–20mA) | 8 |
| 4 | 5069-IY4/A | Slot4_TC | Thermocouple | 4 |
| 5 | 5069-IY4/A | Slot5_TC | Thermocouple | 4 |
| 6 | 5069-IY4/A | Slot6_TC | Thermocouple | 4 |

Controller IP: `192.168.1.11` · Firmware: v36.11

---

## Program Structure

```
MainTask (Continuous, 10ms, Priority 10, Watchdog 500ms)
  └── MainProgram
        ├── IO_MAP         ST — Physical I/O ↔ UDT array binding (only place raw I/O tags appear)
        ├── CM_DI          ST — Discrete input processing and alarms
        ├── CM_AI          ST — Analog input scaling and alarms
        ├── CM_TC          ST — Thermocouple processing
        ├── CM_DO          ST — Discrete output processing
        ├── CM_AO          ST — Analog output processing
        ├── [ProjectName]  ST/LAD — Project-specific process logic
        └── CM_ALARMS_AGG  ST — Aggregate alarm summary rollup
```

Raw I/O tags are referenced **only** in `IO_MAP`. All other routines use UDT arrays exclusively.

---

## UDT Arrays

| Tag | UDT | Size | Maps To |
|-----|-----|------|---------|
| CM_DIN | CM_DIN | [0..15] | Slot1_DI channels |
| CM_AIN | CM_AIN | [0..7] | Slot3_AI (4–20mA) |
| CM_AIN | CM_AIN | [8..11] | Slot4_TC channels |
| CM_AIN | CM_AIN | [12..15] | Slot5_TC channels |
| CM_AIN | CM_AIN | [16..19] | Slot6_TC channels |
| CM_SOV | CM_SOV | per project | Slot2_DO channels |

Unused channels: `TAG_ID = "SPARE"`, `ALARM_DISABLE = 1`, all alarm enables = 0.

---

## ACD File Naming

```
GeneralPanel_[ProjectName]_v[Major].[Minor].ACD
```

Examples: `GeneralPanel_TCDataLog_v1.0.ACD`, `GeneralPanel_TCDataLog_v1.1.ACD`

**Version increment rules:**
- Minor increment: bug fixes, parameter adjustments, tag additions within existing structure
- Major increment: new project, structural program changes, module config changes, new UDTs

---

## Projects

| # | Name | Status | ACD |
|---|------|--------|-----|
| 1 | TC Data Log | Active | `GeneralPanel_TCDataLog_v1.x.ACD` |
| 2 | TC Temperature Control | Planned | — |

### Project 1 — TC Data Log
Records and visualizes 8 thermocouple temperatures via Ignition historian.
- **I/O:** Slot4_TC CH00–CH03 (TT-101 to TT-104), Slot5_TC CH00–CH03 (TT-105 to TT-108)
- **Unused:** Slots 1, 2, 3, 6 — all disabled
- **Logic:** CM_TC processing only; no control outputs
- **Ignition:** Historian on `CM_AIN[8..15].PV`; trend display screen
- **TC config:** Type J, CONV=2 (pass-through), EULO/EUHI set per application range

---

## Version Control

Maintained in Smartsheet. Every ACD change requires a log entry.

| Sheet | Purpose |
|-------|---------|
| [ACD Version Log](https://app.smartsheet.com/sheets/wPFGFJ2MH99vRfh5rHCX9Vmw4jRhJXc97c9hHrC1) | All ACD revisions |
| [Project Catalog](https://app.smartsheet.com/sheets/pJvHpf2Q56m4mG56pjvwvJpQ5rXfw2gh7r8f2Xv1) | Project registry |
| [IO Assignment — TC Data Log](https://app.smartsheet.com/sheets/gHWF8fpFXvxMGPPQQ9mVhXwFHGw2ghCqwrmVr631) | I/O channel assignments |
| [Panel Hardware Registry](https://app.smartsheet.com/sheets/vfMpw9Rvcm88MjGQM7W8j2JgmghWxV3fpCWXWm81) | Hardware BOM and config |

---

## Reference Files

L5X reference files are stored in `Google Drive > My Drive > Projects > R&D General Panel > reference`.

| File | Contents |
|------|----------|
| CM_AIN.L5X | Analog input UDT |
| CM_DIN.L5X | Discrete input channel UDT |
| CM_AOUT.L5X | Analog output channel UDT |
| CM_SOV.L5X | Solenoid valve UDT |
| CM_ALARMS.L5X | System alarm aggregate UDT |
| CM_XS.L5X | Position switch UDT |
| DateTime_Type.L5X | DateTime UDT |
| IOMAP.L5X | IO_MAP routine reference |
| MainProgram - example.L5X | Full example program structure |

---

## Notes

- IY4 thermocouple modules default °C. Verify unit at the start of every new project — early hardware was configured °F.
- IF8 analog inputs report 0–100% from hardware. EU scaling (PSI, GPM, etc.) is handled in software via `CM_AIN.EULO` / `CM_AIN.EUHI` with `CONV=0`.
- ISA-5.1 tag naming is used for all instruments (e.g., TT-101, PT-201, XV-101).
