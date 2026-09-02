# CTR: Nitro-Fueled — Eden / yuzu mods

Exefs patches for **Crash Team Racing Nitro-Fueled** `[0100F9F00C696000]`, in the
`.pchtxt` (IPSwitch) format used by Eden, yuzu forks, Ryujinx and Atmosphère.

The game drops its internal render resolution to **1024x576** whenever 3 or 4
players share the screen. Emulator resolution scaling cannot undo that — it
multiplies an image that is already reduced. These mods remove the drop at the
source.

### Mods

| Mod | Description | Applies to | Credit |
| --- | ----------- | ---------- | ------ |
| [Disable Splitscreen Dynamic Resolution 720p](https://raw.githubusercontent.com/drxid/ctr-nf-eden-mods/main/files/Crash%20Team%20Racing%20Nitro-Fueled/Disable%20Splitscreen%20Dynamic%20Resolution%20720p.zip) | 3-4 player splitscreen renders at 1280x720 instead of 1024x576, matching 1 and 2 player mode. No extra cost over stock 2-player. | `1.0.14` | [drxid](https://github.com/drxid) |
| [Disable Splitscreen Dynamic Resolution 1080p](https://raw.githubusercontent.com/drxid/ctr-nf-eden-mods/main/files/Crash%20Team%20Racing%20Nitro-Fueled/Disable%20Splitscreen%20Dynamic%20Resolution%201080p.zip) | All splitscreen modes (2, 3 and 4 players) render at 1920x1080 — game-side supersampling on top of the fix above. | `1.0.14` | [drxid](https://github.com/drxid) |

**Enable only one of the two.** They patch the same instruction.

### Installation

1. Download a `.zip` from the table above.
2. Right-click the game in Eden → **Open Mod Data Location**.
3. Extract the zip there, so you end up with:
   `load\0100F9F00C696000\Disable Splitscreen Dynamic Resolution 720p\exefs\*.pchtxt`
4. Right-click the game → **Properties → Add-Ons**, tick the mod.

To confirm it is active, check `eden\log\eden_log.txt` for:

```
[IPSwitchCompiler ('disable-splitscreen-dynamic-resolution-720p.pchtxt')]
    - Patching value at offset 0x00AC3D68 with byte string 'E8031F2A'
```

### Compatibility

Built for game version **1.0.14**, `main` NSO build id
`47E60871471F1CE9E212B81DEC5A6249`. Patch offsets are build-specific — on any
other version the `@nsobid` will not match and Eden will silently skip the mod.

Your build id is printed in `eden_log.txt` on every boot:

```
HasNSOPatch: Querying NSO patch existence for build_id=..., name=main
```

### Notes on the 1080p version

- It stacks with the emulator's own resolution multiplier: 1.5x game-side times
  whatever Eden is set to. Lower the multiplier a couple of steps or you will
  render absurd resolutions for no visible gain.
- The engine allocates render targets from a fixed pool (it tracks overruns in
  `_dynamicVramShortfall` / `_dynamicVRAMHighWater`). 2.25x the pixels may not
  fit. If a 3-4 player race crashes or loses effects, use the 720p version.
- To pick another resolution, edit the two floats in the pchtxt (keep 16:9):

  | Resolution | width `040A9B88` | height `040A9CD0` |
  | ---------- | ---------------- | ----------------- |
  | 1280x720   | `0000A044`       | `00003444`        |
  | 1600x900   | `0000C844`       | `00006144`        |
  | 1920x1080  | `0000F044`       | `00008744`        |
  | 2560x1440  | `00002045`       | `0000B444`        |

### How it works

The game ships on the Alchemy/IG engine with a full symbol table, so the path is
easy to follow.

`CRender::setGameRenderer(eSRM_Splitscreen)` reads the internal resolution from a
two-entry float table indexed by the local player count:

```
width  table @ 0x040A9B88 : [0] = 1280.0   [1] = 1024.0
height table @ 0x040A9CD0 : [0] =  720.0   [1] =  576.0

count = *(CRender + 0x230)
if (count < 2) -> fullscreen path
w24 = count - 3
idx = (unsigned)w24 < 2 ? 1 : 0     // cset w8, lo  @ 0x00AC3C68
      count == 2 -> idx 0 -> 1280x720
      count == 3 -> idx 1 -> 1024x576
      count == 4 -> idx 1 -> 1024x576
```

That resolution becomes a pair of global scale factors on the graphics device:

```
device[0x500] = width  / 1280.0
device[0x504] = height /  720.0
```

`Render::igRenderTarget::activate()` applies them to every render target whose
authored `_highResolutionScale` lies in `(1.0, 2.0)`:

```
computedWidth  = max(1, backBufferWidth  * _widthScale)
computedHeight = max(1, backBufferHeight * _heightScale)
if (_highResolutionScale > 1.0) {
    factor = (_highResolutionScale * 0.5 >= 1.0) ? (1.0, 1.0)
                                                 : (device[0x500], device[0x504])
    computedWidth  *= factor.x
    computedHeight *= factor.y
}
```

So with 3-4 players the whole scene is rendered at 80% linear resolution and
upscaled. `Render::igRenderContext::activate()` and
`Gui::igGuiContext::updateScreenResolution()` read the same pair, which is why
viewports and HUD stay in proportion when the value is changed.

**720p patch** — force the table index to 0:

```
0x00AC3C68   cset w8, lo   (E8279F1A)  ->  mov w8, wzr   (E8031F2A)
```

**1080p patch** — the same, plus entry `[0]` of each table raised to 1920/1080.
Both table entries are referenced from exactly one instruction each in the whole
34 MB of `.text`, so nothing else in the binary is affected.

### Credits

Reverse engineering and patches by [drxid](https://github.com/drxid).

Format and repository layout follow
[yuzu-mod-archive](https://github.com/yuzu-mirror/yuzu-mod-archive) and
[StevensND/switch-port-mods](https://github.com/StevensND/switch-port-mods).
