# ZMK Keyboard Firmware Configuration

This is a ZMK (Zephyr Mechanical Keyboard) firmware configuration repository for the SplitKB Aurora Lily58 keyboard with nice!nano controller.

## Architecture

- **Build System**: GitHub Actions CI/CD using ZMK's reusable workflow (`zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`)
- **Configuration Location**: All keyboard-specific configs are in [config/](config/)
- **Hardware**: nice!nano controller with splitkb_aurora_lily58 shield + nice_view display
- **Build Matrix**: Defined in [build.yaml](build.yaml) - builds left/right halves separately plus settings_reset

## Key Files

- [config/splitkb_aurora_lily58.keymap](config/splitkb_aurora_lily58.keymap): Main keymap definition using ZMK devicetree format
- [config/splitkb_aurora_lily58.conf](config/splitkb_aurora_lily58.conf): Feature flags (encoders, display, pointing)
- [config/west.yml](config/west.yml): West manifest pointing to `zmkfirmware/zmk@main`
- [build.yaml](build.yaml): Build matrix for GitHub Actions (left, right, settings_reset)

## Keymap Structure (Devicetree)

The keymap uses **devicetree syntax** (.dtsi), not C code:

```dts
/ {
    behaviors { /* custom behavior definitions */ };
    keymap {
        compatible = "zmk,keymap";
        layer_name {
            bindings = <&kp KEY ...>;  // 58-key matrix
            sensor-bindings = <...>;    // encoder behavior
        };
    };
};
```

### Current Layers
1. **default_layer** (0): QWERTY with homerow mods (hml/hmr behaviors)
2. **lower_layer** (1): Symbols, Bluetooth controls
3. **raise_layer** (2): Navigation, numbers, shortcuts
4. **adjust_layer** (3): Function keys

Access layers via `&mo N` (momentary) in bindings.

## Homerow Mods Pattern

Custom `hml` (left) and `hmr` (right) behaviors implement positional hold-tap:
- **Tapping term**: 200ms
- **Quick tap**: 175ms within same hand
- **Flavor**: "balanced"
- **Hold-trigger-key-positions**: Opposite hand only (prevents accidental mods on same-hand rolls)
- **hold-trigger-on-release**: Enables rolling modifiers

Example from [splitkb_aurora_lily58.keymap#L62](config/splitkb_aurora_lily58.keymap#L62):
```dts
&hml LCTRL A  // A on tap, Left Ctrl on hold (only triggers mod when right-hand keys pressed)
```

## Configuration Conventions

### Feature Flags ([splitkb_aurora_lily58.conf](config/splitkb_aurora_lily58.conf))
- Encoder support: `CONFIG_EC11=y` + `CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y`
- Mouse/pointing: `CONFIG_ZMK_POINTING=y` (enables `&msc` behaviors)
- Display: `CONFIG_ZMK_DISPLAY=y` (for nice!view)
- macOS compatibility: `CONFIG_ZMK_HID_CONSUMER_REPORT_USAGES_BASIC=y`

### Sensor Bindings (Encoders)
Each layer defines encoder behavior via `sensor-bindings`:
- Volume: `&inc_dec_kp C_VOL_DN C_VOL_UP`
- Scroll: `&msc SCRL_DOWN`, `&msc SCRL_UP` (requires CONFIG_ZMK_POINTING)
- Page: `&inc_dec_kp PAGE_DOWN PAGE_UP`

## Build & Deploy Workflow

1. **Push to GitHub** → GitHub Actions builds firmware automatically
2. **Artifacts**: `firmware.zip` contains `.uf2` files for left/right halves
3. **Flash**: Download `.uf2` files, drag to nice!nano USB drive when in bootloader mode
4. **Settings Reset**: Flash `settings_reset.uf2` to clear bond data if Bluetooth issues occur

No local build needed - CI handles everything via [.github/workflows/build.yml](.github/workflows/build.yml).

## Making Changes

### Adding a Key Binding
Edit the 58-key matrix in the relevant layer (count positions carefully):
```dts
bindings = <
    &kp ESC   &kp N1   ...  // Row 1 (6 left + 6 right = 12 keys)
    &kp TAB   &kp Q    ...  // Row 2 (12 keys)
    ...
>;
```

### Adding a Custom Behavior
Define in the `behaviors` block before `keymap`, reference via `&behavior_name` in bindings.

### Adding a New Layer
Add to `keymap` block, access via `&mo N` where N is zero-indexed layer number.

## External Dependencies

- **ZMK Firmware**: Pulled from `zmkfirmware/zmk@main` via West (Zephyr build tool)
- **Shields**: `splitkb_aurora_lily58_{left,right}`, `nice_view_adapter`, `nice_view`
- **Board**: `nice_nano` (nRF52840 controller)

West automatically fetches dependencies during CI build - no manual setup required for normal keymap changes.
