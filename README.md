# ESP32 Home Assistant Touchscreen Display

A compact ESP32 touchscreen that lives on your desk and shows real-time home telemetry without opening Home Assistant.

![ESP32 Display](images/IMG_4459.jpeg)

## 🎯 Goal

Create a dedicated desktop display providing at-a-glance access to critical smart home information, integrated directly with Home Assistant via ESPHome.

## ✨ Features

The display shows:
- **Current time & date** - Always visible clock display
- **Real-time ComEd electricity pricing** - 5-minute pricing updates
- **Indoor office temperature** - From SmartThings temperature sensor
- **Outdoor "feels like" temperature** - Via OpenWeatherMap integration
- **HVAC action state** - Current heating/cooling status
- **Door status monitoring** - Front, garage, and patio doors with color-coded status
- **Touch-to-toggle backlight** - Tap anywhere on screen to toggle display backlight
- **Onboard LED control** - LED exposed to Home Assistant for automations

## 🛠️ Hardware

**Board:** ESP32 2.4" TFT Touchscreen (ILI9341 + XPT2046)

You can grab the same board here: [https://amzn.to/4cBAFZ0](https://amzn.to/4cBAFZ0)

> **Disclosure:** I'm an Amazon affiliate, so I may earn from qualifying purchases.

### Specifications
- **Display:** 2.4" TFT LCD (320x240)
- **Display Controller:** ILI9341
- **Touch Controller:** XPT2046
- **Microcontroller:** ESP32-WROOM-32
- **Connection:** SPI interface
- **Backlight:** PWM-controlled LED
- **Onboard LED:** GPIO4

## 🏗️ Architecture

### Stack
- **ESPHome** - Device firmware and configuration
- **Home Assistant API** - Native integration for entity imports
- **OpenWeatherMap** - Weather data integration
- **SmartThings** - Temperature sensor
- **ComEd 5-minute pricing sensor** - Real-time electricity rates
- **Binary contact sensors** - Door status monitoring

Everything is pulled live from Home Assistant via the native ESPHome API integration.

**No MQTT. No polling hacks. Just direct entity imports.**

## 🚀 Deployment to Home Assistant

### Prerequisites

Before you begin, ensure you have:
1. **Home Assistant** installed and running
2. **ESPHome** add-on or dashboard installed in Home Assistant
3. The ESP32 board connected via USB or available on your network
4. The following Home Assistant entities configured:
   - `sensor.comed_5_minute_price` - ComEd pricing sensor
   - `sensor.openweathermap_feels_like_temperature` - OpenWeatherMap integration
   - `sensor.smartthings_button_temperature` - SmartThings temperature sensor
   - `climate.great_room` - HVAC/thermostat entity with `hvac_action` attribute
   - `binary_sensor.front_entrance_contact` - Front door sensor
   - `binary_sensor.garage_entrance_contact` - Garage door sensor
   - `binary_sensor.patio_entrance_contact` - Patio door sensor

### Font Files Required

Download and place these font files in the same directory as `esp-display.yml`:
- `Roboto-Bold.ttf`
- `Roboto-Medium.ttf`
- `Roboto-Regular.ttf`

You can download Roboto fonts from [Google Fonts](https://fonts.google.com/specimen/Roboto).

### Setup Instructions

#### 1. Configure Secrets

Copy the example secrets file and fill in your credentials:

```bash
cp secrets.example.yaml secrets.yaml
```

Edit `secrets.yaml` with your information:

```yaml
api_encryption_key: "your-32-character-api-key-here"
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
ota_password: "YourOTAPassword"
```

> **Note:** The `secrets.yaml` file is in `.gitignore` to protect your credentials from being committed to version control.

#### 2. Customize Entity IDs (Optional)

If your Home Assistant entity IDs differ from the defaults, edit `esp-display.yml` and update the `entity_id` values in the sensor, text_sensor, and binary_sensor sections to match your entities.

#### 3. Install to ESP32

##### Option A: Using ESPHome Dashboard (Recommended)

1. Open the ESPHome dashboard in Home Assistant (Settings → Add-ons → ESPHome → Open Web UI)
2. Click **"+ NEW DEVICE"**
3. Click **"SKIP"** (we'll use the existing configuration)
4. Upload the `esp-display.yml` file
5. Upload the `secrets.yaml` file
6. Upload the three Roboto font files
7. Click **"INSTALL"**
8. Choose your installation method:
   - **Plug into this computer** (for first-time USB install)
   - **Wirelessly** (if already configured and on network)

##### Option B: Using ESPHome CLI

```bash
# Install/compile and upload via USB
esphome run esp-display.yml

# Or upload wirelessly (after initial USB install)
esphome upload esp-display.yml
```

#### 4. Add to Home Assistant

After successful installation:

1. The device should be automatically discovered in Home Assistant
2. Go to **Settings → Devices & Services → Integrations**
3. Look for the ESPHome integration notification
4. Click **"CONFIGURE"** and enter the API encryption key from your `secrets.yaml`
5. The device will be added with entities:
   - `light.display_backlight` - Control display backlight
   - `light.board_led` - Control onboard LED
   - `button.restart` - Restart the device
   - `sensor.wifi_signal` - WiFi signal strength
   - `sensor.uptime` - Device uptime in hours

### Customization

#### Modify Display Layout

Edit the `display:` section in `esp-display.yml` to customize what's shown and how it's arranged. The lambda function uses the display's coordinate system (0,0 is top-left).

#### Change Update Interval

By default, the display updates every 5 seconds. Modify the `update_interval` parameter:

```yaml
display:
  - platform: ili9xxx
    update_interval: 10s  # Change to desired interval
```

#### Adjust Touch Calibration

If touch input isn't accurate, modify the calibration values in the `touchscreen:` section:

```yaml
touchscreen:
  - platform: xpt2046
    calibration:
      x_min: 200    # Adjust these values
      x_max: 3900
      y_min: 200
      y_max: 3900
```

## 🔧 Troubleshooting

### Device Won't Connect to WiFi
- Verify credentials in `secrets.yaml`
- Check that your WiFi network is 2.4GHz (ESP32 doesn't support 5GHz)
- Look for the fallback hotspot "Esphome05 Fallback Hotspot" and connect to check logs

### Display Shows Nothing
- Verify the fonts are in the same directory as the YAML file
- Check ESPHome logs for compilation errors
- Ensure the display pins are correctly configured for your board

### Entities Show "Unknown" or No Data
- Verify the entity IDs exist in Home Assistant
- Check that the entities are available (not "unavailable" or "unknown")
- Review the ESPHome logs for API connection issues

### Touch Not Responding
- Check the calibration values
- Verify interrupt_pin GPIO36 is not being used by another component
- Test with the light toggle functionality

## 📝 License

This project is Unlicense licensed   - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with [ESPHome](https://esphome.io/)
- Integrated with [Home Assistant](https://www.home-assistant.io/)
- Fonts from [Google Fonts](https://fonts.google.com/)
