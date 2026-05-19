# HP ProDesk 600 G3 SFF Tahoe OpenCore

OpenCore EFI for HP ProDesk 600 G3 SFF running macOS Tahoe.

## Hardware

- Model: HP ProDesk 600 G3 SFF
- Audio codec: Conexant CX20632 (`0x14f15098`)
- Target macOS: Tahoe
- SMBIOS: `iMac20,1`

## Contents

```text
EFI/                              OpenCore EFI
Audio/VoodooHDA-CX20632-speakerfix.kext
                                  Patched VoodooHDA for the internal speaker
```

## Before Use

Generate your own SMBIOS values before booting this EFI. The published `config.plist` has been sanitized.

Replace these fields in `EFI/OC/config.plist`:

- `PlatformInfo -> Generic -> SystemSerialNumber`
- `PlatformInfo -> Generic -> MLB`
- `PlatformInfo -> Generic -> SystemUUID`
- `PlatformInfo -> Generic -> ROM`

## Notes

- No graphics `device-id` injection is used. Injecting a graphics device ID can remove supersampled HiDPI modes; for example, on a 4K display, 2560x1440 HiDPI may disappear and only 1080p HiDPI remains.
- SMBIOS is set to `iMac20,1` for Tahoe compatibility. Other SMBIOS models may not be supported by Tahoe.
- Disable Intel Optane optimization for the M.2 slot in BIOS. Otherwise the M.2 NVMe drive may not be detected.
- Set DVMT pre-allocated memory to 64 MB in BIOS.
- USB mapping has already been customized for this machine.
- Tahoe removed AppleHDA, so the usual AppleALC path for analog audio may require restoring or patching system audio components and reducing SIP more aggressively. This EFI instead uses VoodooHDA for analog audio, together with a small CX20632 pin patch for the built-in speaker, so audio can work without taking the full AppleALC + system patch route.

## Audio

The bundled patched VoodooHDA is for the onboard Conexant CX20632 codec. On this machine, the internal speaker is `nid 31`. Stock VoodooHDA detected it as:

```text
nid 31 0x9217011f Speaker Fixed Analog Internal
```

but merged it into the same output association as the rear line-out/headphone path and then disabled the analog output because of a duplicate pin sequence:

```text
Duplicate pin 15 (31) in association 1! Disabling association.
```

The patched kext changes the internal speaker pin configuration to:

```text
Node:   31
Codec:  0
Config: 0x92170150
```

This places the internal speaker into its own association so VoodooHDA can expose a working analog speaker output.

## Installation Notes

- Copy `EFI` to your EFI partition.
- Use the patched VoodooHDA from `Audio/` for the internal speaker fix.
- Do not use AppleALC and VoodooHDA for the same analog codec at the same time.
- Reset NVRAM after changing OpenCore, SMBIOS, or audio kexts.

## Disclaimer

This configuration is shared as-is for the HP ProDesk 600 G3 SFF. Review every setting before using it on different hardware, especially SMBIOS, USB mapping, storage configuration, and audio routing.
