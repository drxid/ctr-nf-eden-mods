# CTR: Nitro-Fueled — Eden / yuzu mods

Exefs patches for **Crash Team Racing Nitro-Fueled** `[0100F9F00C696000]`, in the
`.pchtxt` (IPSwitch) format used by Eden, yuzu forks, Ryujinx and Atmosphère.

With 3 or 4 players on screen the game drops its internal render resolution to
1024x576. Emulator resolution scaling cannot undo that, because it multiplies an
image that is already reduced. These mods remove the drop at the source.

### Mods

| Mod | Description | Applies to | Credit |
| --- | ----------- | ---------- | ------ |
| [Disable Splitscreen Dynamic Resolution 720p](https://raw.githubusercontent.com/drxid/ctr-nf-eden-mods/main/files/Crash%20Team%20Racing%20Nitro-Fueled/Disable%20Splitscreen%20Dynamic%20Resolution%20720p.zip) | 3-4 player splitscreen renders at 1280x720 instead of 1024x576, matching 1 and 2 player mode. | `1.0.14` | [drxid](https://github.com/drxid) |
| [Disable Splitscreen Dynamic Resolution 1080p](https://raw.githubusercontent.com/drxid/ctr-nf-eden-mods/main/files/Crash%20Team%20Racing%20Nitro-Fueled/Disable%20Splitscreen%20Dynamic%20Resolution%201080p.zip) | All splitscreen modes render at 1920x1080 — extra sharpness on top of the fix above, at 2.25x the pixels. | `1.0.14` | [drxid](https://github.com/drxid) |

**Enable only one of the two.**

### Installation

1. Download a `.zip` from the table above.
2. Right-click the game in Eden → **Open Mod Data Location**.
3. Extract the zip there, so you get
   `load\0100F9F00C696000\<mod name>\exefs\*.pchtxt`
4. Right-click the game → **Properties → Add-Ons**, tick the mod.

### Compatibility

Built for game version **1.0.14**, `main` NSO build id
`47E60871471F1CE9E212B81DEC5A6249`. Patches are build-specific: on any other
version the build id will not match and the mod is silently skipped. Yours is
printed in `eden\log\eden_log.txt` on every boot.

### Notes on the 1080p version

- It stacks with the emulator's own resolution multiplier. Lower that multiplier
  a couple of steps, or you will render absurd resolutions for no visible gain.
- The engine allocates render targets from a fixed pool. If a 3-4 player race
  crashes or loses effects, use the 720p version instead.

### Credits

Reverse engineering and patches by [drxid](https://github.com/drxid).

Repository layout follows
[yuzu-mod-archive](https://github.com/yuzu-mirror/yuzu-mod-archive).

### License

[MIT](LICENSE)
