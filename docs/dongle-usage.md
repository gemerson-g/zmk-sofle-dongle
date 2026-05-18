# Dongle usage (receiver mode)

This keyboard is configured to run in **dongle mode**:

- The **dongle/receiver** is the ZMK "central" device.
- The **left + right halves** are ZMK "peripherals".
- Keystrokes are sent from the halves to the dongle first, then forwarded to your computer via **USB** or **Bluetooth**.

## What changes in dongle mode

- The halves do **not** directly connect to the computer; they connect to the dongle.
- After the dongle is configured, power usage on the left half is reduced (per the vendor documentation) and the dongle can operate at a higher report rate.

## Hardware controls on the dongle

- **Battery toggle switch (ON/OFF)**: disconnects the internal battery.
  - If you always use the dongle connected to USB, you can leave the battery switch **OFF** to reduce battery wear.
- **Reset button**:
  - With the dongle connected via USB, **double-press reset** to enter bootloader mode.
  - A USB drive will appear on your computer. Copy the new firmware file (`.uf2`) onto it.

## OLED display

The dongle has a small OLED display. It shows:
- Battery levels for the dongle, left half, and right half
- Current connection state (USB or Bluetooth)
- Active layer

> The macOS modifier logo can be enabled by uncommenting `CONFIG_ZMK_DONGLE_DISPLAY_MAC_MODIFIERS=y` in `config/eyelash_sofle_central_dongle.conf`.

## Suggested placement

- Use the included stand to angle the dongle on the desk.
- Optionally mount it on top of your monitor (the dongle includes a magnet; a metal patch can be attached to the monitor).

## First-time setup / re-pairing

