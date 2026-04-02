# FPGA-Based Auto-Scaling Digital Voltmeter EEE304-2026

A high-precision digital voltmeter implemented on Basys 3 (Artix-7) FPGA featuring automatic range switching (3.3V/10V) with hysteresis protection and real-time 7-segment display.

---

### Features
- **Auto-Scaling**: Automatically switches between 3.3V (high resolution) and 10V ranges using CD4051 analog multiplexer
- **Dual Mode Operation**: 
  - `SW=0`: Auto mode with hysteresis (switch up at 3.1V, switch down at 2.8V)
  - `SW=1`: Manual mode (forces 10V range)
- **12-bit XADC**: Utilizes Xilinx XADC for precise voltage measurement
- **Fixed-Point Arithmetic**: Optimized scaling using bit-shifts (÷1024) for hardware efficiency
- **7-Segment Display**: XX.YY format with fixed decimal point
- **LED Bar Graph**: Visual indication of ADC input level

---

### Hardware Requirements
- **FPGA Board**: Basys 3 (Artix-7) or compatible Xilinx board
- **Analog Multiplexer**: CD4051BE
- **Voltage Dividers**:
  - 3.3V Range: 1/3 divider network (R=28.5kΩ total)
  - 10V Range: 1/10 divider network (R=94.5kΩ total)
- **Display**: On-board 7-segment display (active low)
- **Input**: DC Voltage source (0-10V max)

---

### Pinout & Connections

| FPGA Pin | Connection | Description |
|----------|------------|-------------|
| `vauxp3` / `vauxn3` | CD4051 Output (Pin 13) | Differential ADC input |
| `pmod` | CD4051 Pin A (Pin 11) | Range select (0=CH0/10V, 1=CH1/3.3V) |
| `sw` | On-board switch | Mode select (0=Auto, 1=Manual 10V) |
| `an[7:0]` | 7-segment anodes | Display control (active low) |
| `seg[6:0]` | 7-segment cathodes | Segment control |
| `LED[15:0]` | On-board LEDs | ADC level bar graph |

**CD4051 Configuration:**
- **CH0 (10V)**: Connect to 1/10 divider output
- **CH1 (3.3V)**: Connect to 1/3 divider output  
- **Pins B & C**: Ground (logic 0)
- **Inhibit**: Ground (active low enable)

---

### Usage Instructions

1. **Connect Input**: Apply 0-10V DC to the resistor divider network
2. **Select Mode**:
   - **Auto Mode** (`SW=0`): System automatically selects optimal range
   - **Manual Mode** (`SW=1`): Forces 10V range regardless of input
3. **Read Display**: 
   - Format: `XX.YY` (e.g., `05.29` = 5.29V)
   - Decimal point fixed at hundreds place
4. **Over-range**: Displays overflow pattern if input exceeds range limits

---

### Calibration Parameters

| Range | Multiplier | Formula | Resolution |
|-------|------------|---------|------------|
| 3.3V | 825,000 | $V = (ADC \times 825000) \gg 10$ | ~0.8mV (ADC) |
| 10V | 2,500,000 | $V = (ADC \times 2500000) \gg 10$ | ~2.4mV (ADC) |

**Thresholds:**
- **Upscale (3.3V→10V)**: ADC > 3847 (~3.1V equivalent)
- **Downscale (10V→3.3V)**: ADC < 1158 (~2.8V equivalent)

---

### Module Hierarchy

```
XADCdemo (Top)
├── xadc_wiz_0 (XADC IP Core)
├── DigitToSeg (7-segment controller)
│   ├── sevensegdecoder (BCD to 7-seg)
│   ├── mux4_4bus (Digit multiplexer)
│   ├── segClkDevider (Scan clock ~5kHz)
│   ├── counter3bit (Digit select)
│   └── decoder_3_8 (Anode driver)
└── FSM Logic (Auto-range state machine)
```

---

### Safety Notes
⚠️ **Do not exceed 12V input** to prevent damage to analog multiplexer  
⚠️ **Verify CD4051 VDD** (3.3V/5V) before connecting  
⚠️ **Differential ADC inputs**: Ensure proper grounding of `vauxn3`

---

### License
This project is provided as-is for educational purposes. Suitable for academic projects and hobbyist electronics applications.
