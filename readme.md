# Dactyl Manuform 4x6 — ZMK Firmware Config

A beginner-friendly guide to building, flashing, and using the **Dactyl Manuform 4x6** split ergonomic keyboard running [ZMK Firmware](https://zmk.dev) on **nice!nano v2** controllers.

---

## Table of Contents

1. [What Is This Keyboard?](#what-is-this-keyboard)
2. [Parts List](#parts-list)
3. [How the Keyboard Is Wired](#how-the-keyboard-is-wired)
4. [Understanding the Layout](#understanding-the-layout)
5. [Layer Guide](#layer-guide)
6. [How Layer-Tap Works](#how-layer-tap-works)
7. [Bluetooth & Connectivity](#bluetooth--connectivity)
8. [Building the Firmware](#building-the-firmware)
9. [Flashing the Firmware](#flashing-the-firmware)
10. [Troubleshooting](#troubleshooting)
11. [Customizing Your Keymap](#customizing-your-keymap)
12. [File Structure Reference](#file-structure-reference)
13. [Resources](#resources)

---

## What Is This Keyboard?

The Dactyl Manuform is a **split ergonomic keyboard** — meaning it comes in two separate halves (left and right) that you position independently for comfort. The "4x6" in the name refers to the main key grid: **4 rows × 6 columns per side**, plus a **5-key thumb cluster** on each half.

**Total keys:** 46 (36 main keys + 10 thumb keys)

This is a **wireless** build. Both halves communicate over Bluetooth. The **left half** is the "central" side — it's the one your computer actually connects to. The right half talks to the left half, which relays everything to your computer.

### Why Split Ergonomic?

- Your hands rest at shoulder width instead of cramped together
- The curved (concave) shape follows your fingers' natural reach
- Thumb clusters put your strongest digit to work on important keys like Space, Enter, and layer switches
- Fewer keys means less finger travel — layers give you full access to everything

---

## Parts List

| Component | Qty | Notes |
|-----------|-----|-------|
| Dactyl Manuform 4x6 case (left + right) | 1 set | 3D printed; many STL files available online |
| nice!nano v2 controllers | 2 | One per half; provides Bluetooth + battery support |
| 1N4148 diodes | 46 | One per switch; SOD-123 (SMD) or through-hole |
| MX-compatible key switches | 46 | Cherry MX, Gateron, etc. — your preference |
| Keycaps | 46 | Any MX-compatible set |
| LiPo batteries (3.7V) | 2 | 301230 or similar; 110mAh+ recommended |
| Wiring | — | Enameled copper wire or flexible silicone wire |
| Hot glue / adhesive | — | To secure controllers and batteries inside the case |
| Micro USB or USB-C cable | 1 | For flashing and charging (nice!nano v2 uses USB-C) |

---

## How the Keyboard Is Wired

This build uses a **matrix wiring** scheme based on the Corne keyboard's pin layout, with an expanded thumb cluster.

### Matrix Basics

Instead of wiring each of the 46 keys to its own pin (impossible with a small microcontroller), keys are arranged in a **grid of rows and columns**. Each key sits at the intersection of one row wire and one column wire. A **diode** on each key ensures the controller can tell exactly which key was pressed without "ghosting" (false signals).

### Pin Mapping

**Rows** (active high, active on both halves):

| Row | nice!nano Pin |
|-----|---------------|
| Row 0 | Pin 4 (GPIO 4) |
| Row 1 | Pin 5 (GPIO 5) |
| Row 2 | Pin 6 (GPIO 6) |
| Row 3 | Pin 7 (GPIO 7) |

**Columns** (active high, active on both halves — same pins, mirrored):

| Column | nice!nano Pin |
|--------|---------------|
| Col 0 | Pin 21 |
| Col 1 | Pin 20 |
| Col 2 | Pin 19 |
| Col 3 | Pin 18 |
| Col 4 | Pin 15 |
| Col 5 | Pin 14 |

The **diode direction** is `col2row` — current flows from column to row through each diode.

### Why "Corne-Compatible"?

The Corne (CRKBD) is a popular 3×6 split keyboard. This Dactyl Manuform config reuses the same row/column GPIO pins, but adds a 4th row for the extra thumb keys. If you've built a Corne before, the wiring will feel familiar.

---

## Understanding the Layout

Here's the physical arrangement of all 46 keys. The keyboard has **4 layers**, and you switch between them using the thumb keys.

### Physical Key Positions

```
LEFT HALF                                    RIGHT HALF
┌─────┬─────┬─────┬─────┬─────┬─────┐      ┌─────┬─────┬─────┬─────┬─────┬─────┐
│ R0C0│ R0C1│ R0C2│ R0C3│ R0C4│ R0C5│      │ R0C6│ R0C7│ R0C8│ R0C9│R0C10│R0C11│
├─────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼─────┤
│ R1C0│ R1C1│ R1C2│ R1C3│ R1C4│ R1C5│      │ R1C6│ R1C7│ R1C8│ R1C9│R1C10│R1C11│
├─────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼─────┤
│ R2C0│ R2C1│ R2C2│ R2C3│ R2C4│ R2C5│      │ R2C6│ R2C7│ R2C8│ R2C9│R2C10│R2C11│
└─────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘      └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴─────┘
         │ R3C1│ R3C2│ R3C3│ R3C4│ R3C5│  │ R3C6│ R3C7│ R3C8│ R3C9│R3C10│
         └─────┴─────┴─────┴─────┴─────┘  └─────┴─────┴─────┴─────┴─────┘
                      THUMB CLUSTERS
```

---

## Layer Guide

This firmware has **4 layers**. You're always on Layer 0 by default. Hold a thumb key to temporarily activate another layer.

### Layer 0 — Default (QWERTY)

Your everyday typing layer. Stays close to a standard keyboard layout so the transition is smooth.

```
┌──────┬─────┬─────┬─────┬─────┬─────┐      ┌─────┬─────┬─────┬─────┬─────┬──────┐
│ ESC  │  Q  │  W  │  E  │  R  │  T  │      │  Y  │  U  │  I  │  O  │  P  │ BSPC │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│ TAB  │  A  │  S  │  D  │  F  │  G  │      │  H  │  J  │  K  │  L  │  ;  │  '   │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│ SHFT │  Z  │  X  │  C  │  V  │  B  │      │  N  │  M  │  ,  │  .  │  /  │ SHFT │
└──────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘      └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──────┘
          │ FN  │ ALT │ GUI │LWR/ │LCTL│  │RCTL│RSE/ │ ALT │  [  │  ]  │
          │(L3) │     │     │SPACE│    │  │    │ENTER│     │     │     │
          └─────┴─────┴─────┴─────┴────┘  └────┴─────┴─────┴─────┴─────┘
```

**Key things to notice:**
- **ESC** is top-left (where \` usually is on a full keyboard)
- **Backspace** is top-right (easy reach for your right pinky)
- **Shift** keys are on the outer edges of the bottom row, one per side
- **`[` and `]`** are on the right thumb cluster for quick access
- The two most important thumb keys use **layer-tap** (explained below):
  - Left thumb: hold for **Lower layer**, tap for **Space**
  - Right thumb: hold for **Raise layer**, tap for **Enter**

---

### Layer 1 — Lower (Hold Left Thumb Space)

Activated by **holding** the left thumb Space key. Focused on **function keys** and **navigation**.

```
┌──────┬─────┬─────┬─────┬─────┬─────┐      ┌─────┬─────┬─────┬─────┬─────┬──────┐
│  ~   │ F1  │ F2  │ F3  │ F4  │ F5  │      │ F6  │ F7  │ F8  │ F9  │ F10 │ DEL  │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │ F11 │ F12 │     │     │     │      │HOME │PG DN│PG UP│ END │     │  =   │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │     │     │     │     │     │      │  ←  │  ↓  │  ↑  │  →  │     │      │
└──────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘      └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──────┘
          │     │     │     │█████│    │  │    │     │     │     │     │
          └─────┴─────┴─────┴─────┴────┘  └────┴─────┴─────┴─────┴─────┘
```

**What you get here:**
- **F1–F12** function keys across the top two rows on the left side
- **Arrow keys** (←↓↑→) on the right side in a Vim-like arrangement (HJKL positions)
- **Home / End / Page Up / Page Down** for document navigation
- **Delete** replaces Backspace on the top right
- **Tilde (~)** and **Equals (=)** for quick access
- Blank keys (marked with spaces) pass through to Layer 0

---

### Layer 2 — Raise (Hold Right Thumb Enter)

Activated by **holding** the right thumb Enter key. Focused on **symbols and special characters**.

```
┌──────┬─────┬─────┬─────┬─────┬─────┐      ┌─────┬─────┬─────┬─────┬─────┬──────┐
│  ~   │  !  │  @  │  #  │  $  │  %  │      │  ^  │  &  │  *  │  (  │  )  │ BSPC │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │     │     │     │     │     │      │  -  │  =  │  [  │  ]  │  \  │  `   │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │     │     │     │     │     │      │  _  │  +  │  {  │  }  │  |  │  ~   │
└──────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘      └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──────┘
          │     │     │     │     │    │  │    │█████│     │     │     │
          └─────┴─────┴─────┴─────┴────┘  └────┴─────┴─────┴─────┴─────┘
```

**What you get here:**
- **Number row symbols** (`!@#$%^&*()`) across the top — no need for Shift+number
- **Brackets in pairs:** `[ ]` and `{ }` side by side on the right
- **Math/programming operators:** `-`, `=`, `_`, `+`
- **Pipe `|`** and **backslash `\`** for terminal and programming
- **Backtick `` ` ``** and **tilde `~`** for code and terminal use

---

### Layer 3 — Function / Bluetooth (Hold Left Thumb FN)

Activated by **holding** the FN key (leftmost thumb key). Used for **Bluetooth device management**.

```
┌──────┬─────┬─────┬─────┬─────┬─────┐      ┌─────┬─────┬─────┬─────┬─────┬──────┐
│BT CLR│ BT1 │ BT2 │ BT3 │ BT4 │ BT5 │      │     │     │     │     │     │      │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │     │     │     │     │     │      │     │     │     │     │     │      │
├──────┼─────┼─────┼─────┼─────┼─────┤      ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │     │     │     │     │     │      │     │     │     │     │     │      │
└──────┴──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘      └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┴──────┘
          │█████│     │     │     │    │  │    │     │     │     │     │
          └─────┴─────┴─────┴─────┴────┘  └────┴─────┴─────┴─────┴─────┘
```

**Bluetooth controls:**
- **BT CLR** — Clear the current Bluetooth profile (unpairs the current device)
- **BT1–BT5** — Switch between up to 5 paired Bluetooth devices

This lets you pair the keyboard with multiple devices (e.g., your laptop, tablet, and phone) and switch between them instantly.

---

## How Layer-Tap Works

Layer-tap is the secret sauce of small keyboards. Two of your thumb keys use this behavior:

| Key | Tap (quick press) | Hold | 
|-----|------------------|------|
| Left thumb (inner) | Space | Activate Layer 1 (Lower) |
| Right thumb (inner) | Enter | Activate Layer 2 (Raise) |

**The timing rule:** If you press and release the key in under **200 milliseconds**, it registers as a tap (Space or Enter). If you hold it for 200ms or longer, it activates the layer instead.

**In practice:** You'll naturally tap for Space/Enter during normal typing, and hold when you need symbols or navigation. It becomes second nature surprisingly fast.

**Tip:** If you find yourself accidentally triggering layers when trying to type Space, or accidentally typing Space when trying to access a layer, you can adjust the `tapping-term-ms` value in the keymap file. A lower value (e.g., 150) makes taps easier; a higher value (e.g., 250) makes holds easier.

---

## Bluetooth & Connectivity

### How Split Bluetooth Works

```
   ┌─────────────┐     Bluetooth      ┌─────────────┐     Bluetooth      ┌────────────┐
   │  Right Half  │ ──────────────── → │  Left Half   │ ──────────────── → │  Computer  │
   │ (peripheral) │                    │  (central)   │                    │            │
   └─────────────┘                     └─────────────┘                     └────────────┘
```

The **left half** is the "central" device — it's the one your computer sees as a Bluetooth keyboard. The right half talks to the left half wirelessly. You only pair the **left half** to your computer.

### Pairing Your Keyboard

1. Turn on both halves (plug in or flip battery switch if your case has one)
2. The left half should enter pairing mode automatically the first time
3. On your computer, open Bluetooth settings and look for **"Dactyl 4x6"**
4. Select it to pair
5. Start typing!

### Switching Between Devices

You can pair up to **5 different devices** using the Bluetooth profiles:

1. Hold **FN** (leftmost thumb key on the left hand)
2. Press **BT1** through **BT5** (the Q through T keys) to select a profile
3. If the profile is empty, the keyboard enters pairing mode for a new device
4. If the profile already has a paired device, it switches to that device

### Clearing a Bluetooth Profile

If you're having connection issues or want to re-pair a device:

1. Hold **FN**
2. Select the problematic profile (BT1–BT5)
3. Press **BT CLR** (the ESC key position) while still holding FN
4. The profile is now cleared — re-pair by selecting it and pairing from your computer

---

## Building the Firmware

You have two options: build in the cloud with GitHub Actions (recommended for beginners) or build locally.

### Option A: GitHub Actions (Recommended)

This is the easiest method — GitHub compiles the firmware for you.

1. **Fork this repo** to your own GitHub account
2. Make any keymap changes you want (edit `config/dactyl_manuform_4x6.keymap`)
3. **Push your changes** to GitHub
4. Go to the **Actions** tab in your GitHub repo
5. The build should run automatically; if not, trigger it manually
6. Once the build completes (green checkmark), click on the build run
7. Download the **firmware artifact** — it will be a `.zip` containing two `.uf2` files:
   - `dactyl_manuform_4x6_left-nice_nano_v2-zmk.uf2`
   - `dactyl_manuform_4x6_right-nice_nano_v2-zmk.uf2`

### Option B: Build Locally

If you prefer to build on your own machine, follow the [ZMK development setup guide](https://zmk.dev/docs/development/setup). Then:

```bash
west build -d build/left -b nice_nano_v2 -- -DSHIELD=dactyl_manuform_4x6_left
west build -d build/right -b nice_nano_v2 -- -DSHIELD=dactyl_manuform_4x6_right
```

The `.uf2` files will be in `build/left/zephyr/zmk.uf2` and `build/right/zephyr/zmk.uf2`.

---

## Flashing the Firmware

Flashing means copying the compiled firmware onto your nice!nano controllers. You need to flash **each half separately** with its own firmware file.

### Step-by-Step

1. **Connect** the left nice!nano to your computer via USB-C
2. **Enter bootloader mode** by double-pressing the reset button on the nice!nano
   - The timing can be tricky — press reset twice quickly (within ~500ms)
   - A new drive called **"NICENANO"** should appear on your computer (like a USB flash drive)
3. **Copy** the `dactyl_manuform_4x6_left-nice_nano_v2-zmk.uf2` file onto the NICENANO drive
4. The controller will automatically reboot and the drive will disappear — **this is normal!**
5. **Repeat** steps 1–4 for the right half with the right `.uf2` file

### Important Notes for Windows Users

- Windows may show an **error dialog** saying the device was disconnected or the copy failed. **This is a false alarm** — the nice!nano reboots itself after receiving the firmware, which Windows interprets as an unexpected disconnection. The flash almost certainly succeeded.
- If the NICENANO drive doesn't appear, try a different USB cable (some cables are charge-only and don't carry data).

---

## Troubleshooting

### Keyboard won't connect via Bluetooth

- Make sure you're pairing the **left half** (the central device)
- Try clearing the Bluetooth profile: hold FN → select the profile → press BT CLR
- On your computer, remove "Dactyl 4x6" from your Bluetooth devices and re-pair
- Flash both halves with fresh firmware

### Bluetooth name shows old/wrong name after firmware update

ZMK stores the Bluetooth name in **persistent flash memory**. Updating firmware alone won't change it. To force a reset:

1. Add this line to `config/dactyl_manuform_4x6.conf`:
   ```
   CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=y
   ```
2. Build and flash the new firmware
3. **Important:** Remove that line and flash again afterward, or bonds will clear on every boot

Alternatively, you can do a full settings reset by entering the bootloader and flashing a [ZMK settings reset](https://zmk.dev/docs/troubleshooting#split-keyboard-halves-unable-to-pair) `.uf2` file before flashing your actual firmware.

### Right half not communicating with left half

- Both halves need to be powered on
- Flash the **settings reset** firmware to both halves, then flash the actual firmware to both
- Make sure you're flashing the correct file to each half (left `.uf2` to left, right `.uf2` to right)

### A key registers the wrong character

- Double-check your wiring matches the pin mapping in this guide
- Verify your diode orientation (the cathode stripe should point toward the row wire)
- Check the keymap file for any unintended changes

### Double-press the reset button but no NICENANO drive appears

- Try different USB cables — many cables are charge-only
- Try different USB ports
- Hold the reset button for a few seconds, then double-tap
- On some nice!nano v2 boards, you may need to bridge the RST and GND pins manually

---

## Customizing Your Keymap

The keymap lives in `config/dactyl_manuform_4x6.keymap`. Here's a quick reference for making changes.

### Common Key Codes

| ZMK Code | Key | ZMK Code | Key |
|----------|-----|----------|-----|
| `&kp A` – `&kp Z` | Letters | `&kp N1` – `&kp N0` | Numbers |
| `&kp SPACE` | Space | `&kp RET` | Enter |
| `&kp BSPC` | Backspace | `&kp DEL` | Delete |
| `&kp TAB` | Tab | `&kp ESC` | Escape |
| `&kp LSHFT` | Left Shift | `&kp RSHFT` | Right Shift |
| `&kp LCTRL` | Left Ctrl | `&kp RCTRL` | Right Ctrl |
| `&kp LALT` | Left Alt | `&kp RALT` | Right Alt |
| `&kp LGUI` | Left GUI (Win/Cmd) | `&kp RGUI` | Right GUI |
| `&kp UP` | Arrow Up | `&kp DOWN` | Arrow Down |
| `&kp LEFT` | Arrow Left | `&kp RIGHT` | Arrow Right |
| `&trans` | Transparent (pass through to layer below) | `&none` | Do nothing |

### Special Behaviors

| Code | What It Does | Example |
|------|-------------|---------|
| `&kp KEY` | Simple keypress | `&kp A` → types "a" |
| `&lt LAYER KEY` | Layer-tap: hold for layer, tap for key | `&lt 1 SPACE` → hold = Layer 1, tap = Space |
| `&mo LAYER` | Momentary layer: active while held | `&mo 3` → Layer 3 while held |
| `&bt BT_SEL N` | Select Bluetooth profile N (0–4) | `&bt BT_SEL 0` → switch to profile 1 |
| `&bt BT_CLR` | Clear current Bluetooth profile | — |

### Adjusting Tapping Term

The `tapping-term-ms` setting (currently 200ms) controls how long you need to hold a layer-tap key before it activates the layer instead of typing the key. Find this section in the keymap:

```dts
behaviors {
    lt {
        tapping-term-ms = <200>;
    };
};
```

Change the value to suit your preference. Lower = faster taps, higher = easier holds.

### Adding a New Layer

To add a layer, define it inside the `keymap` block and reference it with `&mo` or `&lt` from another layer.

Full key code reference: [ZMK Key Codes Documentation](https://zmk.dev/docs/codes)

---

## File Structure Reference

Here's what every file in this config does:

```
├── build.yaml                          # Tells GitHub Actions what to build
│                                       # (board: nice_nano_v2, both shield halves)
│
├── config/
│   ├── dactyl_manuform_4x6.conf        # Keyboard settings (Bluetooth name, features)
│   ├── dactyl_manuform_4x6.keymap      # ★ YOUR KEYMAP — this is what you edit
│   └── west.yml                        # ZMK dependency/version management
│
├── boards/shields/dactyl_manuform_4x6/
│   ├── dactyl_manuform_4x6.dtsi        # Hardware definition: matrix size, pin mapping,
│   │                                   # physical key-to-matrix position mapping
│   ├── dactyl_manuform_4x6_left.overlay # Left half column GPIO pins
│   ├── dactyl_manuform_4x6_right.overlay# Right half column GPIO pins + column offset
│   ├── dactyl_manuform_4x6.zmk.yml     # Shield metadata (name, type, features)
│   ├── Kconfig.shield                  # Shield detection for build system
│   └── Kconfig.defconfig               # Default config (split mode, central role)
│
└── zephyr/
    └── module.yml                      # Tells Zephyr where to find board files
```

**The file you'll edit most:** `config/dactyl_manuform_4x6.keymap` — this is where your entire layout lives.

---

## Resources

- **ZMK Documentation:** [zmk.dev/docs](https://zmk.dev/docs)
- **ZMK Key Codes:** [zmk.dev/docs/codes](https://zmk.dev/docs/codes)
- **ZMK Behaviors:** [zmk.dev/docs/keymaps/behaviors](https://zmk.dev/docs/keymaps/behaviors)
- **Keymap Editor (GUI):** [nickcoutsos.github.io/keymap-editor](https://nickcoutsos.github.io/keymap-editor/) — visual editor that works with ZMK GitHub repos
- **ZMK Troubleshooting:** [zmk.dev/docs/troubleshooting](https://zmk.dev/docs/troubleshooting)
- **Dactyl Manuform Info:** [github.com/abstracthat/dactyl-manuform](https://github.com/abstracthat/dactyl-manuform)

---

*Happy typing! The first few days on a split ergonomic keyboard feel strange, but most people find their groove within 1–2 weeks. Stick with it — your wrists will thank you.*