See the [README](../README.md#how-to-flash-firmware) for the full reset and re-pairing sequence.

If pairing is broken or devices won't connect, always do the full sequence: flash `settings_reset` to all three devices first, then flash the normal firmware to each.

## Live keymap changes (ZMK Studio)

Connect the **dongle** via USB and follow the [ZMK Studio guide](zmk-studio.md).

## Troubleshooting

**Halves won't pair with the dongle after flashing**
- Make sure you flashed `settings_reset` to all three devices before flashing the normal firmware.
- Power cycle all devices after flashing.

**Dongle connects to the computer but keypresses aren't registering**
- Check that both halves are powered on.
- Move the halves closer to the dongle; BLE range can be limited by desk obstacles.

**OLED is blank**
- The display turns off when the keyboard is idle. Press any key to wake it.
- If it stays blank after activity, reflash the dongle firmware.

**One half is not responding**
- Try power cycling that half (flip its power switch off and back on).
- If it still doesn't respond, flash `settings_reset` to that half only, then reflash its normal firmware.

---

<details>
<summary>Switching from dongle mode to direct keyboard mode</summary>

In direct mode the **left half** connects to your computer via USB or Bluetooth and acts as the central device. The right half connects wirelessly to the left half. The dongle is not used.

**Your keymap is fully reusable** - you will copy it as-is into the new config file.

---

### Overview of changes

| What | Why |
|------|-----|
| Create new shield files in `boards/shields/eyelash_sofle/` | Register `eyelash_sofle_left` and `eyelash_sofle_right` as valid shields |
| Update `Kconfig.shield` and `Kconfig.defconfig` | Tell ZMK the left half is the central device |
| Copy your keymap to a new file name | ZMK looks for config files that match the shield name exactly |
| Update `build.yaml` | Build the new shields instead of the dongle shields |

---

### Step 1 - Create `boards/shields/eyelash_sofle/eyelash_sofle_left.overlay`

This defines the GPIO pins for the left half. Copy this exactly:

```c
#include "eyelash_sofle.dtsi"

&kscan0 {
    compatible = "zmk,kscan-gpio-matrix";
    row-gpios
        = <&gpio0 19 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio0 8 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio0 12 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio0 11 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio1 9 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        ;
    col-gpios
        = <&gpio0 3 GPIO_ACTIVE_HIGH>
        , <&gpio0 28 GPIO_ACTIVE_HIGH>
        , <&gpio0 30 GPIO_ACTIVE_HIGH>
        , <&gpio0 21 GPIO_ACTIVE_HIGH>
        , <&gpio0 23 GPIO_ACTIVE_HIGH>
        , <&gpio0 22 GPIO_ACTIVE_HIGH>
        ;
};

&left_encoder {
    status = "okay";
};

nice_view_spi: &spi0 {
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 6 GPIO_ACTIVE_HIGH>;
};
```

---

### Step 2 - Create `boards/shields/eyelash_sofle/eyelash_sofle_right.overlay`

Same idea for the right half:

```c
#include "eyelash_sofle.dtsi"

&kscan0 {
    compatible = "zmk,kscan-gpio-matrix";
    row-gpios
        = <&gpio0 19 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio0 8 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio0 12 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio0 11 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        , <&gpio1 9 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
        ;
    col-gpios
        = <&gpio0 22 GPIO_ACTIVE_HIGH>
        , <&gpio0 29 GPIO_ACTIVE_HIGH>
        , <&gpio0 3 GPIO_ACTIVE_HIGH>
        , <&gpio0 28 GPIO_ACTIVE_HIGH>
        , <&gpio0 30 GPIO_ACTIVE_HIGH>
        , <&gpio0 21 GPIO_ACTIVE_HIGH>
        , <&gpio0 23 GPIO_ACTIVE_HIGH>
        ;
};

&default_transform {
    col-offset = <7>;
};

nice_view_spi: &spi0 {
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 6 GPIO_ACTIVE_HIGH>;
};
```

---

### Step 3 - Create `boards/shields/eyelash_sofle/eyelash_sofle_left.conf`

This enables features for the left half (central). Only 1 peripheral now (right half), not 2 like in dongle mode:

```
CONFIG_ZMK_EXT_POWER=y
CONFIG_ZMK_BLE_PASSKEY_ENTRY=n
CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=n
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=n
CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y
CONFIG_ZMK_MOUSE=y
CONFIG_ZMK_POINTING=y
CONFIG_BT_MAX_CONN=4
CONFIG_BT_MAX_PAIRED=4
CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=1
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
CONFIG_SPI=y
CONFIG_ZMK_BACKLIGHT=y
CONFIG_ZMK_BACKLIGHT_BRT_START=100
CONFIG_ZMK_BACKLIGHT_ON_START=y
CONFIG_ZMK_BACKLIGHT_AUTO_OFF_IDLE=n
CONFIG_WS2812_STRIP=y
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_ZMK_RGB_UNDERGLOW_EXT_POWER=y
CONFIG_ZMK_RGB_UNDERGLOW_ON_START=y
CONFIG_ZMK_RGB_UNDERGLOW_BRT_MAX=90
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_USB=y
CONFIG_ZMK_RGB_UNDERGLOW_HUE_START=160
CONFIG_ZMK_RGB_UNDERGLOW_EFF_START=3
```

---

### Step 4 - Create `boards/shields/eyelash_sofle/eyelash_sofle_right.conf`

The right half config is unchanged from dongle mode. Create this file with the same content as `eyelash_sofle_peripheral_right.conf`.

---

### Step 5 - Update `boards/shields/eyelash_sofle/Kconfig.shield`

Add two lines at the bottom to register the new shield names:

```
config SHIELD_EYELASH_SOFLE_LEFT
    def_bool $(shields_list_contains,eyelash_sofle_left)

config SHIELD_EYELASH_SOFLE_RIGHT
    def_bool $(shields_list_contains,eyelash_sofle_right)
```

---

### Step 6 - Update `boards/shields/eyelash_sofle/Kconfig.defconfig`

Add this block at the end of the file. It tells ZMK that `eyelash_sofle_left` is the central device and enables split mode for both halves:

```
if SHIELD_EYELASH_SOFLE_LEFT

config ZMK_KEYBOARD_NAME
    default "E_Sofle"

config ZMK_SPLIT_ROLE_CENTRAL
    default y

endif

if SHIELD_EYELASH_SOFLE_LEFT || SHIELD_EYELASH_SOFLE_RIGHT

config ZMK_SPLIT
    default y

if ZMK_BACKLIGHT

config PWM
    default y

config LED_PWM
    default y

endif # ZMK_BACKLIGHT

endif
```

---

### Step 7 - Copy your keymap

Create `config/eyelash_sofle_left.keymap` by copying the full content of your current `config/eyelash_sofle_central_dongle.keymap`. Nothing inside needs to change - the layers, bindings, and behaviors are identical.

---

### Step 8 - Create `config/eyelash_sofle_left.conf`

Create this file based on your current `config/eyelash_sofle_central_dongle.conf`, removing the dongle display lines:

```
# Sleep after one hour
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000
CONFIG_ZMK_SLEEP=y

# Encoder
CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y

# Mouse
CONFIG_ZMK_MOUSE=y

CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=n
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=8
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=8
```

---

### Step 9 - Update `build.yaml`

Replace the current dongle entries with the new ones (keep `settings_reset`):

```yaml
---
include:
  - board: nice_nano_v2
    shield: eyelash_sofle_left
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO=y -DCONFIG_ZMK_STUDIO_LOCKING=n
    artifact-name: eyelash_sofle_left
  - board: nice_nano_v2
    shield: eyelash_sofle_right
    artifact-name: eyelash_sofle_right
  - board: nice_nano_v2
    shield: settings_reset
```

---

### Step 10 - Build and flash

1. Commit your changes. GitHub Actions will build the firmware automatically.
2. Download the artifact from the Actions run.
3. Flash `settings_reset` to **both halves**.
4. Flash `eyelash_sofle_left` firmware to the **left half**.
5. Flash `eyelash_sofle_right` firmware to the **right half**.
6. Power on both halves. They will pair automatically.

> To go back to dongle mode at any time, revert `build.yaml` to the original entries and flash the dongle firmware again (plus `settings_reset` on all three devices).

</details>
