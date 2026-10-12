# GTA V PS3 (BLES01807) – 60 FPS, Frame Rate and Graphics options

Binary patches (BSDIFF40, apply with `bspatch`) for **title update 1.27** of the European
PS3 release (`EP1004-BLES01807_00-GTAVTITLEUPDATER`). They add two rows to
**Settings > Display**, both switchable live in game:

- **Frame Rate**: 60 FPS (uncapped, vsync off) / 30 FPS (stock)
- **Graphics**: Default / Very Low (see below)

No game files are included. The patches only contain the changed bytes and the new menu text.

## Files

| Patch | Applies to | Result |
|---|---|---|
| `eboot/eboot_60fps_only.bsdiff` | decrypted `EBOOT.BIN` | plain uncapped 60 FPS, no menu rows, no `update.rpf` changes needed |
| `eboot/eboot_fps_toggle.bsdiff` | decrypted `EBOOT.BIN` | Frame Rate row |
| `eboot/eboot_fps_toggle_graphics.bsdiff` | decrypted `EBOOT.BIN` | Frame Rate + Graphics rows |
| `update_rpf/pausemenu.xml.bsdiff` | `common/data/ui/pausemenu.xml` in `update.rpf` | adds both rows |
| `update_rpf/lang/<language>.gxt2.bsdiff` | `global.gxt2` in `update.rpf/ps3/patch/data/lang/<language>.rpf` | row labels (English text in all 11 languages) |

The two toggle eboots need the patched `update.rpf` for their menu rows. They also disable the
game's SHA1 check of `update.rpf`, which is what allows the edited archive to load. With
`eboot_fps_toggle.bsdiff` the Graphics row is shown but does nothing.

## What you need

- A PS3 that runs modified NPDRM eboots (CFW or HEN), with title update 1.27 installed in
  `/dev_hdd0/game/BLES01807/USRDIR/`
