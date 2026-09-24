# custom-fetch

A fastfetch-like system info tool in **pure bash** where you can just drop in a
standard **JPEG or PNG** (also WebP/GIF/BMP) and get **ASCII/truecolor art** —
no kitty graphics protocol, no sixel setup, no terminal config. Plain ANSI, so
it works over SSH and in every terminal.

```sh
./fetch --image photo.jpg
./fetch -i logo.png -w 50 --mode half
./fetch ~/Pictures/wallpaper.jpg
./fetch --logo arch
```

## Why?

Normal fastfetch image support needs a graphics-capable terminal
(kitty/sixel/iTerm) plus configuration. `custom-fetch` converts images to
colored `▀` half-blocks / `█` blocks / classic ASCII, so **zero setup** is
required — if your terminal shows text, it shows your image.

## Requirements

- `bash` 4+, `python3` (both preinstalled on virtually every distro)
- **Best quality (recommended, usually already present):** one of
  - `python3-pillow` (`pip install pillow` / `pacman -S python-pillow` / `apt install python3-pil`)
  - `imagemagick` (`convert`/`magick`)
  - `ffmpeg`
- **Zero-dependency fallback:** PNGs decode with python stdlib only
  (`zlib`); JPEGs need one of the above.

Check what you have:

```sh
python3 -c "import PIL; print(PIL.__version__)"
which convert magick ffmpeg
```

## Install / run

Single file, no build:

```sh
git clone <this-repo> && cd custom-fetch
chmod +x fetch
./fetch --image images/example.jpg
```

Optional: symlink into PATH:

```sh
ln -s "$PWD/fetch" ~/.local/bin/fetch
```

## Usage

```text
Usage:
  fetch [OPTIONS] [IMAGE]

Options:
  -i, --image PATH     JPEG/PNG/WebP/GIF/BMP file (or directory: picks first image)
  -w, --width N        ASCII width in chars (default: 40, range 10-200)
  --mode MODE          half (default) | full | ascii | mono
                         half  = ▀ 2 pixels per cell, best fidelity, truecolor
                         full  = █ 1 pixel per cell, tall but vivid
                         ascii = charset-mapped, truecolor
                         mono  = charset-mapped, no color
  --ascii              shortcut for --mode ascii
  --mono               shortcut for --mode mono --color off
  --charset STR        custom charset for ascii/mono (default: ' .:-=+*#%@')
  --color on|off       ANSI colors (default: on)
  --logo NAME          built-in text logo instead of image
                       (arch, ubuntu, debian, fedora, manjaro, mint,
                        pop, alpine, gentoo, nix, void, generic)
  --list-logos         list built-in text logos
  --no-info, --image-only   show image only
  --info-only               show system info only
  -h, --help           help
  -v, --version        version
```

Examples:

```sh
fetch --image photo.jpg                    # side-by-side system info
fetch -i logo.png -w 60 --mode half        # wide half-block render
fetch -i pic.jpg --ascii -w 70             # classic ASCII, still colored
fetch -i pic.jpg --mono -w 80              # pure monochrome ASCII
fetch -i pic.jpg --color off               # no ANSI colors
fetch -i ~/Pictures --width 40             # first image in a directory
fetch --image photo.jpg --image-only       # just the art (for motd/scripts)
fetch --logo ubuntu                        # text logo fallback
fetch --info-only                          # no art at all
NO_COLOR=1 fetch -i photo.jpg              # respects NO_COLOR
```

## How image → ASCII works

Backends are auto-detected in order:

1. **python3 + Pillow** — direct decode, LANCZOS resize, truecolor `▀`
   (handles JPG/PNG/WebP/GIF/BMP + transparency composited on black).
2. **ImageMagick (`magick`/`convert`) or `ffmpeg`** → capped PPM → stdlib
   python scaler/renderer (no Pillow needed, handles JPG + PNG).
3. **Pure-python PNG decoder** (stdlib `zlib` only, 8-bit non-interlaced
   RGB/RGBA/palette/gray) — PNG works with literally zero extra packages.
4. `chafa` / `jp2a` if present (last resort).

Aspect correction: terminal cells are ~1:2 (w:h), so

- `half` renders `char_width × char_width·h/w` pixels then packs 2 rows per
  `▀` (aspect-correct),
- `ascii`/`full` render `char_width × char_width·h/w·0.5` pixels.

## Config file (optional)

`~/.config/custom-fetch/config` (`KEY=VALUE`, one per line):

```sh
IMAGE=$HOME/Pictures/logo.png
WIDTH=45
MODE=half
CHARSET= .:-=+*#%@
COLOR=on
```

CLI flags override config.

## System info shown

`user@host`, OS (`/etc/os-release`), host (`DMI`), kernel, uptime
(`/proc/uptime`), shell, DE (`XDG_CURRENT_DESKTOP`), terminal, CPU
(`/proc/cpuinfo`), GPU (`lspci`), memory (`/proc/meminfo`), disk (`df /`),
packages (pacman/dpkg/rpm/apk/xbps/portage/nix), plus an 8-color palette bar.

Typical runtime: ~0.08s (text logo), ~0.2s (image via Pillow).

## Project layout

```text
fetch            # the whole program (bash + embedded python, executable)
images/          # drop your .jpg/.png here; shipped with examples
README.md
```

## Tips

- For sharp logos use PNG with flat colors + `--mode half -w 40`.
- For photos use JPG + `--mode half -w 50`.
- For a retro terminal look use `--ascii` or `--mono --charset " .,:;irsXA253hMHGS#9B&@"`.
- Put `fetch -i ~/Pictures/fav.jpg` in `~/.bashrc` as a fastfetch replacement.
