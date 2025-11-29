📁 PROJECT STRUCTURE
text
CannabisGrowMonitor/
│
├── 📄 CannabisGrowMonitor.ino          # Main sketch file
├── 📄 config.h                          # Configuration & credentials
├── 📄 sensors.h                         # Sensor management
├── 📄 display.h                         # Display & UI functions
├── 📄 network.h                         # WiFi & MQTT handlers
├── 📄 storage.h                         # SPIFFS data logging
├── 📄 calculations.h                    # VPD & environmental math
│
├── 📁 data/
│   └── 📄 wifi_config.json             # Persistent WiFi config
│
├── 📄 platformio.ini                    # PlatformIO config (optional)
├── 📄 README.md                         # Project documentation
└── 📄 HARDWARE_GUIDE.md                 # Wiring & assembly guide
🎯 BEFORE YOU START - CRITICAL CHECKLIST
✅ Hardware Requirements
 ESP32-S3 T-Display (170x320 ST7789 screen) ✓ YOU HAVE THIS

 SCD40 or SCD41 CO2 sensor (I2C address: 0x62)

 BME280 sensor (I2C address: 0x76 or 0x77)

 4x Dupont wires (Female-Female)

 USB-C cable for programming

 Optional: 2000mAh LiPo battery for portable use

✅ Software Requirements
 Arduino IDE 2.3.0+ OR PlatformIO (recommended)

 ESP32 Board Support 3.0.0+

 Libraries (auto-install instructions below)

📦 STEP 1: ARDUINO IDE SETUP (5 MINUTES)
1️⃣ Install ESP32 Board Support
text
1. Open Arduino IDE
2. File → Preferences
3. Additional Board Manager URLs, add:
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
4. Tools → Board → Boards Manager
5. Search "ESP32" by Espressif
6. Install version 3.0.0 or newer
2️⃣ Board Configuration
text
Tools → Board → ESP32 Arduino → ESP32S3 Dev Module

⚙️ EXACT SETTINGS (COPY THESE):
├─ USB CDC On Boot: "Enabled"          ⭐ CRITICAL
├─ USB DFU On Boot: "Disabled"
├─ CPU Frequency: "240MHz (WiFi)"
├─ Flash Mode: "QIO 80MHz"
├─ Flash Size: "16MB (128Mb)"
├─ Partition Scheme: "16M Flash (3MB APP/9.9MB FATFS)"  ⭐ MORE STORAGE
├─ PSRAM: "OPI PSRAM"                  ⭐ CRITICAL FOR GRAPHICS
├─ Upload Mode: "UART0 / Hardware CDC"
├─ Upload Speed: "921600"
└─ USB Mode: "Hardware CDC and JTAG"
3️⃣ Install Libraries
Method A: Library Manager (Easy)

text
Tools → Manage Libraries → Search & Install:

📚 REQUIRED LIBRARIES:
1. "TFT_eSPI" by Bodmer (v2.5.43+)
2. "Adafruit BME280 Library" (v2.2.4+)
3. "Adafruit Unified Sensor" (v1.1.14+)
4. "Sensirion I2C SCD4x" (v0.4.0+)
5. "PubSubClient" by Nick O'Leary (v2.8.0+)
6. "ArduinoJson" by Benoit Blanchon (v7.0.0+)
Method B: Terminal (Fast - 30 seconds)

bash
# Linux/Mac
cd ~/Arduino/libraries/
git clone https://github.com/Bodmer/TFT_eSPI.git
git clone https://github.com/adafruit/Adafruit_BME280_Library.git
git clone https://github.com/adafruit/Adafruit_Sensor.git
git clone https://github.com/Sensirion/arduino-i2c-scd4x.git
git clone https://github.com/knolleary/pubsubclient.git
git clone https://github.com/bblanchon/ArduinoJson.git

# Windows
cd Documents\Arduino\libraries\
# Use GitHub Desktop or download ZIPs manually
⚡ STEP 2: CONFIGURE TFT_eSPI LIBRARY
🚨 CRITICAL STEP - DON'T SKIP!

Find User_Setup_Select.h
Path depends on OS:

text
Windows: Documents\Arduino\libraries\TFT_eSPI\User_Setup_Select.h
Linux:   ~/Arduino/libraries/TFT_eSPI/User_Setup_Select.h
Mac:     ~/Documents/Arduino/libraries/TFT_eSPI/User_Setup_Select.h
Edit User_Setup_Select.h
Comment out line 29:

cpp
// Line 29: COMMENT THIS OUT:
// #include <User_Setup.h>  // ❌ DISABLE DEFAULT

// Add this around line 60:
#include <User_Setups/Setup25_TTGO_T_Display.h>  // ✅ T-Display preset
OR Create Custom Setup (Advanced - Better)
Create: libraries/TFT_eSPI/User_Setups/Setup_T_Display_S3.h