- `bspatch` (macOS ships it; Linux: package `bsdiff`)
- [scetool](https://github.com/naehrwert/scetool) with your own keys (not included), to decrypt
  and re-sign the eboot
- For the menu rows: an RPF7 tool that can open the PS3 `update.rpf` (big-endian, AES encrypted
  table of contents) and replace files inside it

## How to apply

**1. Eboot.** Copy `EBOOT.BIN` from `/dev_hdd0/game/BLES01807/USRDIR/`, decrypt it and patch it:

```sh
scetool --decrypt EBOOT.BIN EBOOT.ELF
shasum EBOOT.ELF          # must be 7048a2c963dfb3b0440c13db21adde8ad4286a45
bspatch EBOOT.ELF EBOOT_PATCHED.ELF eboot/eboot_fps_toggle_graphics.bsdiff
shasum EBOOT_PATCHED.ELF  # compare with the table below
```

Re-sign it as a free-license NPDRM update eboot:

```sh
scetool --sce-type=SELF --compress-data=TRUE --skip-sections=TRUE --key-revision=0A \
  --self-auth-id=1010000001000003 --self-vendor-id=01000002 \
  --self-app-version=0001000000000000 --self-fw-version=0003005500000000 --self-add-shdrs=TRUE \
  --self-type=NPDRM --np-license-type=FREE --np-app-type=UEXEC \
  --np-content-id=EP1004-BLES01807_00-GTAVTITLEUPDATER --np-real-fname=EBOOT.BIN \
  --encrypt EBOOT_PATCHED.ELF EBOOT.BIN
```

`--skip-sections=TRUE` is required (otherwise the console reports 80010017). scetool's
`data/keys` file must use LF line endings.

**2. update.rpf** (toggle eboots only). Extract these files, patch each one, and put it back
into the archive:

```sh
bspatch pausemenu.xml pausemenu.xml.new update_rpf/pausemenu.xml.bsdiff
# for every language archive ps3/patch/data/lang/<language>.rpf:
bspatch global.gxt2 global.gxt2.new update_rpf/lang/<language>.gxt2.bsdiff
```

Check the SHA1 of every input before patching. The archive may grow; that is fine with the
patched eboot.

**3. Install.** Back up the originals, then copy `EBOOT.BIN` and `update.rpf` to
`/dev_hdd0/game/BLES01807/USRDIR/`.

## SHA1 checksums

| File | Input | Output |
|---|---|---|
| eboot (all three) | `7048a2c963dfb3b0440c13db21adde8ad4286a45` | |
| `eboot_60fps_only` | | `796fd010eded9d8eb55fe4e91df0d257c5ab89a1` |
| `eboot_fps_toggle` | | `f8d0a2c3aabcd7622a76d8d7a9c7743436fca845` |
| `eboot_fps_toggle_graphics` | | `0ac048aefe183ae4e78f975a33ce9d4d9a00e8e8` |
| `pausemenu.xml` | `a23c6c8d426c54de1051379e9c918b84243b9dcc` | `f42e5bba55f55448b2051f1f156b5f2cef8e1da9` |
| `american` gxt2 | `4cc24dbbe4b26b24813649c997a641763439449b` | `a203bbb8653c3fab243f4bf7caec88be06585bc2` |
| `chinese` gxt2 | `1080774b33e7b00dfd6d2ea1ca7cf2df27514168` | `c8211bda15f980355a99eda7761485e937ef907b` |
| `french` gxt2 | `0a73c532b84c2bb102f1e71d383ba81a57a7e2dd` | `f7facca5f8fad0093b6b22a5c28f6ed5713f437d` |
| `german` gxt2 | `57aa5726b3eed1128f1e6e81eeac3f4d72e23287` | `166b8892d73e9e73b2c991d9ceaa8d6103b62532` |
| `italian` gxt2 | `685e33d5109abb60b4db2f48a0681ee6e147be56` | `f702f5aeef809118b671e819f31133f923531159` |
| `korean` gxt2 | `940ccc5911650764d6f8832e0907892e87742ef5` | `d0039b568c565e4adaa610f4b961a138ac0a5de7` |
| `mexican` gxt2 | `c9e6d8fcc3f1e5d37766013d1d0e1c0baf979e84` | `1b028c15c72662115be7ecf4de7e88356fae0c9d` |
| `polish` gxt2 | `710ba15a095c5ae4ec8c127dcdcdc68275473bec` | `90f2f00b9511185a34e63c03866e44fb4e27db06` |
| `portuguese` gxt2 | `b708d433f80eb75b6d6ed2c3a56149eddc9f7e0b` | `4e09ba81bbe157ca7a9aa8d8ff6b46984064de62` |
| `russian` gxt2 | `df3702a7f1887ba3171a6da372bc88a0ec3b2f9c` | `ce3b52232fba5ac0c7baaf8f2904bafcbc392ac6` |
| `spanish` gxt2 | `88b2e43c80b0c069d81e51c280e954ff81f33163` | `412013c5667bd5f367481f2d3a8acd6520c79bac` |

## Graphics: Very Low

Very Low is meant for the highest possible frame rate, not for looks. It only changes what the
game renders and simulates while the row is set to Very Low; Default is the stock game.

- No shadows (sun, street lamps, headlights) and no grass
- No water or car/glass reflections, mirrors, rain height map, rain update or ped damage decals
- Water surface hidden, clouds frozen, ambient occlusion off
- Draw distance 35%, traffic and pedestrians 12.5%, distant lights 25%, light radius 50%

Both options may reset to their defaults (60 FPS, Default) when the game starts.

## Status

- `eboot_60fps_only` and `eboot_fps_toggle`: confirmed working on a CFW console.
- `eboot_fps_toggle_graphics`: tested on one CFW console. Known issue: at high frame rates
  textures can take longer to stream in (under investigation).

## Disclaimer

Not affiliated with or endorsed by Rockstar Games or Take-Two Interactive. You need your own
legally obtained copy of the game. Released into the public domain, see `LICENSE`.
