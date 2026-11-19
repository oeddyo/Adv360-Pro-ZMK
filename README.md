# My Kinesis Advantage360 Pro firmware

This is the firmware config for my Kinesis Advantage360 Pro: Kinesis's
[Adv360-Pro-ZMK](https://github.com/KinesisCorporation/Adv360-Pro-ZMK) `V3.0` branch, plus my
own base layer. I use the keyboard for my wrists, so only the base layer is customised. Every
other layer is Kinesis's default.

Kinesis's own README is [upstream](https://github.com/KinesisCorporation/Adv360-Pro-ZMK/blob/V3.0/README.md).
The [User's Manual](https://kinesis-ergo.com/support/kb360pro/#manuals) is the source for the
keyboard's shortcuts and lights. Check it before guessing.

## My layout

Only `default_layer` in `config/adv360.keymap` differs from Kinesis's default:

| Position (Kinesis default)       | Mine                    |
| -------------------------------- | ----------------------- |
| Top-left (`=`)                   | Esc                     |
| Top-right (`-`)                  | Backspace               |
| Left home-row pinky (Esc)        | Left Ctrl               |
| Right Shift                      | `=`                     |
| Bottom-left pinky (hold for Fn)  | Hold for the keypad layer |
| Bottom-right pinky (hold for Fn) | `-`                     |
| Left thumb, tall keys (Backspace, Delete) | Space, Cmd     |
| Right thumb, tall keys (Enter, Space)     | Backspace, Enter |

The 2023 version of this layout, on the older V2 firmware, is on the `V2.0` branch.

## Keyboard cheat sheet

**Mod** is the key marked "mod" at the top of the right half's inner column. The hotkeys are the
four middle keys marked ①–④: ① and ② sit under "kp" on the left half, and ③ and ④ sit under
"mod" on the right half.

| Shortcut                     | What it does                                          |
| ---------------------------- | ----------------------------------------------------- |
| Mod + 1 … 5                  | Switch Bluetooth profile (white, blue, red, green, off) |
| Mod + right thumb Win/Cmd key | **Clear the pairing** on the current profile        |
| Mod + ①                      | Left half into bootloader mode (for flashing). **Not** a Bluetooth clear. |
| Mod + ③                      | Right half into bootloader mode                       |
| Mod + ④ (hold)               | Show battery level on the lights                      |
| Mod + V                      | Type the firmware version                             |

The middle light on the left half shows the Bluetooth profile:

- Flashing fast: ready to pair.
- Flashing slowly: paired, but that computer is out of range.
- Solid: connected.

If the right half's lights flash red, the halves have lost each other. Switch both off, then
turn on the left and, straight after it, the right.

**Battery colours (Mod + ④):** green above 80%, yellow 51–79%, orange 21–50%, red below 20%.

### Re-pairing with a Mac

1. If the Mac lists "Adv360 Pro" under My Devices, remove it there.
2. Pick the profile with Mod + 1 … 5.
3. Press Mod + the right thumb Win/Cmd key. The profile light flashes fast.
4. Click "Adv360 Pro" under Nearby Devices in the Mac's Bluetooth settings.

## Changing a key

1. Edit `default_layer` in `config/adv360.keymap`. Key names are in the
   [ZMK keycode list](https://zmk.dev/docs/keymaps/list-of-keycodes).
2. Build (see below).
3. Flash the **left half only**. The left half holds the keymap, and no settings reset is needed.

## Building

Run this from the repo root, with Docker Desktop running:

```sh
make
```

It writes `firmware/<time>-<commit>-left-clique.uf2` and `...-right-clique.uf2` (git ignores
both). The first build downloads the toolchain and takes several minutes; later builds are quicker.

`make` runs Docker with `-it`, which needs a real terminal. From a script or an agent, run the same
steps without `-it`:

```sh
bin/get_version_local.sh clique > /dev/null
docker build --tag zmk --file Dockerfile .
docker run --rm --name zmk \
  -v "$PWD/firmware:/app/firmware" -v "$PWD/config:/app/config:ro" \
  -e TIMESTAMP="$(date -u +%Y%m%d%H%M)" -e COMMIT="$(git rev-parse --short HEAD)" \
  -e BUILD_RIGHT=true zmk
git checkout config/version.dtsi
```

The first line stamps the version into `config/version.dtsi`; the last line puts that file back.

## Flashing

To flash a half, put it into bootloader mode and copy a `.uf2` file onto the drive it mounts:

1. Plug **only that half** into the Mac with a USB cable. Both halves mount under the same
   name, so with both plugged in you can't tell which is which.
2. Enter bootloader mode. Either double-press the reset button with a paperclip (in the thumb
   cluster, where three keys meet), or press Mod + ① (left) or Mod + ③ (right). The shortcut
   only works while the keyboard has working firmware.
3. A drive called `ADV360PRO` appears. Copy the file:

   ```sh
   cp -X firmware/*-left-clique.uf2 /Volumes/ADV360PRO/
   ```

   Swap in `right` for the right half. `-X` skips the file's metadata. Without it, `cp` prints
   "could not copy extended attributes … Device not configured", which is harmless.
4. The drive ejects itself when the flash is done. macOS may warn that it wasn't ejected
   properly; ignore the warning.

### Full update: new firmware or halves not syncing

The settings reset wipes Bluetooth pairings and the link between the halves. The keyboard can't
type from the first reset until both halves have new firmware, so keep another keyboard handy,
and use the paperclip from step 2 on.

1. Left half: `settings-reset.uf2`.
2. Right half: `settings-reset.uf2`.
3. Left half: `firmware/*-left-clique.uf2`.
4. Right half: `firmware/*-right-clique.uf2`.
5. Unplug, switch both halves off, then turn on the left and, straight after it, the right.
6. Pair again (see [Re-pairing with a Mac](#re-pairing-with-a-mac)).

## Pulling Kinesis updates

```sh
git fetch upstream
git merge upstream/V3.0
```

- **Conflicts:** if `README.md` conflicts, keep this one. If `config/adv360.keymap` conflicts,
  take theirs and re-apply the [My layout](#my-layout) rows to `default_layer`.
- **Settings reset:** read `CHANGELOG.md` for what changed. If the update says a settings reset
  is needed, use the full update steps. Otherwise build and flash both halves, left then right.

Remotes: `origin` is this repo; `upstream` is
`https://github.com/KinesisCorporation/Adv360-Pro-ZMK.git`.

## Clique

The firmware built here works with Kinesis Clique, an app for changing keys without flashing.
Clique saves its changes on the keyboard, not in this repo, and the next settings reset wipes them.
Make lasting changes in `config/adv360.keymap`.
