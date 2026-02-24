This is my context file, please read this first.

# Ham Radio Operator Context File
*Paste this at the start of a new Claude conversation for continuity.*

---

## Operator Profile

| Field | Detail |
|-------|--------|
| **Ham Callsign** | KC1KCE |
| **GMRS Callsign** | WRNC678 |
| **Home Location** | Southwest of Sapulpa, OK (Creek County / Tulsa metro area) |
| **Nearest Major Airport** | KRVS — Tulsa Riverside (5 mi S of Tulsa, ~15 mi NE of home) |
| **Also nearby** | KTUL — Tulsa International (~20 mi NE of home) |
| **Travel** | Frequent work travel; home setup not always accessible |

---

## Radio Inventory

### Handheld Radios (HTs)

| Radio | Modes | Notes / Current Focus |
|-------|-------|----------------------|
| **BridgeCom Maverick** (= AnyTone AT-D890UV) | VHF/UHF, DMR, Analog, Airband RX | Current primary focus. Programmed via AnyTone CPS. Airband zone set up and working. Has built-in satellite tracking. |
| **Yaesu FT3DR** | VHF/UHF, C4FM/Fusion, APRS | Dual-band, touch screen. Programmed via RT Systems. |
| **Kenwood TH-D75A** | VHF/UHF, D-STAR, APRS, HF RX | Triband. High-end HT. D-STAR capable. |
| **Radioddity GD-88** | VHF/UHF, DMR, Analog | DMR/analog dual band. |
| **BTech UV Pro** | VHF/UHF, Analog | Wide RX coverage. |

### Base / Mobile Radios

| Radio | Location | Modes | Notes |
|-------|----------|-------|-------|
| **Yaesu FTM-300D** | Home base | VHF/UHF, C4FM/Fusion, APRS | Primary home ham station. Dual band mobile used as base. |
| **BTech GMRS 50 Pro** | Home | GMRS | Home GMRS base. Callsign WRNC678. |
| **Radioddity DB-25D** | Jeep (mobile) | VHF/UHF, DMR, Analog | True mobile radio. Dual band. |
| **Icom IC-7300** | Home shack | HF, 50 MHz | Primary HF transceiver. SDR-based receiver architecture. |
| **Xiegu G90** | Portable / field | HF, 0.5–30 MHz, 20W | Used for Parks on the Air (POTA), portable ops, field use. Built-in antenna tuner. |

---

## BridgeCom Maverick — Programming Notes

- **Platform:** AnyTone AT-D890UV (same firmware/CPS family as D878UV series)
- **Software:** AnyTone CPS (Customer Programming Software)
- **Zone structure:** Channels must be added to both the master channel list AND assigned to a zone. Scan lists for each zone are configured in the same location as zone channel assignment — easy once you know where it is.
- Has built-in satellite tracking functionality (not yet fully tested)
- **Airband RX:** Supported (108–137 MHz, AM receive only). Airband zone created and populated.
- **Frequency format used in this codeplug:** 8-digit Hz format (e.g., 121.500 MHz = `12150000`)

### Airband Zone — Programmed Frequencies

#### General Aviation (National)
| MHz | Description |
|-----|-------------|
| 121.500 | Civil Emergency / Guard |
| 122.900 | Multicom — CTAF (no tower/UNICOM) |
| 122.950 | UNICOM at towered airports (FBO) |
| 122.750 | Air-to-Air |
| 122.850 | Air-to-Air Chat 1 |
| 123.450 | Air-to-Air Chat 2 |
| 123.025 | Helicopter common |
| 123.100 | Search & Rescue / Civil Air Patrol |

#### Tulsa Area — Local Frequencies
| MHz | Description |
|-----|-------------|
| 126.500 | KRVS (Riverside) ATIS |
| 121.700 | KRVS Ground Control |
| 120.300 | KRVS Tower (primary) |
| 119.200 | KRVS Tower (secondary — when advised) |
| 124.500 | KRVS Clearance Delivery |
| 119.850 | Tulsa Approach/Departure (N/E sector) |
| 134.700 | Tulsa Approach/Departure (S/W sector — home side) |
| 124.900 | KTUL (Tulsa Intl) ATIS |
| 118.700 | KTUL Tower (Rwy 18R/36L) |
| 121.900 | KTUL Ground |

---

## Yaesu FT3DR — Notes

- Dual band VHF/UHF, C4FM/Fusion, APRS
- Touch screen display — toggle display by long-pressing the frequency on touchscreen (NOT the DISP button)
- Programmed via RT Systems software

---

## Local Area Context (Sapulpa / Tulsa Metro)

- **Home:** SW of Sapulpa, OK — Creek County
- **Tulsa Class C airspace** covers the area; KRVS Class D is closer
- **134.700 MHz** Tulsa Approach covers the south/southwest sector (directly over home area)
- **Medical helicopters** (Saint Francis, Saint John) common in area — often on 123.025

---

## Operating Interests

- Airband monitoring (primary current interest with Maverick)
- DMR (BrandMeister / TGIF networks)
- APRS / GPS tracking
- HF — home station IC-7300
- Portable / field HF — Xiegu G90
- Parks on the Air (POTA)
- GMRS — home and mobile (callsign WRNC678)
- Satellite receive (learning / experimenting with Maverick)

---

*Generated, initially, with Claude (claude.ai / VSCode) — paste at session start for context continuity*
*Maintained, by me, Grant*