cpp
// ============================================
// 🎨 T-DISPLAY-S3 CUSTOM CONFIG
// ============================================
// Hardware: ESP32-S3 + ST7789V 1.9" IPS
// Resolution: 170x320 pixels
// ============================================

#define USER_SETUP_ID 999

// 📺 DISPLAY DRIVER
#define ST7789_DRIVER      // ST7789V chipset
#define TFT_RGB_ORDER TFT_RGB  // Color byte order

// 📐 DISPLAY RESOLUTION
#define TFT_WIDTH  170
#define TFT_HEIGHT 320

// 📌 PIN CONNECTIONS (T-Display-S3 SPECIFIC)
#define TFT_MOSI 11   // SPI MOSI (Master Out Slave In)
#define TFT_SCLK 12   // SPI Clock
#define TFT_CS    6   // Chip Select (LOW = active)
#define TFT_DC    7   // Data/Command (LOW = command, HIGH = data)
#define TFT_RST   5   // Hardware Reset (LOW = reset, HIGH = run)
#define TFT_BL   38   // Backlight PWM control (0-255)

// 🚫 OPTIONAL PINS (NOT USED)
#define TFT_MISO 13   // Not connected but defined

// 🔤 FONTS TO LOAD (Reduces memory usage)
#define LOAD_GLCD   // Original Adafruit font
#define LOAD_FONT2  // Small 16 pixel high font
#define LOAD_FONT4  // Medium 26 pixel font
#define LOAD_FONT6  // Large 48 pixel font
#define LOAD_FONT7  // 7-segment style
#define LOAD_FONT8  // Large 75 pixel font

#define SMOOTH_FONT // Anti-aliased fonts

// ⚡ SPI SPEED SETTINGS
#define SPI_FREQUENCY       40000000  // 40MHz - Fast writes
#define SPI_READ_FREQUENCY  20000000  // 20MHz - Stable reads
#define SPI_TOUCH_FREQUENCY  2500000  // 2.5MHz - Touch (if used)

// 🎨 COLOR DEPTH
#define TFT_INVERSION_ON    // Required for proper colors on ST7789
Then in User_Setup_Select.h:

cpp
#include <User_Setups/Setup_T_Display_S3.h>  // ✅ YOUR CUSTOM CONFIG
🔌 STEP 3: HARDWARE WIRING
📊 Complete Connection Map
text
╔═══════════════════════════════════════════════════════════╗
║               ESP32-S3 T-DISPLAY PINOUT                   ║
╚═══════════════════════════════════════════════════════════╝

📍 I2C BUS (Shared between sensors):
┌────────────────────────────────────────────┐
│  ESP32 Pin  │  SCD40  │  BME280  │ Color  │
├─────────────┼─────────┼──────────┼────────┤
│  GPIO 43    │   SDA   │   SDA    │  🟡    │ ← Data line
│  GPIO 44    │   SCL   │   SCL    │  🟠    │ ← Clock line
│  3.3V       │   VCC   │   VCC    │  🔴    │ ← Power
│  GND        │   GND   │   GND    │  ⚫    │ ← Ground
└────────────────────────────────────────────┘

🔋 POWER SPECIFICATIONS:
├─ Total Current: ~150mA @ 3.3V
├─ SCD40: 20mA (measurement), 0.5mA (idle)
├─ BME280: 3.6mA (max)
└─ Display: ~100mA (full brightness)

⚠️ IMPORTANT NOTES:
• Both sensors share same I2C bus (parallel connection)
• Pull-up resistors (4.7kΩ) usually on sensor modules
• Keep wires SHORT (<10cm) for I2C stability
• SCD40 needs 30 seconds warm-up after power-on
📸 Visual Wiring Guide
text
┌─────────────────────────────────────────────┐
│         T-DISPLAY-S3 (Top View)             │
│  ┌───────────────────────────────────┐     │
│  │         [170x320 Screen]          │     │
│  └───────────────────────────────────┘     │
│                                             │
│  Left Side Pins:          Right Side Pins: │
│  ├─ GPIO43 (SDA) 🟡       GPIO16 ─┤        │
│  ├─ GPIO44 (SCL) 🟠       GPIO21 ─┤        │
│  ├─ 3.3V        🔴        GND    ─┤        │
│  └─ GND         ⚫        5V     ─┤        │
└─────────────────────────────────────────────┘
         │         │
         │         └─────────┐
         │                   │
    ┌────▼────┐         ┌────▼────┐
    │  SCD40  │         │ BME280  │
    │ CO2     │         │Temp/Hum │
    │ 0x62    │         │ 0x76    │
    └─────────┘         └─────────┘
